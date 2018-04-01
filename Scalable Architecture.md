### Scalable Architecture ###

2 aspects: **Decentralization** and **Independence**

More decentralization incrase scalabilty, but also increase the cost of communication. e.g. a centralized scheduler can determine the optimial schedule, but have high communication cost, whereas a decentralized scheduler reduce cost of communication but cannot find optimial schedule.

algorithms allowing for less dependency increase scalability. e.g. *multi-version concurrency control* using *differential reference counting*

generally should use `K * P` threads, `P` is the number of processors, and K is subscription coefficient, usually in the value between 1 to 4. A system should not make the number of threads based on the data size or transaction rate, or subsystem plugin count. Doing that would lose control of subscription level.

a thread is not an application-level abstraction, but an abstraction of a processor so don't create threads to do particular funciton. let any thread in the program to be able to execute any function. Don't create threads only to do IO.

should have work distribution/balancing mechnism to make sure all threads are utilized evenly. should have dynamic feedback process.

don't extensively use mutexes and other explicit or implicit forms of mutual exclusion. mutex are anti-thread, they are not means for concurrency but means to supress concurrency. 

eliminate mutable shared state when possible.



Reference:
(Reference)[http://www.1024cores.net/home/scalable-architecture/introduction]
