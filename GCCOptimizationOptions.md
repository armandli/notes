### Optimization Options
`-O0` optimize = 0, disable optimization, code is direct translation of language
`-Og` optimize = 1 + debug, very fast optimization with low impact on debug, allows dead code elimiation, instruction reordering, replacement of common sub-expression, openMP simplification, 
`--auto-inc-dec` generate auto increment decrement instruction on address when architecture supports them
`--ira-share-save-slots` share slots for saving different hard registers, allow using register to pass function parameter
`--dce` dead code elimination pass
`--dse` remove dead store instruction
`--defer-pop` deffer popping function args from stack until later
`--delay-branch` attempt to fill delay slots of branch instructions
`--guess-branch-probability` guessing branch probability
`--cprop-registers` perform register copy-propagation optimization pass
`--forward-propagate` perform forward propagation pass on RTL
`--ipa-pure-const` discover pure and const functions
`--ipa-reference` discover read only and non-addressable static variables
`--ipa-profile` perform interprocedural profile propagation
`--merge-constants` merge identical constants across compilation units
`--reorder-blocks` reorder basic block to improve code placement
`--shrink-wrap` emit function prologues only before parts of function that needs it rather than at top of function
`--split-wide-types` split wide types into independent registers
`--tree-ccp` enable SSA-CCP optimization trees, may simplify fold, call and assign into constant assignments, drop unreachable block
`--tree-coalesce-vars` enable SSA coalescing of user variables
`--tree-dce` enable SSA dead code elimination on trees
`--tree-dse` enable dead store elimination
`--tree-ter` replace temporary expression in SSA->normal pass
`--tree-fre` enable full redundancy elimination on trees
`--tree-copy-prop` enable copy propagation on trees
`--tree-sink` enable SSA code sinking on trees
`--tree-ch` enable loop header copy on trees
`--combine-stack-adjustments` look for opportunities to reduce stack adjustment and stack references
`--compare-elim` compare elimination after register allocation has finished
`--tree-slsr` perform straight line strength reduction
`--omit-frame-pointer` when possible, do not generate stack frame

-O0 -Og limit maximum vectorization factor for openMP to be 1

`-O1` optimize = 1, perform only fast optimization, target specific vectorization size can be used, atomic operation into atomic bit test and set, complement or reset.

`--tree-cselim` transform condition store into unconditional one
`--ssa-backprop` enable backward propagation, detect numeric variable where sign does not matter
`--tree-phiprop` enable hoisting loads from conditional pointers
`--tree-forwprop` enable forward propagation in trees
`--stdarg-opt` optimize amount of stdarg registers saved to stack at start of function
`--tree-builtin-call-dce` enable conditional dead code elimination on builtin calls
`--tree-dominator-opts` enable dominator optimization
`--tree-reassoc` enable reassociation on tree level
`--tree-loop-im` enable loop invariant motion on trees
`--tree-loop-optimize` enable loop optimization on tree level
`--tree-scev-cprop` copy propagation of scalar-evolution information
`--tree-loop-ivcanon` create canonical induction variable in loops
`--ivopts` optimize induction variable on trees
`--if-conversion` perform conversion on conditional jumps to branchless equivalents
`--if-conversion2` perform conversion of conditional jumps to conditional execution
`--tree-bit-ccp` enable SSA-BIT-CCP optimiztion trees
`--tree-sra` perform scalar replacement of aggregates
`--branch-count-reg` replace add, compare, branch with branch on count register
`--move-loop-invariants` move loop invariant computations out of loop
`--tree-pta` perform function local points-to analysis on trees
`--ssa-phiopt` optimize conditional pattern using SSA PHI nodes
`--inline-functions-called-once` integrate functions only required by their single caller

`-Os` optimize=2 + size, perform optimization at level 2 that also reduce code size

`--dce` use the RTL dead code elimination pass
`--tree-dce` aggressive dead code elimination, take control of dependencies, enable dead branch elimination
`--inline-small-functions` integrate function into caller code if code size is known and does not grow
`--indirect-inlining` perform indirect inlining 
`--partial-inlining` perform spliting of functions so part can be inlined while other stand alone
`--thread-jumps` perform RTL jump threading optimization
`--crossjumping` identify common trailng instructions in predecessor block, or leading instruction in successor block, splitting one of the block so that other can have a jump
`--optimize-sibling-calls` optimize sibling and tail recursive calls, turn tail recursion into loops, 
`--cse-follow-jumps` when running CSE, follow jump to target
`--gcse` perform global common subexpression elimination
`--expensive-optimizations` perform a number of minor expensive optimizations
`--rerun-cse-after-loop` add a CSE pass after loop optimization
`--caller-saves` save register around function calls
`--peephole2` enable RTL peephole pass before sched2
`--schedule-insns2` reschedule instructions after register allocation
`--strict-aliasing` assume strict aliasing rules apply
`--reorder-functions` reorder function to improve code placement
`--tree-vrp` perform value-range-propagation on trees
`--code-hoisting` enable code hoisting
`--tree-pre` enable SSA-PRE optimization on trees
`--tree-switch-conversion` perform conversion of swich initialization
`--ipa-cp` perform interprocedural constant propagation
`--ipa-bit-cp` perform interprocedural bitwise constnat propagation
`--ipa-vrp` perform IPA value range propagation
`--devirtualize` convert virtual calls into direct ones
`--devirtualize-speculatively` perform speculative devirtualization
`--ipa-sra` perform interprocedural reduction of aggregates
`--align-loops` align start of loops
`--align-jumps` align labels which are only reachable by jumping
`--align-labels` align all labels
`--align-functions` align the start of functions
`--tree-tail-merge` enable tail merging on trees
`--vect-cost-model=cheap` cheap cost model of vectorization
`--hoist-adjacent-loads` hoisting adjacent loads to encourage generating conditional move instructions
`--ipa-icf` perform identical code folding for functions and read-only variables
`--isolate-erroneous-paths-deference` turn undefined behavior into traps
`--ipa-ra` use caller save register across calls if possible
`--ira-remat` do CFG-sensitive rematerialization in LRA
`--store-merging` merge adjacent stores
`--inline-functions` integrate functions not declared inline into their callers when profittable

`-O2` optimize = 2, perform optimization that make programs run faster

`--no-inline-functions` -Os and -O3 enable inline functions, -O2 does not
`--schedule-insns` reschedule instruction before register allocation
`--reorder-blocks-algorithm=stc` the STC algorithm may duplicate block and rotate loops 
`--optimize-strlen` enable string length optimization on trees

`-O3` optimize = 3, perform expensive optimizations that may make the program larger and slower

`--inline-functions` inline function
`--tree-loop-distribution` enable loop distribution on trees
`--tree-loop-distribute-patterns` enable loop distribution for patterns transformed into a library call
`--loop-interchange` enable loop interchange on trees
`--predictive-commoning` run predictive commonining optimization
`--split-paths` split paths leading to loop backedges
`--split-loops` perform loop spliting
`--unswitch-loops` perform loop unswitching
`--loop-unroll-and-jam` perform loop unroll and jam on loops
`--gcse-after-reload` perform global common subexpression elimination after register allocation has finished
`--tree-loop-vectorize` enable loop vectorization on trees
`--tree-loop-if-convert` transform multiple loop bodies into a single block
`--tree-slp-vectorize` enable basic block vectoriation on trees
`--vect-cost-model=dynamic` dynamic cost model for vectorization
`--ipa-cp-clone` perform cloning to make interprocedural constnat propagation stronger
`--tree-partial-pre` in SSA-PRE optimization on trees, enable partial-partial redundancy elimination
`--peel-loop` perform loop peeling

`-Ofast` optimize=3 + fast, perform expensive optimizations, fast math transformations that could maek standard compliant program misbehave

`--fast-math` disable various aspect of floating point strict correctness, from folding to removal of exception handling
`--reciprocal-math` enable optimization substitutes floating point division by SSA_NAME with multiplication by the reciprocal.

#### GCC option to print optimization information
`-fopt-info-all-vec` generate all optimization information and which part failed to optimize
`-fsave-optimization-record` saves optimization information in json form in GCC9

additional resources for reading optimization information:
github.com/drepper/gcc-passes is a gcc plugin that discover optimization passes used during compilation
github.com/drepper/optmark is a reader of json file generated from -fsave-optimization-record to further refine the information for human reading

#### jump threading optimization
goal is to reduce the number of jumps in code, resulting in improved performance due to eliminating conditionals, which allows other optimizations. simplification
of control flow also reduce false positive rates of option `-Wuninitialized`, which also coorelate with missed optimization opportunities.

example:
```
if (a > 5)
  goto j;
doStuff();
doStuff();
j:
  goto somewhere;
```

is optimized into:
```
if (a > 5)
  goto somewhere;
doStuff();
doStuff();
j:
  goto somewhere;
```

also deals with overlapping conditionals. example

```
void foo(int a, int b, int c){
  if (a && b)
    foo();
  if (b || c)
    bar();
}
```

optimized into

```
void foo(int a, int b, int c){
  if (a && b){
    foo();
    goto skip;
  }
  if (b || c){
skip:
    bar();
  }
}
```

jump threading can duplicate block to avoid jumping. example

```
void foo(int a, int b, int c){
  if (a && b)
    foo();
  tweak();
  if (b || c)
    bar();
}
```

transformed into

```
void foo(int a, int b, int c){
  if (a && b){
    foo();
    tweak();
    goto skip;
  }
  tweak();
  if (b || c){
skip:
    bar();
  }
}
```

jump threading eliminate conditional jumps, which may be at the expense of duplicating code.

GCC limite jump threading inling capability by option `--param max-fsm-paths-insns=500` which controls the maximum number of instructions to duplicate or `--param max-fsm-thread-length` which controls the maximum basic block length instead of instruction length.

jump threading optimization is enabled at O2, and is intertwined with other optimization such as value range propagation (VRP) so it cannot be independently turned
off. `-fno-thread-jumps` only turns off jump threading in low level RTL optimizer.

to see jump threading in action, use option `-fdump-tree-all-details -O2` and look at `*.C*{ethread, thread1, thread2, thread3 thread4}` as well as VRP dumps
`*.c*{vrp1, vrp2}` which contains lines like Threaded jump 3 --> 4 to 7

##### warning for data structure padding
use option `-Wpadded` to warn when padding is used in data structures

##### warnings for bad use of std::move
`-Wredundant-move` warn when move is redundant
`-Wpessimizing-move` warn when move is a bad idea, such as breaking NRVO
