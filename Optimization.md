## Optimization Techniques

### Bypassing Cache
When data is written to memory, normally the memory is read into cache first, causing cache conflict with data needed to be read. When writing large amount of data to memory without ever reading them back, bypassing the read to cache before write can be faster. The technique require using Intel intrinsics such as `_mm_stream_si32` for writing a 32 bit integer. Intrinsics for writing larger types of data without reading to cache exist, such as writing SIMD vector types. All writing to memory intrinsics on Intel are named using the keyword stream.

Example that uses this technique is the memset function.

Combining writing large amount of data to memory can also take advantage of **write combining**, which write is delayed as one cache flush to memory. It is important to write all data in one cache line out one after another, so writing to consecutive memory locations is important.

### Sequencial Read over Random Memory Access
Reading memory sequentially allow for faster memory read due to combined cache load operation and thus fewer cache miss on read. Random memory access is often 70% slower compared to sequential memory access.

### Optimize L1 Data Cache
general idea is improving locality and align code and data.

To get an idea of the cache size for your program, use `getconf LEVEL1_DCACHE_LINESIZE` to get the size of the L1 cache in bytes

Optimize data structure in the way how it's being used, place data that's often used together close to one another to optimize for cache access. Get an idea if an object size is bigger than 1 cache line, members of the object that needs to be accessed together should go to the same cache line.

When accessing large array of objects, use object arrays instead of array of objects to remove unnecessary cache load of data not being used for a loop.

Always move the structure element which most likely be the critical word to the begining of the object.

When accessing data structures, and the order of access is not dictated by the situation, access the elements in the order they are defined in the structure.

Access aligned data is faster than accessing unaligned data. Normally the stack is tightly packed, but we can set stack alignment by using compiler option `-mpreffered-stack-boundary=3` which will align data on the stack by 2^3. i.e. SSE instructions require 16 byte alignment.

`malloc` is only aligned to `alignof(long double)`, to have custom aligned memory allocation, use `posix_memalign` in `stdlib.h`

for object allocated by the compiler, such as static objects, use `__attribute(aligned(64))` for example with the definition of the object. This works for both global variable and automatic variable. The attribute can also be used for object definition to force an alignment of instances of the object.

both methods above does not work for array of objects because each object in the array is not properly aligned.

Variable Length Array have dynamic alignment, causing code to be slower.

Instead of defining objects based on logical group, remove members of object that does not often need to be accessed help reduce junk caching during read when an array of the object is accessed frequently.

increased associativity of the cache benefits normal operation. the larger the cache the higher the associativity it is. L1D cache is too large to be fully associative but not large enough to have the same associativity as L2 cache. This is a problem when many objects of the working set fall into the same cache set. This leads to eviction of the overuse of a set, causing delay even though much of the cache is unused. These cache misses are **conflict misses**. If variables used together are also stored together, the chance of conflict miss is minimized.

E.g. L1D cache often have cache associativity of 8 and total size of 32KB.

A memory address is often divided into 3 segments based on caching: index, bank and bytes. The bytes determine which cache line to go to, and the bank is a small buffer dealing with limited number of conflict miss.

### Optimize L1 Instruction Cache
Major problems with L1i cache:
* jumps are dynamic, branch prediction try to deal with it
* X86 instruction decoding takes time, it is cached as decompressed form which is longer

Way to optimize L1i cache:
* reduce code footprint, balanced with loop unrolling and inlining
* code execution should be linear without bubbles, i.e. wait time on resources
* align code when makes sense

* compiler option `-Os` optimize for code size by disabling loop unrolling and inling. this is good if code will not be able to take advantage of those optimizations.
* compiler option `-finline-limit` specifies how large a function must be considered too large for inling.
* to force a function to be always inlined, use `always_inline` attribute
* to force a function to never be inlined, use `noinline` attribute
* code generated for conditional branch having code segment that is known to execute less often can cause bubble in L1i cache. It is possible to guide the compiler indicating which branch of the condition is often the correct one using `__builtin_expect(long EXP, long C)`, which tells the compiler the expression EXP often will have the value C. we can wrap this in following macros to be used in if condition

```
#define unlikely(expr) __builtin_expect(!!(expr), 0)
#define likely(expr) __builtin_expect(!!(expr), 1)

// using the macro
if (likely(a > 1)){...} else {...}
```

combine these macros with the compiler option `-freorder-blocks`, which is enabled with `-O2` but is disabled in `-Os`, another option `-freorder-blocks-and-partition` has limited usefulness because it doesn't work with exception handling

Intel has Loop Stream Detector (LSD) that optimize for small loops (no more than 18 instructions with no subroutine call), require up to 4 decoder fetch of 16 bytes and has at most 4 branch instructions, and is executed more than 64 times, then the loop is locked into instruction queue and will be more quickly available.

* align instruction can be useful to optimize instruction caching only in small conditions, such as begining of function, begining of basic block which is only reached through jumps, begining of a loop. aligning instruction can be achieved by inserting `nop` instructions
* aligning functions can use compiler option `-falign-function=N` which will align functions to the power of 2 greater than N. This is a waste for small functions. Function alignment can be turned off using `-fno-align-functions`
* aligning basic blocks can use compiler option `-falign-jumps=N`
* aligning loops can use compiler option `-falign-loops=N`

Machine's cache information can be found under `/sys/devices/system/cpu/cpu*/cache/` which each level is organized by the number, inside is a directory containing files usuch as `shared_cpu_map` and `size`. Cache size can be computed using the size file divide by the number of bits set in the bitmask in `shared_cpu_map` file.

### Optimize TLB Usage
we can reduce the number of pages a program has to use, or reduce the cost of TLB lookup by reducing the number of higher level directory tables which must be allocated.

Page faults are often one time cost, TLB miss is perpetual penalty given TLB cache is usually small and flushed frequently. Page faults are order of magnitude more expensive than TLB miss, but if a program is running long enough and certain part of the program is executed often enough, TLB miss can outweight page fault cost.

While page fault optimization require page-wide grouping of code and data, TLB optimization require at any given point in time, as few TLB entries are in use as possible.

The number of page directories in TLB is controlled by distribution of address range used in virtual address space. Widely varying locations in address space means more directories. This is also complicated by Address Space Layout Randomization (ASLR). The load address of stack, heap, and executable are randomized at runtime.

Only way programmer can directly affect is when an address space region is explicitly requested when using `mmap` with `MAP_FIXED`. Allocating a new memory region this way is very dangerous and hardly ever done. Programmer need to know about the last level page directory of TLB and select the address appropriately.

### Prefetching
prefetching can help to hide latency. the processor can perform prefetching on its own, triggered by certain events **(hardware prefetching)** and explicitly requested by program **(software prefetching)**.

#### Hardware Prefetching
triggered by 2 or more cache miss in a certain pattern, miss can happen preceeding or succeeding a cache line. Cache miss in strides can also be recognized, i.e. skipping a fixed number of cache lines.

It is had for performance if every single cache miss triggered hardware prefetch. Random memory access to global variables are quite common and resulting prefetch would mostly waste FSB bandwidth, which is why 2 misses are needed in order to kick off hardware prefetching. Processor expect more than one stream of memory access, tries to assign cache miss to each stream, and if threshold is reached, triggers prefetch. CPU today can keep track of 8 or 16 streams for higher level cache.

Prefetching has one weakness: *it cannot cross page boundary*. The program will experience cache miss at page boundary unless it is explicitly prefetches or read from new page.

Prefetcher also *do not recognize non-linear random access patterns*.

There is not much user can do for hardware prefetching.

#### Software Prefetching
Software prefetch API on Intel:
```
#include <xmmintrin.h>
enum _mm_hint {
  _MM_HINT_T0 = 3,
  _MM_HINT_T1 = 2,
  _MM_HINT_T2 = 1,
  _MM_HINT_NTA = 0,
};
void _mm_prefetch(void*, enum _mm_hint);
```
Processor will ignore if pointer given to `_mm_prefetch` is invalid. The instruction will load the data into cache and evict other data if necessary. Unnecessary prefetch should definitely be avoided.

the behavior of `_mm_prefetch` is implementation defined, each processor may behave differently. In general `_MM_HINT_T0` fetch data to all levels of cache for inclusive cache and to the lowest level of the exclusive cache. `_MM_HINT_T1` pulls data into L2 and not into L1, and `_MM_HINT_T2` can pull into L3 if there is one. `_MM_HINT_NTA` tells processor to treat the cache line specially. NTA stands for non-temporal-aligned. The program tells the processor that polluting cache with this data should be avoided as much as possible since the data is only used for short time. Thus the cache line is not flushed to L2 and goes directly to memory, and lower level cache will choose to evict the record.

* gcc can emit prefetch instructions for iterating over array, use the option `-fprefetch-loop-array`. it highly depends on the length of the array and may hurt performance.

#### Speculation Prefetch
Intel special instruction to begin loading from memory before memory is being used instead of blocked by memory load in order to speed up the code

To reduce code complexity by having prefetch instructions, have a separate ***helper thread*** with the sole purpose to do prefetch for the main thread. e.g. use Intel's hyper threading technology. The helper thraed will be able to move the data from memory to L2, or L1D cache. The helper thread will always be blocked prefetching memory, while the main thread uses the memory, this does not cause cache contension between multiple threads on the same core.

It is important to make sure the helper thread is in sync with the main thread. `futex` system call on linux can help prevent that, or use posix thread synchronization primitives at a higher cost.

It is possible to determine which processors the OS knows are hyperthreads using `NUMA_cpu_level_mask` API e.g.

```
#include <libNUMA.h>
ssize_t NUMA_cpu_level_mask(size_t destsize, cpu_set_t* dest, size_t srcsize, const cpu_set_t* src, unsigned int level);
```

the interface can determine the hierarchy of CPUs as they are connected by cache and memory. The interesting level is level 1, which correspond to hyperthreads. e.g.

```
cpu_set_t self;
cpu_set_t hts;
NUMA_cpu_self_current_mask(sizeof(self), &self);
NUMA_cpu_level_mask(sizeof(hts), &hts, sizeof(self), &self, 1);
CPU_XOR(&hts, &hts, &self);
```

if `NUMA_cpu_level_mask` returns 1, there is no hyperthread available.

The hyperthread prefetching will help consistently when load reaches higher than L2 size. Performance improvement is at best 25%. Code will still slow down as more memory is needed to be loaded since the bottle neck is loading into L2 cache from memory. If there is a lot of cache pollution in code, helper thread will not be very useful.

size of L2 cache can be querried using `sysconf(_SC_LEVEL2_CACHE_SIZE)`

#### Direct Cache Access
Modern hardwares can write to memory without CPU, after data is written, notifies the processor. Because the processor does not know when the data will be written, reading the memory will always be a cache miss and cannot be prefetched. This is especially a problem for network packet as the header of the packet determines how to handle the packet. ***Direct Memory Access*** in Intel chips can handle this problem: device can also cause cache to be populated with the data for the processor that's going to handle it.

DCA reduces the number of cycles to handle network IO on architectures that have it.

### Multi-Thread Optimizations

### Memory Performance Tools

#### Reference
[LWN](https://lwn.net/Articles/255364/)
