### Scalable Architecture ###

2 aspects: **Decentralization** and **Independence**

More decentralization incrase scalabilty, but also increase the cost of communication. e.g. a centralized scheduler can determine the optimial schedule, but have high communication cost, whereas a decentralized scheduler reduce cost of communication but cannot find optimial schedule.

algorithms allowing for less dependency increase scalability. e.g. *multi-version concurrency control* using *differential reference counting*

generally should use `K * P` threads, `P` is the number of processors, and K is subscription coefficient, usually in the value between 1 to 4. A system should not make the number of threads based on the data size or transaction rate, or subsystem plugin count. Doing that would lose control of subscription level.

a thread is not an application-level abstraction, but an abstraction of a processor so don't create threads to do particular funciton. let any thread in the program to be able to execute any function. Don't create threads only to do IO.

should have work distribution/balancing mechnism to make sure all threads are utilized evenly. should have dynamic feedback process.

don't extensively use mutexes and other explicit or implicit forms of mutual exclusion. mutex are anti-thread, they are not means for concurrency but means to supress concurrency. 

eliminate mutable shared state when possible.

#### Task Scheduling Strategies
1. **work-stealing**. reactive asynchronous strategy. when a thread is out of work, it randomly chooses a victim thread and asynchronously tries to steal work from it
2. **work-requesting**. reactive synchronous strategy. when a thread is out of work, it randomly chooses a victim thraed and send synchronous request to it, victim receives the request, send work back
3. **work-distribution**. proactive synchronous strategy. during submission of a new work, it's divided and proactively distributed to some threads (idle or lightly loaded)
4. **work-balancing**. practive asynchronous strategy. dedicated thread periodically collect information about load of all worker thread, calculate optimal distribution of work, re-distribute work among them

it is possible to use multiple or all of the above strategies for scheduling. reactive strategies have limited local information and makes sub-optimal decisions. proactive strategies have information regarding information state, makes one-shot optimal scheduling decisions but unable to cope with inevitable dynamic load balancing.

scheduler must employ at least one reactive strategies to cope with continuous and inevitable dynamic load imbalance, and optionally include one or both proactive strategy in order to cut down stealing/requesting cost.

the general recipe for scheduler is `SCHEDULER = (STEALING ^ REQUSTING)[+DISTRIBUTION][+BALANCING]`

the advantage of work stealing is its asynchronous nature where the thief thread is able to work while victim thread is busy. but it also have 2 problem. One is it requires the worker work queue to synchronize due to potential asynchronous operation from another thread regardless of how often it happens. Two is the join phase of parallel algorithm. Traditional handling of task completion involve decrement of pending child counter in parent task. Due to asynchronous nature of work-stealing, decrement has to be synchronized with other potential concurrent decrement operation.

work-balancing is more cumbersome to implement compared to work-distribution. it can be useful for systems with badly structured work DAGs and with unpredictable load. e.g. Erlang scheduler employs work-balancing.

#### Multilane : a concurrent blocking multiset
goal is to improve the scalability of a producer-consumer system by means of partitioning. Wrap any producer-consumer queue in order to get a queue with the same properties but better scalability.

multilane is composed of multiple queues all have the same property, a *input ticket demux* and a *output ticket mux*. there is no restriction of underlying queue, and it can be bounded or unbounded. the input demultiplexer and the output multiplexer are no more than atomic variables, and tickets are obtained with `atomic_fetch_add(1)`

```
void multilane_enqueue(multilane_t* m, void* data)
{
    unsigned ticket = atomic_fetch_add(&m->enqueue_cursor, 1, memory_order_relaxed);
    underlying_queue_t* queue = &m->queues[ticket % m->count];
    underlying_enqueue(queue, data);
}

void* multilane_dequeue(multilane_t* m)
{
    unsigned ticket = atomic_fetch_add(&m->dequeue_cursor, 1, memory_order_relaxed);
    underlying_queue_t* queue = &m->queues[ticket % m->count];
    return underlying_dequeue(queue);
} 
```
Intel TBB concurrent queue uses the same trick.

It increases performance using partitioning. but there is no way to make a centralized heavy contended data structure scalable. Instead, this should be solved in the architecture level. e.g. per-thread pools, where a thread pushes and pops resources to its own pool, only to query randomly other pool if its own pool is empty. alternative design is to use per-thread pool along with centralized one which is used solely for resource surplus. that is if per-thread pool reaches a threshold, some resource are offloaded into the centralized pool, and thread is out of resources it queries the centralized pool.

in producer-consumer scenario, we can introduce notion of persistent binding between producers and consumers. each producer has a SPMC queue (single producer, multiple consumer queue). each consumer has one or several associated producers, and checks their queue first. if they are empty then fall back to random checking of other queues.

it's a bad idea to divide threads into producers and consumers, it's better idea to have just worker threads and let them decide to do whatever most important. so a thread can switch to producer and push jobs to its own queue, then switch to consumer and consume message on the same queue.

never use unbounded queues in producer-consumer scenario. a bounded queue behaves the same as an unbounded one while everything goes as supposed, and no severe performance degradation or OOM.

#### Multi-version Concurrency Control
eliminate starvation. it allows several versions of an object to exist at the same time. readers acquire the current version and work with it as much as they want. writer can create and publish a new version of an object, which becomes the current. readers will still work with previous version and can't block/starve writers. when readers end with an old version of an object, it goes away.

##### Differential Reference Counting
reference counting with strong thread-safety. *strongly thread-safe object* permit concurrent read and write access, while *basically thread-safe object* permit either concurrent read accesses or exclusive write access.

structure of differential reference counter:
```
{counter + ptr} -> {strong_counter | basic_counter | data}
```

vs plain old reference counting:

```
{ptr} -> {counter | data}
```

an object can be deleted when both inner counters (strong_counter and basic_counter) are equal to 0. Outer counter is used as transient cache. it collects all +1 while inner counters collects -1, thus the same differential. when a pointer is updated with a new value, outer counter value is transferred to the inner counter.

differential reference counting features 2 types of pointers: a strong pointer is one that supports arbitrary concurent access, that is one thread acquires/copies a pointer, and another thread updates the pointer. a basic pointer is one that is thread-safe as int, supports either concurrent acquire/copy exclusive update.

the inner strong counter is the number of strong pointers to the object. the inner basic counter is the number of usual references to the object.

(implementation reference)[http://www.1024cores.net/home/lock-free-algorithms/object-life-time-management/differential-reference-counting/implementation]

#### Optimistic Concurrency Control
makes logically read-only access physically read-only and eliminate writer starvation caused by reader. a reader starts reading an object without any synchronization, and when it finishes it verifies that the object was not changed under its feet. if object was not changed then it has obtained some consistent view of the object, otherwise retry. the technique gives very good result with respect to scalability in many cases. however, a reader must be prepared to read inconsistent data, which can cause crash, infinite loop.

### Producer-Consumer Queue
properties:
1. allowed number of producer and consumer threads: MPMC, SPMC, MPSC, SPSC
2. underlying data structure: array based, linked list, hybrid
3. intrusiveness for linked-list based queues
4. maximum size for linked-list based queue
5. overflow behavior: fails on overflow or overwrites the oldest item on overflow
6. requirement on garbage collection
7. support for priorities
8. ordering guarantees: casual FIFO/LIFO, per-producer FIFO/LIFO, best-effort FIFO/LIFO or no ordering guarantee
9. forward progress guarantee for producers: waitfree producer, lockfree producer, blocking producer
10. forward progress guarantee on consumers: waitfree consumer, lockfree consumer, blocking consumer
11. expected usage: queue usually contains very few or zero messages or queue contains substantial amount of message
12. behavior on failure: when a queue is empty/full consumers/producers get blocked, or instantly get false

##### Queue Catalog
- bounded MPMC queue (reference)[http://www.1024cores.net/home/lock-free-algorithms/queues/bounded-mpmc-queue]
- intrusive MPSC node-based queue (reference)[http://www.1024cores.net/home/lock-free-algorithms/queues/intrusive-mpsc-node-based-queue]
- non-intrusive MPSC node-based queue (reference)[http://www.1024cores.net/home/lock-free-algorithms/queues/non-intrusive-mpsc-node-based-queue]
- unbounded SPSC queue (reference)[http://www.1024cores.net/home/lock-free-algorithms/queues/unbounded-spsc-queue]

#### Reference
(Reference)[http://www.1024cores.net/home/scalable-architecture/introduction]
