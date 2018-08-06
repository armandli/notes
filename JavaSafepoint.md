### Java Safepoint
a safepoint is a state of your application execution where all references to objects are perfectly reachable by VM. some operations of the VM require that all threads reach a safepoint to be performed. Most common operation that need it is GC.

a safepoint means that all threads need to reach a certain point of execution before they are stopped. then the VM operation is performed. then all threads are resumed.

#### VM operations
when GC occurs, a safepoint is requested. then all threads are stopped. GC threads can perform their algorithm. but GC is not the only operation that requires a safepoint. here is some other operations that do:
* Deoptimization
* PrintThreads
* PrintJNI
* FindDeadlock
* ThreadDump
* EnableBiasLocking
* RevokeBias
* HeapDumper
* GetAllStackTrace

if running the application with following JVM options:
`-XX:+PrintSafepointStatistics -XX:+PrintGCApplicationStoppedTime -XX:PrintSafepointStatisticsCount=1`

in Oracle/openJDK assembly, safepoints are marked with {poll}

safepoint status check is implemented in cleverly. Normal memory variable check would require expensive memory barriers. Though, safepoint check is implemented as memory reads a berrier. Then safepoint is required, JVM unmaps page with that address provoking page fault on application thread. This way HotSpot maintains its JITed code CPU pipeline friendly, yet ensures correct memory semantic where page unmap is forcing memory berrier to processing cores.

If to look at safepoints in assembly of the same process, the would all follow the same pattern of pointing at the same global magic address via local relative trick. setting the magic page to protected will trigger a SEGV for all threads. the effective cost of this global safepoint approach goes up the more runnable threads in the application.

example instruction: `test DWORD PTR[rip+0xa2b0966],eax; {poll}` where rip is the relative instruction pointer

some JVM safepoints can be threadlocal, e.g. Zing

all java profilers have sample option to sample only at safe points. this leads to safe point bias. the profiler should sample all points in a program with equal probability, but with safe points sampling this is not achieved.
