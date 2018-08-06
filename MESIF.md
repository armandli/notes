### MESIF Protocol
a cache coherency and memory coherence protocol developed by Intel for cache coherent non-uniform memory architecture. protocol consist of 5 states: modified (M), exclusive (E), shared (S), invalid (I), and forward (F)

The MESI states are the same as MESI protocol. the F state is a specialized version of S state. it means cache should act as a designated responder for any requests for a given line. protocol ensures that if any cache holds a line in S state, at most one other cache holds it in F state.

* modern CPU employ a set of caches to improve memory access latency
* in order to produce a consistent view of the world when concurrent access to memory happens, the caches must communicate to give us a coherent state
* the guarantee for MESI are
    * one change at a time, a line is never in M state in more than one place
    * let me know if I need a new copy, if I have a copy and someone else changed it, I need to get me a new copy. My copy will move from Shared to Invalid

MESI state diagram:

INVALID -> INVALID (BR + BW)
INVALID -> SHARED (PR/S)
INVALID -> EXCLUSIVE (PW, PR/~S) *PW is not trivial
SHARED -> SHARED (PR + BR)
SHARED -> INVALID (BW)
SHARED -> EXCLUSIVE (PW)
EXCLUSIVE -> EXCLUSIVE (PR)
EXCLUSIVE -> INVALID (BW)
EXCLUSIVE -> MODIFIED (PW)
EXCLUSIVE -> SHARED (BR)
MODIFIED -> MODIFIED (PR + PW)
MODIFIED -> SHARED (BR)
MODIFIED -> INVALID (BW)

where

PR = processor read
PW = processor write
S/~S = shared/not shared
BR = observed bus read
BW = observed bus write

#### False Sharing
false sharing happens when 2 threads compete to write to the same cache line, which is constantly Invalid in their own state

here is how it happens
1. thread 1 and thread 2 both have the false shared cahce line, which contains a H and a T
2. thread 1 modifies H, thread 2's copy is now Invalid
3. thread 3 wants to modify T but cache is in I state, so experiences a write miss
   1. issue a read with intent to modify (RWITM)
   2. RWITM intercepted by thread 1
   3. thread 1 invalidates its own copy
   4. thread 1 writes line to meory
   5. thread 2 re-issue RWITM which result in read from memory
4. thread 1 wants to write to H again

#### Read miss
example
1. thread 1 and thread 2 both have H and T cache lines in shared state
2. thread 1 modifies H, thread 2 copy is not Invalid
3. thread 2 read H
   1. thread 2 issue request for H cache line
   2. read is picked up by thread 2
   3. thread 1 delivers the H cache line to thread 2, request to memory is dropped
   4. cache line is written out to main memory and thread 1 and 2 have the line in Shared state

we can avoid this situation by having a copy of H in thread 2 in a different cache line that thread 1 does not modify, this way thread 2 will not be blocked trying to read H if thread 2 does not need to know the most up to date value for H

note false sharing does not need the thread suffering from cache miss to have any cache write activity. The write could come from another thread sharing the same cache.

false sharing can also be introduced by **card marking**.

to detect false sharing, use `perf` on linux, `PCM` on windows/mac/linux or `Overseer` in java

solutions for false sharing:
* add padding by means of class inheritance
* hoisting hot item into a separate cache
