### Constraint Satisfaction Problem
use a set of variables, each with values that will satisfy all constraints on the variable. A CSP is consist of 3 components: 
* X is a set of variables
* D is a set of domains for each variable
* C is a set of constraints that specify allowable combinations of values

solutions exist for CSP with finite discrete variables and **linear constraints**. It can be shown no algorithm exist for solving general **non-linear constraints**.

continuous domain CSP are linear programming problems, or quadratic programming problems.

it is possible to transform any n-ary constraint into binary constraints.

every finite domain can be reduced to a set of binary constraints by introducing enough auxiliary variables. Another way to convert an n-ary CSP to binary problem is using **dual graph transformation**: create a new graph which there be one variable for each constraint in the original graph, and one binary constraint for each pair of constraint in the original graph that share a variable.

real world CSP include **preference constraint** indicating which solution is preferred. Preference constraint can often encode as cost on individual variable assignment. CSP with preference can be solved with optimization search method, either path-based or local. Such problem is **constraint optimization problem**, or **COP**. Linear programming problem do this optimization.

There are 2 general method of solving CSP:
1. constraint propagation
2. backtracking search
3. local search

#### Constraint Propagation
Solve CSP problem by doing inference on constraints, use constraints to reduce the number of legal variables for a variable, which in turn reduce the legal value of another varible etc. Constraint propagation may be interwined with search, or may be done as a preprocessing step and does not require search at all.

Key idea is **local consistency**. The process enforce consistency in the partial problem in each step, producing a complete legal assignment.

unary constraint can be solved using **node consistency**, which eliminate values for a single node that doesn't satisfy the the unary constraint on the variable.

variable Xi is **arc-consistent** with respect to variable Xj if for every value of Xi in domain Di there is some value Dj that satisfies the binary constraint on the arc (Xi, Xj)

most popular arc consistency algorithm is **AC-3**. AC-3 maintains a queue of arcs to consider, pops out an arc with (Xi, Xj) and make it consistent. If there is no reduction, we're done, otherwise we reduce and put the arc into the back of the queue. If a domain for some variable Xk is reduced to empty set, then there is no solution. Algorithm is done when no reduction can be done on any variables using any arcs.

AC-3 Algorithm:
```
input: CSP R(X, D, C)

for every pair {xi, xj} that participate in constraint R in R
  queue <- queue union {(xi, xj), (xj, xi)}
endfor
while queue != {}
  select and delete (xi, xj) from queue
  Revise(xi, xj)
  if Revise(xi, xj) causes changes in Di then
    queu <- queue union {(xk, xi) for i != k}
  endif
endwhile
```

Arc Consistency cannot solve all CSP problems. It is possible where constraints implied among more than 2 variables could cause no solution and cannot be detected using arc consistency. **Path Consistency** tightens the binary constraint by using implicit constraint that are inferred by looking at tripples of variables. A two variable set `{Xi, Xj}` is path consistent with respect to a third variable `Xk` if for every assignment to Xk that satisfies `{Xi, Xk}` and `{Xk, Xj}`. The **PC-2** algorithm achieves path consistency.

Stronger consistency can be defined as **K-Consistency**. A CSP is K-consistency if for any set `k - 1` variables and for any consistent assignment to those variables, a consistent value can always be assigned to any kth variable.

A CSP is **strongly k-consistent** if it is k-consistent and (k - 1) consistent, (k - 2) consistent all the way down to 1-consistent. Given a CSP problem, we can first make it 1-consistent, then use the solution to solve for 2-consistency, and so on until k-consistency. we are guaranteed to find a solution in O(n * d) time. Any algorithm for establishing n-consistency must take exponential time in the worst case. n-consistency also requires space that is exponential in n.

in practice, we often only run 2-consistency and 3-consistency algorithm.

PC-2 algirithm

```
input CSP R(X, D, C)

queue <- (i,j,k) for 1 <= i < j <= n, 1 <= k <= n, k != i and k != j
while queue is not empty
  select and delete 3-tuple (i, j, k) from queue
  Revise-3(i, j, k)
  if Rij changed then
    queue <- queue union (l, i, j) and (l, j, i) for 1 <= l <= n, l != i and l != j
  endif
endwhile
```

complexity of PC-2 is O(n^3 * k^5)

##### Global Constraint
Global constraint can be handled using special algorithm; global constraint often raises the minimum number of possible values in the domain of a variable.

##### Bound Propagation
when CSP problem gets large, instead of representing each variable with a domain set of values, we can represent it as a upper and lower bound of possible values within the domain.

#### Backtracking Search

backtracking search algorithm: depth first search that chooses value for one variable at a time and backtrack when no legal value left to assign.

```
def BackTracking(A, U):
  if A is complete then
    return A
  endif
  remove a variable X from U
  for all values of X do
    if X = x is consistent with A according to constraints then
      add X = x to A
      result <- BackTacking(A, U)
      if result != failures then
        return result
      endif
      remove X = x from A
  endfor
  return failure
```

##### Minimum Remaining Value Heuristic
choose the next variable that has the smallest domain or remaining value

##### Degree Heuristic
choose the next variable with the most number of constraints on other unassigned variables.

##### Least Constraining Value Heuristic
when choosing a value from a variable's domain, choose the value with the least amount of constraint

The idea of choosing the most constraining variable first but choose the least constraining value first is because most constraining variable tend to prune the tree, after that we only need to choose one solution and it is more likely a solution hides behind the least constraining value than the most constraining value

##### Forward Checking
interleave backtracking search with inference using forward checking, where whenever a variable is assigned a value, the forward checking process establish arc consistency for it. For each un-assigned variable V that is connected to X by a constraint, delete from V's domain any value that is inconsistent.

There is no point in doing forward checking if we have already done arc consistency before doing backtracking search.

##### Maining Arc Consistency
After assigning a variable Xi a value during MRV, run AC-3 by pushing all constraints having constraint (Xi, Xj) for some Xj into the queue. MAC will detect problem with assignment sooner and backtrack compared to Forward Checking.

##### Intelligent Backtracking
backtrack to a variable that might fix the problem, a variable that is responsible for making one of the possible values of current assigning variable impossible. To do this, keep track of a set of assignments that are in conflict with some value for the assigning variable. This is the **conflict set**, then **backjump** to the most recent assignment that is in conflict set. The conflict set can be created during the time when choosing a legal value to assign to the assigning variable. If no legal value is found for the assigning variable, algorithm should return the conflict set along with failure. Forward checking can supply the conflict set with no extra work.

Simple backjumping is redundant with forward checking or MAC. 

##### Conflict-Directed Backjumping
let Xi be the current variable and let Conf(Xi) be its conflict set, if every possible value for Xi fails, backjump to the most recent variable Xj in Conf(Xi) and set Conf(Xj) be Conf(Xi) union Conf(Xj) 

##### Constraint Learning
find minimum set of variables from conflict set that causes the problem. This set of variables with the corresponding variables are labeled **no-good**, then the no-good are recorded as new set of constraint or the CSP or by keeping a cache of no-goods.

constraint learning can be used by either forward checking or backjumping.

### Local Search
use complete state information, assign a value to all variables, search change value for one variable at a time. local search to eliminate violated constraints.

##### Min-Conflict Heuristic
select a value that has minimum number of conflict with other variable

run time of min-conflict is independent of the problem size. Some problem can be easy for local search because solutions are densely distributed throughout state space. 

##### Constraint Weighting
concentrate search on important constraints. Each constraint is given a numeric weight, initially all at 1, at each step of the search, algorithm chooses variable/value pair to change that will result in lowest total weight of all violated constraints. Weights are then adjusted by incrementing the weight of each constraint that is violated by the current assignment.

constraint weighting can also handle online setting of scheduling where schedule changes based on condition.

### Divide and Conqure
If we can safely break a CSP problem into a subproblems if there is no constraint between the 2 groups, we reduce the complexity of the CSP and solve the problem faster.

### Direct Arc Consistency (DAC)
A constraint graph is a tree when any 2 variables are connected only by one path. Any tree structured CSP can be solved in the time linear to the number of variables. 

To solve a tree structured CSP, pick any variable to be the root of the tree, choose an ordering of variable such that each variable appears after its parent in the tree. Such ordering is a **topological sort**. Graph can reach arc-consistency in O(n * d), there is no need to backtrack because of the topology. 

Methods of converting graphs into a DAG:
* assigning value to some variables so that remaining un-assigned variables form a tree

General algorithm:
1. choose subset S from CSP variables such that remaining set becomes a tree after removal of S. S is now the **cutset**
2. for each assignment to the variables in S that satisfies all constraints on S, remove from the domain of the remaining variables that are inconsistent with the assignment of S; if the remaining CSP has a solution, return it together with assignment of S 

Finding the cutset is NP-Hard, but several efficient heuristic is known. Overal algorithmic approach is called **cutset conditionoing**.

* deconstruct the original CSP into multiple trees as subproblems, each is solved independently, resulting solution is combined

Tree decomposition algorithm:
1. every variable in the original problem appear in at least one of the sub-tree
2. if 2 variables are connected by a constraint in the original problem, they must appear together in at least one of the sub-tree
3. if a variable appear in 2 sub-tree, it must appear in every sub-tree along the path connecting those sub-tree

solve each sub-tree independently, if one sub-tree has no solution, then there is no solution to the original problem.

to construct the global solution, treat each sub-tree as a mega-variable whose domain is the set of all solutions for the subproblem. 

solving a CSP broken into multiple sub-trees can be done in polynomial time, however, method of breaking the CSP into multiple sub-tree is NP-hard, but good heuristics exist.

### Value Symmetry
reduce down solution space by removing value symmetry, where possible solutions differ only in the order of value they are assigned. e.g. using alphabetical order on the value assignment. It is also NP-hard to eliminate all symmetry among intermediate set of values during search.
