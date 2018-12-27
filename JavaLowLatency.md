## Low Latency Optimization in Java

limit of latency in different programming methodology:
Java and C/C++ : 10 us
FPGA: 1us
ASIC: 400ns or less

levels of latency optimization in Java:
1) to reach seconds response time, use small methods, minimize branching, use cohesion, abstract cleanly
2) to reach 100ms response time, optimize data structures, algorithm complexity, use batching and caching
3) to reach 10ms response time, optimize memory access patterns, CPU cache, use lock free algorithm, asynchornous processing, stateless, RamFS/TmpFS, GC and object lifecycle tuning
4) to reach 1ms or under, optimize thread afinity, NUMA, use large pages, avoid false sharing, data oritented design, disable c-states, ensure CPU cache friendly operations, no GC, 0 copy

for low latency Java programming, GC is not allowed. how to not create garbage? create API such that we don't create temporary. use mutatable data structures with interfaces that does not create temporary. e.g. call to map with key that is not a temporary object, instead, take parameters used to create the key, and reuse the same key object to search with replacable fundamental type values (include string too)

avoid using String class (because it creates temporary), instead, work with CharSequence i.e. class StringView implements CharSequence. CharSequence can take a buffer and use set method to set where the string begins in the buffer, with no object being created. example:

```
public void countUniqueWord(CharSequence word){
  //specially written map that does not create new objects
  ObjLongMap<CharSequence> wordToCount = HashObjLongMaps.<CharSequence>getDefaultFactory().withKeyEquivalence(charSequence()).newMutableMap();
  if (wordToCount.containsKey(word) != null)
    wordToCount.addValue(word, 1);
  else
    wordToCount.put(word.toString(), 1);
}
```

contrary to fundamental theory, memory access is always slower than CPU operations, this means an algorithm that rely more on complex CPU computation for a value could be faster than caching the expensive computation result using a highly efficient data structure. even though from big-O notation the CPU computation looks worse, the reality is CPU intensive computation sometimes look almost like it's free in comparison to memory access.

### determinism vs. absolute latency
* hard real time such as pacemater or weapons system, require a best worst bound
* first to the bell such as a cycling team, does not matter which is fastest, just one being fast is good, opposite of hard real time
* web real time, user does not notice the delay
* soft real time, care about all latencies, but can sacrifice the very furthest outliers for a better mean, which is most trading systems

### coordinated omission
when we measure we tend to put a timing statement in the first and last line of the code, run some iterations and average. should make sure to measure need to start measuring when the process should have started, not when it is started, if we do that we miss a point, example train station, should measure from when train should have left the station isntead of when train actually left station.

document latency requirements. should write the latency requirement as a real contract

only work in my circle of influence. work out what i can be responsible work. benchmark the minimum application using the framework, this being the null hypothesis, the bare minimum it could possibly achieve. if I am responsible for the framework, then need to benchmark the operation system.

### Micro Service for Low Latency Programming
* can build low latency microservices and can be efficient, need fast transport, no TCP, need shared memory. use low latency transport protocols such as Chronicle Queue and Aeron.
* use single thread, use small JVM, because there is no synchronization, no complicated object pooling. scale out by partition data and add more microservices.
* hog the CPU, and CPU pin.
* record input and output, so it can be replayed.
* map microservices to hardware, should not cross NUMA region. make sure no memory access cross NUMA regions.

### Hardest areas in low latency systems
* wait free off heap data structures. it means not only the data structure is lock free, it also will not put thread to sleep and ensure process is done in guarateed time before thread goes to sleep.
* resilience and high availability
* failover and hot-hot
* measurement
