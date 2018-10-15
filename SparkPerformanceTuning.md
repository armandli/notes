### Optimize Spark 2.2

#### Spark Execution Model
spark application is consisted of a single driver process and a set of executor processes. driver is in charge of high level control flow, executor processes are responsible for executing the work in the form of **task**, as well as for storing data for caching. typically the same driver and executor stick around for the entire sprak job, but dynamic resource allocation changes that. a single executor has a fixed number of slots to for running **task**, and will run many tasks concurrently throughout its lifetime. deployment of task in the cluster is up to cluster manager (YARN, Mesos, Spark Standalone), but driver and executor exist in every spark application.

driver holds and runs a spark job, executor execute tasks distributed from cluster manager that's part of a job. Spark examine the graph of RDD and determine where to start by looking at RDD that are not dependent on anything, either from cache or already exist. the execution plan assemble the job transformations into stages, a **stage** correspond to a collection of task that all execute the same code, each on different subset of data. each stage contains a sequence of transformations that can be completed without shuffling of the full data.

RDD is composed of fixed number of **partitions**, each partition contains a number of **record**. in a **narrow transformation**, records required for computation all reside in a single partition in the parent RDD. Each object is only dependent on a single object in the parent. e.g. map, filter, but coalesce is also narrow, because even though it requires a task to process multiple partitions, the input records used to compute any single output record still reside in a limited subset of partitions.

**transformations** require wide dependencies such as groupByKey, reduceByKey need records from many partitions in parent RDD, and the set of partitions is not restricted to potentially all partitions of parent RDD. Spark need to execute a **shuffle**, which transfers data from the entire cluster and result in a new stage with new set of partitions.

a single stage contains all operations (transformation, action) that can be done without shuffling (needing data that come from different partitions than the input). e.g.

```
sc.textFile("someFile.txt").map(mapfunc).flatMap(flatMapFunc).filter(filterFunc).count() // all done in 1 stage

// following is done in 3 stages: reduceByKey, reduceByKey + collect, this is because reduceByKey uses groupByKey
val tokenzied = sc.textFile(args(0).flatMap(_.split(' ')))
val wordCounts = tokenized.map((_, 1)).reduceByKey(_ + _)
val filtered = wordCounts.filter(_._2 >= 1000)
val charCounts = filtered.flatMap(_._1.toCharArray).map((_, 1)).reduceByKey(_ + _)
charCounts.collect()
```

at each stage boundary, data is written to disk by tasks in the parent stages and then fetched over the network by tasks in the child stage, requiring heavy disk and network IO. number of partitions may differ between parent stage and child stage. transformations that will trigger a stage boundary typically take in a **numPartitions** parameter.

tuning the number of partitions at stage boundaries can often make or break an application's performance.

best practice involve 
* leverage tungsten
* execution plan analysis
* data management (caching, broadcast)
* cloud related optimizations (S3)

#### RDD Anatomy
composed of 5 parts:
1. set of partitions
2. list of dependencies on parent RDD
3. function to compute a partition given parent
4. partitioner
5. preffered lcation of each partition

first 3 properties combined to form **lineage**. last 2 properties are optimizations.

#### GC Optimization
by default Spark will use 60% of executor memory to cache RDD and 40% for regular objects. This can be changed using the option `spark.storage.memoryFraction=0.4` . it is recommended to reduce the amount of objects being created.

#### Spark Speculation
spark spawn additional tasks if it suspect a task is running on a stragler node, this happens if one task is taking much longer than other tasks. this should not happen if data is partitioned evenly and jobs are not expensive and have additional bottlenecks. configuration for spark speculation is `spark.speculation=true`, default is false. `spark.speculation.multiplier==1.5` denote how much slower a task is compared to its finished peers. `spark.speculation.interval=200` determine the frequency of speculation examination. `spark.speculation.quantile=0.95` determine the number of other tasks has to finish before spark speculation to kick in

#### Reduce Shuffle
shuffle is expensive because it needs to be written to disk and transfered through the network: repartition, join, cogroup, and *By operations and *ByKey operations. 

* avoid doing `groupByKey().mapValues(_.sum)`, which is the same as `reduceByKey(_ + _)`, which the later only transfer summaries from each executor, the first will shuffle the entire dataset
* avoid doing `ReduceByKey` when the input and output values types are different. e.g. finding all unique strings corresponding to each key, instead of doing

```
rdd.map(kv => (kv._1, new Set[String]() + kv._2)).reduceByKey(_ ++ _)
```

which generates a lot of unnecessary object creation because of a new set must be allocated for each record, use aggregateByKey

```
val zero = new collection.mutable.Set[String]()
rdd.aggregateByKey(zero)((set, v) => set += v, (set1, set2) => set1 ++= set2)
```

* avoid doing flatMap-join-groupBy. when 2 datasets are already grouped by key and want to be joined keeping the groupBy, you can just use **cogroup**, which avoid the overhead with unpacking and repacking the group

* use operations where there is no shuffle. spark knows to avoid shuffle when a previous transformation has already partitioned the data according to the same partitioner. e.g.

```
rdd1 = someRdd.reduceByKey(...)
rdd2 = someOtherRdd.reduceByKey(...)
rdd3 = rdd1.join(rdd2) // if both previous reduceByKey use the same partitioner, have the same number of partitions, the join will require no additional shuffling
```

because the RDD are partitioned identically, the set of keys in any single partition of rdd1 can be only show up in a single partition in rdd2. shuffle is not required.

when rdd1 and rdd2 use different partitioner or use the default hash partitioner with different number of partitions, only one of the rdd will need to be reshuffled for join.

* use broadcast variables to avoid shuffle. when one of the dataset is small enough to fit in memory in a single executor, it can be loaded into a hash table on the driver and then braodcast to every executor. a map transformation can then reference the hash table to do lookups.

#### When More Shuffles are Better
an exception to the rule is using more shuffle can be advantageous to performance by increasing parallelism. if data is in a few large unsplittable files, the partitioning done by **InputFormat** might place large number of records in each partition, repartition with high number of partitions after loading the data will allow operation to be more parallel.

when using reduce or aggregate action to aggregate data into the driver, aggregating over high number of partitions can quickly become bottlenecked on a single thread in the driver merging all results together. to loosen the load, use reduceByKey or aggregateByKey to carry out a round of distributed aggregation that divides the dataset into smaller number of partitions. value within each partition are merged with each other in parallel before sending result ot the driver for final round of aggregation. e.g. use **treeReduce** or **treeAggregate**.

#### Secondary Sort
**repartitionAndSortWithinPartition** transformation pushes sorting down to the shuffle machinery, creating a **secondary sort** pattern which you want to group records by key and then while iterating overt he values that correspond by key, have them show up in particular order.

#### Find out how many partitions programatically
`rdd.partitions.size` or `rdd.getNumPartitions` can tell the number of partitions in a RDD

#### Project Tungsten
with the improvement over network IO from 1Gbps to 40 Gbps, as well as disk IO from 50MB/s to solid state, CPU and memory optimization has become the new bottleneck. Tungsten looks into optimizing Spark for CPU and memory usage. e.g. a string "abcd" in Java requires 48 bytes of memory, not 4

tungsten option `spark.sql.tungsten.enabled=true`

tungsten 2 phase optimization
1. bypass GC by using native unsafe operations such as sun.misc.Unsafe. use data schema knowledge in dataset to minimize memory footprint, and attempt to leverage CPU cache instead of memory; optimize code generation for expression evaluation, reduce number of virtual function call
2. wholesale code generation fusing operators together; adopt in memory columar format such as parquet for denser storage and enable compatibility with parquet tensorflow

#### Use Tungsten
Tungsten rewrites spark operations in bytecode at runtime, able to suppress virtual functions and leverage close to bare metal performance by focusing on jobs CPU and memory efficiency.

* use Dataset instead of DataFrame or RDD; Dataset is type safe, better error handling and more readable unit test; map and filter function perform poorer. Potentially switch to Frameless
* avoid User-Defined Functions (UDF); using UDF implies deserialization to process the data in classic scala and then reserialize, can be replaced with Spark SQL builtin functions; this might not generate instant improvement but can prevent future performance issue. even if there is no builtin replacement, it is better to implement and extend Catalyst (SQL Optimizer) expression class, also avoids the serialization step. e.g.

```
def currency = udf {
  (currencySub: String, currencyParent: String) =>
    Option(currencyParent) match {
      case Some(curr) => curr
      case _ => curencySub
    }
}

//can be replaced with coalesce builtin function
```

* avoid User-Defined Aggregate Functions (UDAF); UDAF generates SortAggregate operations which is significantly slower than HashAggregate. e.g.

```
df.stat.approxQuantile("value", Array(0.5), 0) //10 times faster than finding median through UDAF
```

* avoid UDF and UDAF that perform more than one thing; would simplify testing; code would be less reusable

#### Execution Plan Optimization
* look for abnormal number of stages; bad plan can require 10 stages instead of 2-3 for basic join operations between 2 dataframes
* sending data over the network is the most expensive operation; avoid shuffle by avoiding join and groupBy (implemented by shuffle)

use `.explain(true)` command to show the execution plan detailing all steps. DAG in Spark UI can be used to visualize task repartition in each stage.

Spark SQL optimization replies on mechanic rules to optimize, in 2.3.0 a cost based optimization engine is introduced that may help with plan optimization

#### Data Management
improve performance by manage data efficiently.

* highly imbalanced datasets; review execution duration of each task and look for heterogeneous process time, if one task is significantly slower than others it will extend the overall job duration and waste resources of the fastest executor; check for min,max,median duration in Spark UI
* inappropriate use of caching; caching intermediate result could improve performance, tempting to cache a lot, but due to spark caching strategy to be memory then swap and disk, cache going to disk is slow. should stop putting pressure on caching if cache cost is more than reading a dataframe. use Spark UI to find out the amount of cache end up in disk, make sure it's not too big
* broadcasting; use broadcasting of small dataframes to all executors to avoid a shuffle. e.g.

```
auction.join(broadcast(website) as "w", $"w.id" === $"website_id")
```

the broadcast keyword allows to mark DataFrame that is small enough to be used in broadcast join

spark is supposed to figure out small dataframes for broadcast, but the truth is it needs Hive and metastore information in order to identify them, so it's not automatic

#### Cloud Related Optimizations
S3 is not a filesystem, but a object store. simple operations such as renaming copies the entire data block. use the following configuration

```
spark.hadoop.mapreduce.fileoutputcommitter.algorith.version 2
spark.speculation false
```

doing the above configuration allows files to be written progressively instead of being written as a whole at the end of the job. improve read, transformations and then write operations by factor of 2.

* use (hadoop output committers for S3)[github.com/rdblue/s3committer]

spark supports predicate pushdown with parquet (but may not always be effective), and parquet allows to limit the amount of data read from S3

