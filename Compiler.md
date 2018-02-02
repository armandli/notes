### Static Single Assignment Form (SSA)
dataflow analysis tend to need to keep track of **use-sites** of defined variable and **definition-site** of the variable. the **def-use chain** a data structure that makes this efficient: for each statement in the flow graph, compiler can keep pointers to its use sites of variables defined here, and a list of definition sites the variables used here, so compiler can hop quickly between instructions. The **single static assignment form (SSA)** improves this. It is an intermediate representation which each variable has only one definition. The one static definition-site may be in a loop executed many (dynamic) times, thus the name **static** single assignment form instead of single assignment form (in which variables are never redefined at all).

SSA is useful for:
1. dataflow analysis and optimization algorithm can be made simpler when each variable has only one definition
2. if a variable has N uses and M definitions (which could occupy N + N instructions), it takes space and time porprotional to N * M to represent def-use chain - a quadratic blowup. for realistic programs, size of SSA form is linear in size of original program.
3. use and def of variables in SSA form relate in a useful way to the **dominator structure** of control-flow graph, which simplifies algorithms such as **interference-graph construction**
4. unrelated use of the same variable in the source program become different variables in SSA form, eliminating needless relationships.

we can easily turn a normal program into SSA by **value numbering** where each redefinition of the same variable are given a number to distinquish between the definitions.

#### solving the most recent use problem in multi-predecessor control flow
when multiple control-flow path merge together, it is not obvious how to have only one assignment for each variable. e.g. an if-else condition where a is defined differently, which version after the else block is for a ? when a statement has more than one predecesor, there is no notion of "most recent". to solve the problem we need to introduce a fictional function called **theta function**, where we can combine a1 and a2 into a3 = theta(a1, a2). but unlike ordinary functions, theta(a1, a2) yields a1 if control flow from if condition, and a2 if control flow from else condition.

example 1: if condition

original control flow
```
b <- M[x]
a <- 0
if b < 4:
  a <- b
c <- a + b
```
SSA transformation:
```
b1 <- M[x0]
a1 <- 0
if b1 < 4:
  a2 <- b1
else:
  nop
a3 <- theta(a2, a1)
c1 <- a3 + b1
```

example 2: loop
original control flow:
```
a <- 0
loop:
  b <- a + 1
  c <- c + b
  a <- b * 2
  if a < N:
    break
return c
```

SSA transformation:
```
a1 <- 0
loop:
  a3 <- theta(a1, a2)
  b1 <- theta(b0, b2) # result not used
  c2 <- theta(c0, c1) # will be assigned multiple times, because this is not static, only static site can have single assignment, thus SSA
  b2 <- a3 + 1
  c1 <- c2 + b2
  a2 <- b2 * 2
  if a2 < N:
    return c1
  else:
    continue
```
where b0, c0 are initial values of the variable before the block

* we can implement the theta function using a move instruction on each incoming edge
* in many cases we simply need the connection of uses and definitions, and don't need to execute the theta function during optimization, and can ignore the question of which value to produce

criteria for inserting theta function:
we could add theta-function for every variable at each join point, but this is not necessary. the following criterion characterizes the node where a variable's data flow path merge (**path convergence criterion**):
1. there is a block x containing a definition of a
2. there is a block y containing a definition of a where y != x
3. there is a non-empty path Pxz of edge from x to z
4. there is a non-empty path Pyz of edge from y to z
5. path Pxz and Pyz do not have any node in common other than z
6. node z does not appear within both Pxz and Pyz prior to the end, though it may appear in one or the other

note theta-function itself counts as a definition of a

iterated path-convergence criterion:
```
while there are nodes x, y, z satisfying conditions 1-5 and z does not contain theta-function for a; do
  insert a <- theta(a, a, ...) at node z
  where the theta funtion has many a arguments as there are predecessors of node z
```

**dominance** property of SSA form:
1. if x is the ith argument of a theta-function in block n, then the definition of x dominates the ith predecessor of n
2. if x is used in a non-theta statement in block n, then definition of x dominates n

note the dominance relation is **d dominates n if every path from the start node to n goes through d**

#### dominance frontier
the iterated path convergence algorithm for placing theta-function is not practical since it's costly to examine every triplet x, y, z and every path leading from x and y. more efficient algorithm uses **dominator tree** of flow graph

definition: **x strictly dominates w if x dominates w and x != w. node x is an ancestor of y if there is a path x -> y of tree edges and is a proper ancesor if that path is non-empty**

the **dominance frontier** of a node x is set of all nodes w such that x dominates a predecessor of w, but does not strictly dominate w

in essence dominance frontier is the "border" between dominated and undominated nodes

**dominance frontier criterion** whenever node x contains a definition of some variable a, then any node z in the dominance frontier of x needs a theta-function for a

**iterated dominance frontier** since a theta-function itself is a kind of definition, we must iterate the dominance frontier criterion until there are no nodes that need theta functions

theorem: the iterated dominance frontier criterion and the iterated path-convergence criterion specify exactly the same set of nodes at which to put theta-functions

computing the dominance frontier: define 2 auxiliary sets:
1. DFlocal[n]: the successors of n that are not strictly dominated by n
2. DFup[n]: nodes in the dominance frontier of n that are not dominated by n's immediate dominator

dominance frontier of n can be computed from DFlocal and DFup:
DF[n] = DFlocal[n] union DFup[c] where c are all nodes in children[n], which are nodes whose immediate dominator is n

to compute DFlocal[n] more easily using immediate dominators instead of dominators, we use the following theorem: DFlocal[n] = the set of those successors of n whose immediate dominator is not n

following algorithm should start from the root of the dominator tree, and walk the tree computing DF[n] for every node n: it computes DFlocal[n] by examining the successors of n, the combines DFlocal[n] and DFup[c]

```
compute DF[n] = 
  S <- {}
  for each node y in succ[n]:
    if idom(y) != n:
      S <- S union {y}
  for each child c of n in the dominator tree:
    compute DF[c]
    for each element w of DF[c]:
      if n does not dominate w:
        S <- S union {w} 
  DF[n] <- S>
```

inserting theta-functions algorithm: starting with a program not in SSA form, insert just enough theta-functions to satisfy the iterated dominance frontier critierion, to avoid re-examining

```
place-theta-function = 
  for each node n:
    for each variable a in Aorig[n]:
      defsites[a] <- defsites[a] union {n}
  for each variable a:
    W <- defsites[a]
    while W not empty:
      remove some node n from W
      for each Y in DF[n]:
        if Y not in Atheta[n]:
          insert the statement a <- theta(a, a, ..) at the top of the block Y, where the theta function has many arguments as Y has predecessors
          Atheta[n] <- Atheta[n] union {Y}
          if Y not in Aorig[n]:
            W <- W union {Y}
```

#### Renaming Variables
after theta functions are placed, we can walk the dominator tree, renaming different definitions of variables a to a1, a2, ... etc

in straight line we would rename all definitions of a, then each use of a is using the most recent definition of a. for a program with control flow branches, and joins whos graph satisfies the dominance-frontier criterion, we rename each use of a to use the closest definition d of a that is above a in the dominator tree.

#### Edge Spliting
some analysis and transformations are simpler if there is never a control flow edge that leads from a node with multiple successors to a node with multiple predecessors. to give the graph a unique successor or predecessor property, we perform the edge spliting transoformation: for each control flow edge a -> b such that a has more than one successor and b has more than one predecessor, we create a new empty control flow node z and replace a -> b edge with a -> z, and z -> b edge

#### Efficient Computation of The Dominator Tree
in order to do optimization fast, doing SSA transformation also needs to be fast. The near-linear time algorithm **Lengauer and Tarjan** relies on properties of depth-first-spanning-tree of control-flow graph. This is just the recursion tree implicitly traversed by depth-first-search (DFS) algorithm, which numberes each node of the graph with **depth first number (dfnum)** as it is first encountered.

#### SSA and Liveness Analysis
we can construct the interference graph of an SSA program just prior to converting the theta-function to move instructions.

## Liveness Analysis
liveness anaylsis is done using flow analysis

flow analysis definitions:
**succ[n]**: successor node of n are nodes where there is an out-edge between n and the node
**pred[n]**: predecessor node of n are nodes where there is an in-edge from the node to n
**use[n]**: set of graph nodes that uses variable n
**def[n]**: set of graph nodes that defines a variable n
**liveness**: a variable is live on an edge if there is a directed path from that edge to a use of the variable that does not go through any def. a variable is live at a node if it is live on any of the in-edges of that node, it is live-out at the node if it is live on any of the out-edge of the node

liveness information (live-in live-out) can be calculated from use and def as follows:
**in[n]** = use[n] U (out[n] - def[n]) where U is union
**out[n]** = U[s in succ[n]] (in[s]) where U is union

note out[n] should be computed before in[n]

algorithm:
for each n:
  in[n] <- {}; out[n] <- {}
repeat:
  for each n:
    in'[n] <- in[n]; out'[n] <- out[n]
    in[n] <- use[n] U (out[n] - def[n])
    out[n] <- U[s in succ[n]] (in[s])
until in'[n] = in[n] and out'[n] for all n

1. if variable is in use[n], then it is live-in at node n. thus if the statement uses a variable, the variable is live on entry to that statement
2. if variable is live-in at a node n, then it is live-out at all nodes m in pred[n]
3. if variable is live-out at node n, and not in def[n], then variable is also live-in at n.

algorithm complexity: a program of size N at most N nodes in the flow graph, and at most N variables. worst run time is O(N^4). ordering the nodes using depth-first search bring the number of repeat-lopp iterations to 2 or 3, and live-sets are often sparse, so algorithm runs between O(N) and O(N^2)

solutions to dataflow equations are **conservative approximations**. consequence is we may believe a variable is live but will never erroneously believe it is dead. this way to prevent incorrect program, although suboptimal.

**dynamic liveness**: variable a is dynamically live at node n if some execution of the program goes from n to a use of a without going through any definition of a
**static liveness**: variable a is statically live at node n if there is some path of control flow edge from n to some use of a that does not go through a definition of a

dynamic liveness cannot be computed because of halting problem. we can come up with situational improvements, but static liveness is the one we do.

#### Interference Graph
liveness analysis is used for register allocation. interference information can be expressed as a matrix, or as an undirected graph, with nodes as variable and edges connecting variable if they interfere.

interference grpah algorithm:
1. at any non-move instruction that defines a variable a, where the live-out variables b1..bj, add interference edge (a, b1), .. (a, bj)
2. at a move instruction a <- c, where variables b1..bj are live-out, add interference edge (a, b1)..(a, bj) for any b that is not the same as c

## Register Allocation
once we have interference graph on variables, we color the interference graph with as few colors as possible. nodes where there is no connected edge can be assigned the same color. If the machine has K registers, we can K-color the graph. If there is no K-coloring, we need to save some variables to memory, this is called spilling.

register allocation is a NP-complete problem (unless for expression trees). There is a good linear time approximation algorithm, its principle phase are **build, simplify, spill, select**

* build: construct the interference graph
* simplify: color the graph using a simple heuristic. suppose graph G contains a node m with fewer than K neighbours, where K is the number of registers on the machine. let G' be the graph G - {m}. If G' can be colored, then so can G, for when m is added to the colored graph G', the neighbors of m have at most K - 1 colors. This is a stack based algorithm for coloring: repeated push nodes of degree less than K, each simplification will decrease the degrees of other nodes, leading to more simplification.
* spill: at some point during simplification the graph G has a node >= K degree, then simplify heuristic fails, and we mark some node for spilling. We choose some node in the graph and decide to represent it in memory, not registers during program execution. then continue simplification.
* select: assign colors to the nodes in graph, starting with empty graph, we rebuild the original graph by repeatedly adding a node from top of the stack. when a spill node is popped, there is no guarantee it will be colorable. we check if it is, if not we have actual spill. this technique is known as **optimistic coloring**
* start over: if select phase is unable to find a color for some node, then program must be rewritten to fetch them from memory just before each use, and then store them back after each definition. thus a spilled temporary will turn into several new tempoaries with tiny live ranges, which will interfere with other temporaries in the graph. so the algorithm is repeated on the rewritten program. it iterates until simplify succeeds with no spill.


