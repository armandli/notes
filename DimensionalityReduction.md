### Nonlinear Dimensionality Reduction
algorithm: *t-distributed stochastic neighbour embedding (tSNE)*

the t-SNE algorithm models the probability distribution of neighbours around each point. The term neighbours refers to set of points closest to each point. In the original high dimensional space, this is modeled as a Gaussian distribution. In the reduced 2 dimensional output space, it is modeled as t-distribution. goal is to find a mapping onto 2-dimensional space that minimizes the difference between these two distribution's overall points.

the main parameter controlling the fitting is called *perplexity*, which is roughly equivalent to the number of nearest neighbours considered when matching the original and fitted distribution for each point. low perplexity focus on local scale, while high perplexity takes more of a big picture approach.

we want to map **N** *data points* in R^D into *map point* in R^2, expecting that if 2 data points are close together, they are also close together in map point R^2.

let |xi - xj| be the Euclidean distance between two data points, and |yi - yj| be the distance between two map points. we define a conditional similarity between the two data points as:

\begin{equation}
p_{j|i} = \frac{\exp(-||x_i - x_j||^2 / 2\sigma_i^2)}{\sum_{k!=i}\exp(-||x_i - x_k||^2 / 2\sigma_i^2)}
\end{equation}

which measures how close is xj from xi, considering a *Gaussian Distribution* around xi with variance `sigma_i^2`; the variance is different for every point, it is chosen such that points in dense areas are given a smaller variance than sparse areas.

we then define similarity as a symmetrized version of the conditional similarity:

```
pij = (p_j|i + p_i|j) / (2 * N)
```

which creates a *similarity matrix* of the original dataset. then we define similarity matrix for map points.

```
qij = f(|xi - xj|) / sum_k!=i(f(|xi - xk|)) with f(z) = 1 / (1 + z^2)
```

which has the same idea but a different distribution, a *t-student with one degree distribution*, or a *cauchy distribution*

#### Physical Analogy
imagine map points are all connected with springs, the stiffness of the spring connecting points i and j depends on the mismatch between similarity of the two data points and the similarity of the two map points, that is `pij - qij`. now let the system evolve according to law of physics, if two map points are far apart while data points are close, they are attracted together, if they are nearby but data points are dissimilar, they are repelled. final mapping is where equilibrium is reached.

the physical analogy corresponds to minimizing **Kullback-Leiber divergence** between two distributions, pij and qij.

```
KL(P || Q) = sum_ij(pij * log(pij / qij))
```

to minimize the score, we performm gradient descent, which can be computed analytically:

```
dKL(P || Q) / dyi = 4 * sum_j(pij - qij) * g(|xi - xj|) * uij where g(z) = z / 1 + z^2
```

where uij is a unit vector going from yj to yi. this gradient expresses the sum of all spring forces applied to map point i.

#### Why t-student distribution ?
it is well known that the volume of N-dimensional ball of radius r scales are r^N, where N is large, if we pick random points uniformally in the ball, most points will be close to the surface, and very few will be close to the center.

if we use Gaussian distribution for the map points as well, we would get an imbalance in the distribution of the distance of the point's neighbours. this is because distribution is of the distance is so different between a high-dimensional space and low-dimensional space. yet the algorithm tries to reproduce the same distance in two spaces. the imbalance would lead to an excess of attraction forces and unappealing map.

the t-distribution has much heavier tail than Gaussian distribution, which compensates the original imbalance. for given similiarity between two data points, the corresponding map points will be much further apart in order for their similiarity to match the data similarity.
