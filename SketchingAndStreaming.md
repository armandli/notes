### Skeching And Streaming Algorithms
probabislitic algorithms

#### Morris Algorithm for probabilistic counting
problem: storing large count in small integer space
idea: if every time when increment the count, flip a count and increment if it's head, then 1 bit is saved, but with high error factor of 2. the expectation of the value X is `E[X] = N / 2`, so we can recover the original value of N by
```
2X = N
```
we can recover an approximation of the original count value N by timing the value saved in the counter X by 2

improvement: instead we flip X number of coins, and only increment the value if all flipped heads. then the formula becomes
```
2^X - 1 = N
```
because `E[X] = log(N + 1)` we're only using O(log log N) bits to save the counter, but still have high variance. this means we increment the value of X by chance of `0.5^X`, or `1 / (2^X)`

we can reduce the variance by choosing something in between `0.5^X` and `1^X` where `1^X` is deterministic and does not save any storage. For example, `0.99^X`, then `N = 100 * (1.01^X - 1)`. this changes the formula to `(1 + a)^X` where `0 <= a <= 1`. the variance will have the formula `a * N * (N - 1) / 2`, and the space complexity will be in O(log log N + log(1/a)).

using Chebyshev inequality, we can analyze that the optimal `a = 2 * epsilon^2 * eta` where epsilon is in the error rate formula `error rate = 1 + epsilon`, example error rate is 0.1

#### Count distinct elements in stream with finite maximum number of elements
this is impossible to do in sublinear memory unless the algorithm is both randomized and approximate
idea: order statistics. pick random hash function h: [n] -> [0, 1] where hash function h takes each element in n and produce hash value in reals between 0 and 1 inclusive, and store X where X is the h(i) for min i seen in stream. the `E[X] = 1 / (t + 1)` where t is the distinct element count, so t can be approximated by `(1 / X) - 1`

we can improve this further by using k smallest hash values `X1 <= X2 <= X3 <= ... Xk`, then `E[Xk] = k / (t + 1)` and t can be approximated by `1 / Xk - 1`. if k is set to be `k = 1 / epsilon^2` then t has error `(1 +/- epsilon) * t` with probability 2/3

example algorithm is the hyper log log.
basic idea:
create a binary grid of log n rows and k columns, where `k = 1 / epsilon^2`. 2 hash functions `g: [n] -> [k]` and `h: [n] -> [log n]` with `P[h(i) = j] == 1 / 2^j`, a geometric decay probability where 1/2 will be in first row, 1/4 the second row 1/8 the third etc.
given a stream of elements e, g(e) will choose a random column and h(e) will choose a random row with geometric decaying probability, bias towards lower row index, mark the cell true.
to compute the number of distinct elements, we identify the lowest row index i where < 25% of the row cells are true and cut off rows with index `< i`, for the remaining columns of the grid, let o be the number of columns where at least one cell is true. the expectation for `E[o] = k * (1 - (1 - 1 / k)^b)` where the expectation for b is `E[b] = 2 * F / 2^i` where F is the number of unique elements. we solve for b using the `E[o]` formula, knowing the estimate of `E[o]` based on the count of how many columns with true in it, and then use the formula for `E[b]` to solve for F knowing the empirical estimate of `E[b]`

hyper log log algorithm optimize out the column and just save the highest row index where it is true in that column.

#### Turnstile Model
a vector z of R^n where n is huge receives in stream update(i, delta) which cause change in `zi <- zi + delta`. we want to approximate f(z). if delta = 1, then we have the distinct count algorithm

#### Top hitter problem, find the top k most popular element in the stream
given stream of items coming from some universe U, report small list L subset of U containing all frequent items. e.g. k-frequent means appearing bigger than l > k times. there is trivial solution if we keep a count on each item in universe U, if U is small. requirement is no false negatives.

there is a even harder problem, where we want to do change detection, e.g. detecting what was popular but not anymore, or what was not popular but is now.

there are many algorithms:

* Frequent
* StickySampling
* LossyCounting
* CountSketch
* SampleAndHold
* SpaceSaving
* CountMin Sketch
* CountMind sketch with dyadic trick
* Sketch-guided sampling
* HSS (Hierarchical CountSketch)
* Multi-stage bloom filters
* incoherentSketch
* CountSketch with codes
* CountSieve
* BDW
* ExpanderSketch
* BPTree

#### CountSketch ALgorithm
given a grid of counters, with l number of rows and w number of columns. we use a pair of hash function h_j and sigma_j for each row, where `h_j: [n] -> [w]` and `sigma_j: [n] -> {-1, 1}`, using h_j to decide which column to use for each row and increment or decrement the cell using sigma_j by `zi <- zi + sigma_j(i) * delta`.

each counter that i hashes to stores a sigma_j(i) * zi plus noise (from other items colliding). if `w >> k` for k-post popular element, then no other top k items would collide with i under the markov assumption on h. the expected `noise <= 1 / sqrt(w) * || z_tail(k) ||2` where `||.||2` is the L2 norm, and z_tail(k) means take z but zero out the top k entries.

we estimate i as the median of all the counters it hashed to, the most popular i are `|zi| > 1 / sqrt(k) * || z_tail(k) ||2`

if we set w = Ck and return 2k value of the most popular items, with failure probability 1 / n^C

#### Dimensionality Reduction Problem
reducing machine learning input data dimension

for any n vectors of arbitrary dimension, can map to O(n log n) dimension while approximately preserving Euclidean geometry.

#### Random Projection Method
pick some random linear map x -> Px and apply P to input as a preprocessing step

#### Probabilistic Matrix vector multiplication
given matrix S multiplying vector x to produce y. if we create a vector y by sampling m coordinates of x, then `E[1/m||y||2^2] = ||x||2^2`. but this can have high variance, for example, x has only one zero entry.

to fix that, we use the DFT, this way the rare zero entry in x becomes smoothed out into all coordinates of x, |x_hat_i| = 1 / sqrt(n) for all i. now the formula becomes `y = SFx` where F is the DFT matrix.

but this birngs in the problem what if x is evenly smoothed out, DFT will concentrate it into a single coordinate. so in addition we will multiple another matrix D where D is a diagonal matrix with its diagonal value +/-1 with random sign, this way it allows SFDx to be well spread. the formula becomes `y = SFDx`.

we use FFT to do the DFT transform instead to make it faster.

#### Random Sparse Matrix as Feature
when feature matrix is sparse, the CountSketch algorithm is a random sparse matrix, so use CountSketch to reduce the dimensionality of the sparse feature matrix. example: bag of words encoding of short sentences as features, where words in a dictionary is huge.

#### Sketch and Solve
optimize linear algebra operations by approximating the solution probabilistically with near optimal results. example problem such as least squares, PCA, matrix-matrix multiplication.

for PCA, instead of solving P* minimizing `||(I - P)A||F^2`, solve for P* minimizing `||(I - P)AQ^T||F^2` where we use sketch matrix Q to reduce the number of columns of A

for matrix-matrix multiplication, instead of computing `C = A^T * B`, we compute `C = (QA)^T * (QB)`

we can get good approximation by choosing a good sketch matrix, example sketch matrix can be subgaussian entries, SFD, CountSketch

#### k-means with JL embedding
k-means can be optimized by using JL embedding f which preserves cost(P) for all P where P is the partition in k-means. we can reduce the dimension to `O((epsilon^-2) * log n)`

we can sketch and solve k-means.

k-means can be formalized as a constrained low-rank approximation problem. we can use sketch-and-solve to solve for lwo rank approximation. to optimize up to 1 + epsilon, it suffices for sketching matrix Q to only have O(k / epsilon^2) rows, so this is even better if k-means only have a few clusters, the approximation only needs to have size relative to the number of clusters insetad of number of points.

recent advancement shows it can be improved further to O(log(k/epsilon) / epsilon^2)
