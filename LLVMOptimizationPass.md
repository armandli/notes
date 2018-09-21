### LLVM Optimization Procedures

#### Compiler options
`--print-before-all --print-after-all` prints the before and after transformations of each optimization step on the SSA

### Ordered Optimization Steps
1. SimplifyCFG 
    -remove basic block with no predecessor (dead block)
    -merge basic block with predecessor if there is only one and predecessor only has one succeesor
    -eliminate phi() nodes basic block with a single predecessor
    -eliminate basic block with only a single unconditional branch
    -change invoke instructions to unwind functions to be called
    -change nested blocks such as `if (x) if (y)` into `if (x && y)`
2. Scalar Replacement of Aggregates (SROA)
    -examine alloca calls (function scoped memory allocations), and attempt to promote it into SSA register. a alloca can be turned into register if it is being statically assigned multiple times and the alloca can be broken down into its fundamental components (scalar replacement)
    -cannot transform alloca to register if memory address is taken, but with alias analysis sometimes we still can promote (and clang does this)
3. early common subexpression elimination
    -eliminate redundant subcomputations that are trivial
    -fast pass
4. global variable optimizer
    -transform simple global variables that never has its address taken into constants
    -eliminate global variables where it is only written to but not read from
    -interprocedural optimization (not fixed to one function)
5. instruction combiner
    -transform instructions based on list of peephole optimizations
6. SimplifyCFG (again)
7. deduce function attributes
    -deduce if function is recursive (norecurse label)
    -deduce if function mutates global state (readonly label)
    -deduce if function returns a readonly parameter refers to storage location not modified by function (nocapture label)
8. rotate loops
modifies loop from the order:
```
initializer
goto COND
COND:
  if (condition)
    goto BODY
  else
    goto EXIT
BODY:
  body
  modifier
  goto COND
EXIT:
```
into
```
initializer
if (condition)
  goto BODY
else
  goto EXIT
BODY:
  body
  modifier
  if (condition)
    goto BODY
  else
    goto EXIT
EXIT:
```

this help enable subsequent optimizations

9. SimplifyCFG
10. Canonicalize Natural Loops
    -transform natural loop into simpler form
    -loop pre-header insertion guarantee there is a single non-critical entry edge from outside of loop into the loop header
    -loop exit-block insertion guarantee all exit block from the loop only have predecessor from inside the loop and thus dominated by loop header
    -guarantee loop only have one backedge
    -modifies CFG, updates loop information and dominator information
    -enabler for LICM optimization
11. Induction Variable Simplification
    -analyze and transform the induction variable in loop for downstream optimizations
    -exit condition is canonicalized to compare induction variable to exit value. e.g. `for (i = 7; i * i < 1000; ++i)` is turned into `for (i = 7; i < 25; ++i)`
    -use outside of the loop on expression from induction variable is computed outside of the loop, eliminating dependency on the loop exit value. if loop is only used for computing such expression, loop is eliminated
12. Global Value Numbering
    -figuring out if a value computed from previous iteration in a loop can be reused in the next iteration, if so eliminate the load on the next iteration for the variable. works well for a[i+1] elimination, clang does not work on a[i+2] but gcc does
13. bit-tracking dead code elimiation
    -some instructions (shift, and, or etc) kill some of the input bits. track the dead bit and eliminate further computation using the dead bits

#### global optimization steps
1. function inlining
2. langauge specific optimizations
3. autovectorization
