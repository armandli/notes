### General Optimization Techniques used in JVM
1. method inlining - avoid maintenance code during method call
2. loop unrolling - unroll for loops
3. escape analysis - determine if function have side effect, if there is none, allocate variables on stack
4. monomorphic dispatch and dimorphic dispatch - de-virtualization technique, making virtual functions calls non-virtual when there is only 1 or 2 options
5. intrinsic - convert JVM assembly code into machine specific assembly instruction if available, architecture specific
6. on stack replacement (OSR) - replace a stack frame with an optimized version

#### OSR
converts a running function's interpreter frame with a JIT'ed frame in the middle of the method. this happens when a function is running for long, JVM will consider it hot.

#### Loop Optimizations
* loop predication - move constant operations out of the loop
* array filling - pattern match array initilization with call to assembler stub
* range check elimination
