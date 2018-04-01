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

Reference:
(Reference)[http://www.1024cores.net/home/scalable-architecture/introduction]
