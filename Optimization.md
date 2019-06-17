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

## Performance Speed Limits
#### Pipeline Width
intel: maximum 4 fused-uops per cycle
AMD:   maximum 5 fused-uops per cycle
every CPU can execute only a maximum number of operations per second. modern pipeline superscalar processors can execute more than 1 instructions per cycle, up to a
limit. the underlying limit is not always imposed in the same place, some CPU may be limited by instruction encoding, others by register renaming or retirement. on
modern Intel CPU, if a loop contains 4 fused-uops, it will never execute at more than 1 iteration per cycle. example:

```
uint32_t top = 0, buttom = 0;
for (size_t i = 0; i < len; i += 2){
  uint32_t elem;
  elem = data[i];
  top += elem >> 16;
  buttom += elem & 0xFFFF;

  elem = data[i + 1];
  top += elem >> 16;
  bottom += elem & 0xFFFF;
}
```

compare and jump pair of instructions are macro-fused into a single uoup. if there are 14 uops, then this loop will take 3.5 cycles per iteration, or 1.75 cycles
per element.

this is the best it can be, only way to make it better is to reduce the number of instructions. we can also do vectorization to improve the performance.

we can also look for micro-fusion on x86 machines for performance. we can look for opportunities to fuse a fold a load and an ALU operation together since such
micro-fused operation only count as one in fused domain. this generally applies only for values loaded and used once, but rarely it may even be profitable to laod
the same value twice from memory in 2 different instructions in order to eliminate a standalone mov from memory

#### Port/Execution Unit Limit
Intel, AMD: one operation per port per cycle
example:
```
uint32_t mul_by(const uint32_t* data, size_t len, uint32_t m){
  uint32_t sum = 0;
  for (size_t i = 0 i < len - 1; ++i){
    uint32_t x = data[i], y data[i + 1];
    sum += x * y * m * i * i;
  }
  return sum;
}
```

despite the source containing 2 loads per iteration, the compiler was clever enough to reduce to one since y in teration n becomes x in iteration n + 1, so it saves
the load value in a register across iteration.

however, the instructions here are not 10 uops or 2.5 cycle per iteration, the number of cycle is 4. this is because the instruction `imul` can only be issued every
cycle because there is only a single scalar multiplication unit on the CPU.

on modern intel, simple integer arithmetic, bitwise operations and flag settigns run on four ports, so there is no port bottleneck for those instructions, but not
`imul`. more advanced bit operations like `popcnt` and `tzcnt` execute only on p1, shift instructions and bit test/set operations like `bt`, `btr` execute only on
p1 and p6.

there are also only 3 vector ports, so at best only 3 vector operations per cycle, and for AVX-512 there are only 2 ports so best case is 2 per cycle. only a few
operations can use all 3 ports, many are restricted to one or two ports. in particular, shuffle only on p5 and can be a bottleneck for shuffle heavy algorithms

we can see port pressure using different tools:
Intel IACA
RRZE-HPC OSACA
LLVM-MCA

we can measure port pressure using `perf` command under the report uops_dispatched_port counters. when a port column has 4.00, it means that port is occupied fully
during the process and is the bottleneck port.

we can try to replace instructions with other instructions to reduce the port pressure if possible, even if the replacement generates more instructions. example:
shuffle can be replaced with blend operations, blend operation uses port p0 and p1.

#### Load Throughput Limit
Intel, AMD: 2 loads per cycle
CPUs have a maximum load per cycle, which you can achieve if both loads hits in L1 cache. If L1 is missed due to for example bank conflict, then we cannot achieve the limit. It's possible for half of the instructions to be loads while still running at maximum speed, so load that hits L1 cache are not at all that expensive compared to simple ALU.

the loads has to be mostly independent, since otherwise the load latency will limit more than throughput.

note gather instructions count one against this load through throughput limit for each element they load, e.g. `vpgatherdd` instruction count as 8 loads

#### Split Cache Lines
on intel, all loads that hit L1 cache count as 1 except for loads that split cache line, which count as 2. split cache line load is of two bytes and crosses a 64
byte boundary. if loads are naturally aligned, then split cache line load never happens. if alignment is random, then 5% chance load a 32 bit split 5% of the time,
load a 256-bit AVX load splits 48% of the time and AVX-512 splits 98% of the time.

on AMD, load suffer penalty when crossing any 32 byte boundary, such load also count as 2 against the load limit. 32-byte AVX load also count as 2 on Zen1 since
implemented vector path is only 128-bit, so 2 loads are needed. any 32-byte load that is not 16 byte aligned counts as 3, since exactly one of the 16 byte halve
will cross a 32 byte boundary.

the remedy is reduce load. 

#### Memory and Cache Bandwidth
load and store that consistently hit cache line of certain level can be bound by the cache line cycle limit, or cache lines per cycle.

| Microarchitecture | L2    | L3       |
|===================|=======|==========|
| CNL               | 0.75  | 0.2-0.3  |
| SKX               | 1     | 0.1      |
| SKL               | 1     | 0.2-0.3  |
| HSW               | 0.5   | 0.2-0.3  |

memory bandwidth is more complicated. theoretical values are based on memory channel count, but this is complicated by the fact that many chips cannot reach the
maximum bandwidth from a single core since they cannot generate enough requests to saturate DRAM bus due to limit fill buffers.

to rememdy this, pack data structures more tightly, try to ensure locality of reference and prefetcher friendly access patterns, use cache blocking.

#### Carried Dependency Chains
example:
```
for (size_t i = 0; i < len; ++i){
  uint32_t x = data[i];
  product *= x;
}
```

we know this loop should be 1 cycle per iteration because of the multiplication, but the reality is 3 cycles. turns out the `imul` instruction here takes 3 cycles,
not 1. this is because the result of the multiplication depends on the result from previous multiplication, and every multiplication can only start when the
previous one finishes, so 3 cycles is the speed limit of the loop. this is also because the instruction is using only a single register. for both input and output

most of the loops have dependency chains, and the speed limit is set by the longest dependency chain.

there could also be overlaps between loops, the time saving by the overlap is limited by out-of-order buffer structures in the CPU, in particular the re-order
buffer and the scheduler.

tools for dependency chain analysis: IACA, OSACA, llvm-mca

to remedy this, we have to shorten or break up the dependency chain. use lower latency dependency instructions like addition, shift, instead of multiplication. turn
one long dependency chain into several parallel ones. example:

```
uint32_t p1 = 1, p2 = 1, p3 = 1, p4 = 1;
for (size_t i = 0; i < len; ++i){
  p1 *= data[i+0];
  p1 *= data[i+1];
  p1 *= data[i+2];
  p1 *= data[i+3];
}
uint32_t product = p1 * p2 * p3 * p4;
```

this loop runs at 1 cycle per ieration the latency chain speed limit is removed. often compiler can do this, but not always. gcc is reluctant in unroll loops at any
optimization level, and unrolling loops is often a prerequisite for this transformation.

#### Front End Effects
the front end effects being a limiting factor has been dropped a lot since the addition of better decoders and uop cache.
absolute front end limits to delivered uops/cycle depending on where the uops are coming from:
| Architecture | Microcode (MSROM) | Decoder (MITE) | Uop Cache (DSB) |
|==============|===================|================|=================|
| <= Broadwell | 4                 | 4              | 4               |
| >= Skylake   | 4                 | 5              | 6               |

these arn't very important, however, because they are all equal to or larger than the pipeline limit of 4, so these can be ignored.

the more important limitation are specific to the individual sources. for example:
  * legacy decoder (MITE) can only handle up to 16 instruction bytes per cycle, any instruction longer than 4 bytes decode throughput will necessarily be lower than
      4
  * only 1 of the 4 or 5 legacy decoders can handle instructions which generate more than 1 uop, so a series of instructions which generate 2 uops will decode at 1
      per cycle
  * only one uop cache entry can be accessed per cycle, it means any loop that crosses a uop cache boundary will take 2 cycles since 2 uop cache entries are
      involved
  * instructions which use microcode, such as gather have additional restrictions and throughput limitations (pre skylake)
  * the LSD suffers from reduced throughput at boundary between one iteration and the next, although hardware unrolling reduces the impact of the effect. LSD is
      disabled on most recent CPU due to bug.

#### Stores
1 store per cycle
this limit applies also to vector scatter instructions where each element count as one against this limit. stores across cache lines count as 2, but unaligned store count as 1 on Intel but AMD is more complicated and the penalties for stores that cross boundary is larger. should avoid boundary crossing stores on AMD

if repeatedly storing the save value to the same location, may be beneficial to check if the value is different. only do the store if different. this replaces a
store with load. use vectorized stores as much as possible, can do 8 32 bit stores in one cycle with a single vectorized store. try to store contiguously.

#### Complex Addressing Limit
intel: max of 1 load concurrent with a store with complex addressing per cycle

it looks like it's possible to do 2 loads and 1 store per cycle on intel. this is mostly true unless complex addressing mode is used. load and store operation need
address generation which happens on the AGU (address generation unit). there are 3 AGUs on modern Intel chips: p2, p3, p7. however, p7 is restricted and can only be
used by stores, and it can only be used if the store addressing mode is simple. simple addressing is anything that is of the form [base_reg + offset] where offset
is in [0, 2047]. we cannot execute more than 2 complex addressing load and store per cycle.

to remedy this, use simple addressing modes, use incrementing pointer by size of element rather indexed addressing modes.

example:
```
void sum(const int* a, const int* b, int* d, size_t len){
  for (size_t i = 0; i < len; ++i)
    d[i] = a[i] + b[i];
}
```

would be limited by complex addressing and runs at 1.5 cycles per iteration, since there are 1 store that uses complex addressing and one load. instead, use
separate pointers for each array and increment all of them, or just transform the store into simple addressing:

```
void sum2(const int* a, const int* b, int* d, size_t len){
  int* end = d + len;
  ptrdiff_t a_offset = (a - d);
  ptrdiff_t b_offset = (b - d);
  for (; d < end; ++d){
    *d = *(d + a_offset) + *(d + b_offset);
  }
}
```

this is undefined behavior all over the place if you pas in arbiturary arrays, because we subtract unrelated pointers and use pointer arithmetic which outside of
the bounds of the original array. alternative is to use `uintptr_t` which is unspecified instead of undefined behavior.

also loop unrolling helps.

#### Taken Branches
intel: 1 per 2 cycles with exceptions

instruction tables claim one taken branch can be executed per cycle, however, this is only true for very small loops with backward branch. for larger loops or
forward branches, the limit is 1 per 2 cycles. so it's best to arrange code to have a most likely taken path with no branches.

#### Out of Order Limits
these are limits that affect the effective window over which processor can reorder instructions. these limits all have the same pattern: in order to execute out of
order instructions, the CPU needs to track in-flight operations in certain structures. if any of these structures become full, the effect is the same, no more
operations are issued until space in that structure is freed. already issued instructions can still execute, but no more operations will enter the pool of waiting
ops.

in general, the out-of-order window which is roughly the number of instructions/operations that can be in progress, counting from the oldest in-progress instruction
to the newest.

note that the size of the window is not a hard performance limit in itself, you can't use it to directly establish an upper bound on cycles per iterations, but use
it for analysis to refine the estimate.

if a loop is 1000 instructions per iteration, the out-of-order window is only 100 instructions, the CPU will not be able to overlap the much of each iteration at all. the different iterations are too far apart in instruction stream to significant overlap.

dynamic instruction stream - the actual stream of instructions seen by the CPU. this is opposed to the static instruction stream, which is the series of
instructiosn as they appear in the binary. inside a basic block, static and dynamic instruction streams are the same, the difference is that the dynamic stream follows all jumps, so it is a trace of actual execution. when we think about the out-of-order window, we think in terms of dynamic instruction stream. 

#### Reorder Buffer Size
the largest and most general out of order buffer. all uops, even including nop or zeroing idios, take a slot in ROB. this structure holds from the point which
instructions are allocated until they retire. this is a hard upper limit in out-of-order window. ROB on Intel holds micro-fused ops, so size is measured in fused
domain.

example, load instruction take a cache miss and cannot retire until miss is complete. the load takes 300 cycles to finish (typical latency), on Haswell there is 192
ROB, so at most 191 additional instructions can execute while waiting for the load. when ROB is exhausted, the core stalls, this puts an upper bound on the maximum
instructions per cycle of the region to 192 / 300 = 0.64. it also puts a maximum memory level parallelism achievable, since only loads that appear in the next 191
instructions can execute in parallel with the original miss.

tool: github robsize to experimentally determine the ROB size.

if hitting ROB size limit, it's better to switch from optimizing by reducing the number of uops, e.g. use slower instruction that can be used to replace 2 instructions which would otherwise be faster. reorganizing the instruction stream can help too: if hit ROB after a specific long latency instruction, use more expensive instructions into the shadow of that instruction so they can execute while the long latency instruction executes, so less work to do when instruction completes, or jam load the miss together, rather than spreading them out where they would naturally occur, putting them close to one another allows for more of them to fit in the ROB window.

software prefetching can help, prefetch can retire before the load complete, so there is no stalling

#### Load Buffer
every load operation needs a load buffer entry. this means out-of-order window is limited by the number of loads appearing in the window. typical load buffer size
seems to be about 1/3 of ROB size. so more than about one out of 3 operations is a load, you are more likely to be limited by the load buffer.

gather need as many entries as there are loaded elements to laod in the gather. sometimes loads are hidden, such as instruction `pop`. in general, anything that
execues an op on p2 or p3 which is not a store needs an entry in the load buffer.

remedy is fewer loads, or order loads in relative to other instructions so taht the window implied by the full load buffer contains the most useful instructions.
can combine narrower loads with wider ones. ensure to keep values in registers as much as possible, and inline functions that would otherwise pass arguments through
memory to avoid pointless loads. consider spilling register to xmm or ymm vector registers rather than the stack

#### Store Buffer
unlike load, nobody is waiting on a store to complete, except in the case of store-to-load forwarding. you can have long store misses, but they happen after the
store has already retired and is sitting on the store buffer. so store only cause a problem when there are enough of them such that the buffer fills up. store
buffers are often smaller than load buffers.

to rememdy, use less stores, ensure not unnecessarily spilling to the stack, not doing dead stores, such as zeroing a structure before immediately overwriting it
anyway. giving compiler more information about array of structure alignment helps it merge stores.

vectorization of loops with consecutive stores help a lot since it can turn 8 32 bit stores into a single 256 bit store, which only takes one entry in the store
buffer.

scatter operation on AVX-512 don't really help.

#### Scheuler
after op is issued, it sits in the scheduler until it is able to execute. this is often smaller than the ROB, about 40-90 entries. no more operation can issue if
this fills up, which can occur if there are too many instructions dependent on earlier instructions which haven't completed yet. this could happen if a load misses
a cache, and many instructions are dependent on that load, and those instructions won't leave the scheduler until the load is complete.

to remedy, organize the code so that there are some independent instructions to execute following long latency operations, which don't depend on the result of those
operations. consider replacing data dependencies with control dependencies, since the later are predicted and don't cause a dependency. this also has the advantage
of executing many more instructions in parallel, but leads to branch mispredictions.

#### Register File Size Limit
every instruction with a destination register requires a renamed physical register, which is only reclaimed when the instruction is retired. this registers come
from the physical register file (PRF). to fill the entire ROB with operations that require a destination register, you need a PRF as large as the ROB. in practice,
there are 2 separate register files on Intel and AMD chips, the integer registers file used for scaler registers and the vector register file used for SIMD
registers, and the sizes of these register files as shown above are somewhat smaller than ROB.

not all registers are actually available for renaming: some are used to store the non-speculative values of the architectural registers, or other purposes. so it's
about 16-32 less than the values shown above. 

tool: robsize tool

the upshot is there is enough registers available in each file for about 75% of the entries.

some instructions don't consume registers such as branch, zeroing idioms. you can consume from both PRF independently, meaning the vectorized code mixed with at
least some GP code is unlikely to hit PRF limit before it hits ROB limit.

effect of hitting PRF limit is the same as ROB limit.

to remedy, use vectorized instructions, use zeroing idioms such as xor eax,eax, mov eax, 0

#### Branch in Flight
Intel: maximum 48 branches in flight
in flight refers to branches that have not yet retired, usually because some older operation have not yet completed.

same effect as ROB. branches here refer to both conditional and unconditional jumps.

the remedy is fewer branches. try to move unnecessary checks out of hot path, combine several checks into one. organize multi-predicate conditions and short sircit
the evaluation after the first check so subsequent checks don't appear in the dynamic stream. consider replacing N 2-way conditional jumps with one indirect jump
with N^2 targets as this counts as only one instead of N against the branch limit. consider conditional moves or other branch-free techniques.

ensure branches can retire as soon as possible.

#### Calls in Flight
Intel: 14 - 15
only 14-15 calls can be in flight at once, this only applies to `call` instructions.

to rememdy, reduce the number of function calls. inline functions, or partial inlining where fast path is inlined and slow path is not. in extreme case, replace
call + ret pairs with unconditional jumps, saving the return address in a register, plus indirect branch to return to the saved address. e.g.

```
callee:
  jmp [r15]

; caller
  movabs r15, next
  jmp callee
next:
```

this may be achievable using gcc label as value functionality:

```
void* ptr;
ptr = &&foo; // get the address of a label defiend in the current function, the value is constant and can be used whenver a constant of the type is valid.
...
goto *ptr; //need to be able to jump to one
```

## Multi-Thread Optimizations

### Memory Performance Tools

#### Reference
[LWN](https://lwn.net/Articles/255364/)
