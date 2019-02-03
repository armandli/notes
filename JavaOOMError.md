### Explanation for various Java OOM Error messages
#### Error Messages
* java heap space: object could not allocate in java heap, this could mean memory leak, exessive use of finalizers (finalizers are not GCed). solution: fix leak,
    increase heap size by -Xmx
* GC overhead limit exceeded: java process spending more than 98% of its time doing GC and recovering less than 2% of heap and has been doing so far the last 5
    consetive garbage collections. solution: increase heap size -Xmx option, this error can be turned off using option: -XX:-UseGCOverheadLimit . it could also be
    memory leak, so fix it if it is
* Requested array size exceeds VM limit: application attempt to allocate an array that is larger than heap size. solution: increase heap size -Xmx option, redesign
    the application not to have large arrays
* Permgen space: permgen space contains:
  * names, fields, methods of classes
  * object arrays and type arrays associated with class
  * just in time compiler optimizations
  the error is generated when permgen space ran out. solution: increase Permgen size: -XX:MaxPermSize. Often application redeployment without restarting can cause
  this issue, so restart JVM
* Metaspace: from Java8, Permgen space is replaced by Metaspace. Class metadata is allocated in native memory. error is thrown if metaspace ran out. solution:
    -XX:MaxMetaSpaceSize to increase metaspace size to something higher than default. Reduce the size of java heap help increase metaspace. allocate more memory for
    the JVM. Could also be bug in application that is causing metaspace leak
* Unable to create new native thread: there is no sufficient memory to create a new thread. it indicates there is no sufficient native memory space. solution:
    allocate more memory to the machine ,reduce java heap space, fix thread leak in the application, increase the limits at OS level such as using ulimit -a max
    user process; reduce thread stack size with -Xss parameter.
