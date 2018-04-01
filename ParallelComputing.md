### Cache-Oblivious Algorithms
efficient usage of processor cache and reduction of memory bandwidth requirements. memory bandwidth is usually shared between hardware threads and frequently becomes a bottleneck for scalability. e.g. write sharing, false sharing. cache-oblivious algorithms are oblivious of particular cache hierarchy and cache parameters and make efficient use of whatever cache hierarchy/parameters, but not oblivious to the prescence of cache.

we assume "cache" is fast and private and "memory" is slow and shared, and optimize access to slow shared memory. this model would consider L3 cache as "memory" because it is shared across cores.

cache oblivious algorithm work by recursively dividing problem's dataset into smaller parts and then doing as much computations on each part as possible. eventually subproblem dataset fit into cache, and we can do significant amount of computations on it without accessing memory. we don't need to know exact cache hierarchy/parameter for it to work, we just recursively divide a dataset, and it inevitably eventually fits L3 cache, then L2, then L1, such that it automatically optimally exploid cache hierarchy.

e.g. matrix multiplication, divide each matrix into 4 partitions recursively until each partition is smaller than L1 cache size. decompose it into into 

```
A1 | A2     B1 | B2   A1 * B1 + B2 * B3 | A1 * B2 + A2 * B4
---|---  *  ---|--- = ------------------|-------------------
A3 | A4     B3 | B4   B3 * B1 + A4 * B3 | A3 * B2 + A4 * B4
```

and rearrange each input matrix before multiplication optimally. e.g. A = |A1|A2|A3|A4|

### Concurrent Skip List
there is no recipe for scalable concurrent data structures, need to analyze the usage pattern for the data structure, then try to satisfy all user requirements and scalability prerequisites with all possible means.

prerequisites for scalability:
1. no mutexes on fast-path ever. mutexes sacrifice concurrency to provide simplicity, provote write sharing on every operation
2. logically read-only operation must be implemented as a physically read-only operation. so there is no write hidden inside the read such as reader-writer mutex writing to internal state
3. no writes to a centralized shared state on fast-path. writes to shared state are generally unavoidable, but we can distinquish 4 kinds of shared state:
    - mostly private state. no danger for scalability
    - mostly read-only state. with high read-to-write ratio. no danger for scalability
    - decentralized shared state. shared state frequently being written to, but is physically distributed. e.g. hash map. may or may not represent danger for scalability depending on distribution factor, number of threads, access patterns
    - centralized shared state. frequently written to, physically centralized. no way to make it scalable. need to avoid
4. beware of false sharing, where 2 variables are combined into a single cache line, where they look like a single varaible with all implications on scalability. size of cache line is architecture dependent. 
5. atomic RMW operation have some fixed associated costs. e.g. x86 atomic RMW operation cost around 40 cycles.
    
reads to a shared state have 100% scalability while write to a shared state have 0 scalability.

concurrent skip list use `find_or_insert()`. attempt to insert an already present item is logically read-only operation, attempt to insert an absent item is logically mutating operation.

##### Lock-free reader pattern
make data structure consistent and available for reading all the time, regardless of the way of implementing mutating operations. mutation operations can synchronize with each other by means of mutex, each mutation operation must have a linearization point with regard to readers, after which the operation take all effect and is completely visible for all readers.

if there are concurrnet remove operations, then some form of PDR (**partial copy-on-write deffered reclamation**) must be employed in order to ensure that reader can read/traverse a data structure regardless of concurrent removal.

##### node count maintenance
node level in skip list is chosen randomly, with level 0 with probability 2^-1, level 1 2^-2, with certain maximum level either fixed to a value or scale based on number of total elements in list. Maintaining a node count as a centralized counter variable destroys scalability.

a decentralized count algorithm requires each thread to have a private counter, incremented after each insert. counters are periodically aggregated to produce a total node count. period of aggregation is based on estimation of when total node count will cross next 2^N mark.

##### memory allocation
memory allocation must proceed in parallel. memory consumption must consume memory optimally. use per thread **slab allocator**.

### Radix Sort
two radix sort: LSD (least significant digit) radix sort and MSD (most significant digit) radix sort. MSD radix sort can be parallelized. each subdivisions can be sorted independently of the rest.

2 parallelizations: intra-radix paralleization where input data is split into several parts and each processor picks up a part and makes radix split (parallel processing). when all parts have split partial radix arrays are aggregated (join) and directed to the next level of recursion. this help with sorting of not-so-randomly distributed data. inter-radix parallelization sort whole array on lower levels of recursion. this parallelization help mitigate overhead of thread synchronization. 

parallelization is guided at runtime. threads prefer to do inter-radix parallelization, howver if some threads are out of work they help other threads on intra-radix level.

when size of input array reaches some threshold, thread switches to single-threaded mode and no futher sub-task are split.

### Reference
(1024core)[http://www.1024cores.net/home/parallel-computing/cache-oblivious-algorithms]
