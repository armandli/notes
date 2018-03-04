## Dimensionality Reduction

#### Dimensionality Reduction Algorithms
1. PCS (linear)
2. t-SNE (non-parametric/nonlinear)
3. Sammon mapping (nonlinear)
4. Isomap (nonlinear)
5. LLE (nonlinear)
6. CCA (nonlinear)
7. MVU (nonlinear)
8. SNE (nonlinear)
9. Laplacian Eigenmaps (nonlinear)

### Principle Component Analysis (PCA)
limits: linear algorithm. Not able to interpret complex polynomial relationship between features.

### t-SNE Nonlinear Dimensionality Reduction
**t-distributed stochastic neighbour embedding (tSNE)**

t-SNE is a non-linear dimensionality reduction algorithm used for exploring high-dimensional data. It maps multi-dimensional data to two or more dimensions suitable for human observation.

the t-SNE algorithm models the probability distribution with random walk on neighbours around each point. The term neighbours refers to set of points closest to each point. In the original high dimensional space, this is modeled as a **Gaussian distribution**. In the reduced 2 dimensional output space, it is modeled as **t-distribution**. goal is to find a mapping onto 2-dimensional space that minimizes the difference between these two distribution's overall points.

the main parameter controlling the fitting is called **perplexity**, which is roughly equivalent to the number of nearest neighbours considered when matching the original and fitted distribution for each point. low perplexity focus on local scale, while high perplexity takes more of a big picture approach.

****Algorithm****
we want to map N **data points** in $$R^D$$ into **map point** in $$R^2$$.

start by converting the high-dimensional euclidean distances between data points into conditional probabilities that represent similarities. The similarity between data point $$x_i$$ and $$x_j$$ is the conditional probability $$p_{j|i}$$. Pick $$x_j$$ as its neighbor if neighbors were picked in proportion to their probability density under Gaussian centered at $$x_i$$.

for nearby data points, $$p_{j|i}$$ is relatively high, whereas for widely separated data points, $$p_{j|i}$$ will be small. The conditional probability $$p_{j|i}$$ is given by

$$$
p_{j|i} = \frac{\exp(-||x_i - x_j||^2 / 2\sigma_i^2)}{\sum_{k!=i}\exp(-||x_i - x_k||^2 / 2\sigma_i^2)}
$$$

where $$\sigma_i$$ is the variance of the Gaussian distribution centered on data point $$x_i$$, $$||x_i - x_j||$$ is the euclidean distance between $$x_i$$ and $$x_j$$.

In other words, similarity between point is the conditional probability that $$x_i$$ would pick $$x_j$$ as its neighbor if neighbors were picked in proportion to their pobability density under Gaussian centered at $$x_i$$.

let $$||y_i - y_j||$$ be the low-dimensional counterpart distance between 2 data points of the high-dimensional data point $$x_i$$ and $$x_j$$. It is possible to compute a similar conditional probability which we denote $$q|{j|i}$$

$$$
q_{j|i} = \frac{\exp(-||y_i - y_j||^2)}{\sum_{k!=i}\exp(-||y_i - y_k||^2)}
$$$

note that $$p_{i|i}$$ and $$p_{j|j}$$ are set to 0 as we want to model pair wise similarity.

So we calculated the conditional probability of similarity between a pair of points in high dimensional space and in low dimensional space.

Logically the conditional probabilities $$p_{j|i}$$ and $$q_{j|i}$$ must be equal for a perfect representation of the similarity of the datapoints in difference dimensional space. SNE attempts to minimize this difference of conditional probability.

To measure the **minimization of sum of differences of conditional probabilities**, we use **Kullback-Leibler divergence** over all data points using a **gradient descent method**. The KL divergence are asymmetric in nature, but t-SNE uses the symmetric version of the cost function, with simple gradients. It also uses t-distribution in the low dimensional map point space to alleviate both the **crowding problem** and **optimization problem**. The t-distribution with one degree is also called *cauchy distribution*.

crowding happens when the 2-dimensional map that is available to accommodate moderately distant data points will not be nearly large enough compared with the area available to accommodate nearby data points

The remaining parameter to be selected is the $$\sigma_i$$ of the t-distribution that is centered over each data point $$x_i$$. It is not likely that there is a single value of $$\sigma_i$$ that is optimal for all data points in the data set because the density of the data is likely to vary. In dense regions, smaller value of $$\sigma_i$$ is more appropriate. t-SNE performs a binary search for the value of $$\sigma_i$$ that produces $$P_i$$ with fixed perplexity that is specified by $$\sigma_i$$. Perplexity is defined as 

$$$
Perp(P_i) = 2^{H(P_i)}
$$$

where $$H(P_i)$$ is the **Shannon entrpy** of $$P_i$$ measured in bits

$$$
H(P_i) = - \sum_jp_{j|i}\log_2p_{j|i}
$$$

The perplexity can be interpreted as a smooth measure of the effective number of neighbors. The performance of t-SNE is fairly robust to changes in the perplexity, and typical value falls between 5 and 50.

Minimization of the cost function is performed using gradient descent. Physically, the gradient may be interpreted as the resultant force created by a set of springs between the map point $$y_i$$ and $$y_j$$. All springs exert a force along the direction $$y_i - y_j$$. The spring between $$y_i$$ and $$y_j$$ repels or attract the point points depending on whether the disance between the two in the map is too small or too large to represent the similarities between the 2 high-dimensional data points. the force exerted by the spring between $$y_i$$ and $$y_j$$ is proportional to its length, stiffness, which is a mismatch $$(p_{j|i} - q_{j|i) * p_{i|j} + q_{i|j})$$

which measures how close is $$x_j$$ from $$x_i$$, considering a **Gaussian Distribution** around $$x_i$$ with variance $$\sigma_i^2$$; the variance is different for every point, it is chosen such that points in dense areas are given a smaller variance than sparse areas.

we then define similarity as a symmetrized version of the conditional similarity:

$$$
pij = \frac{(p_{j|i} + p_{i|j})}{2 * N}
$$$

which creates a *similarity matrix* of the original dataset. then we define similarity matrix for map points.

#### Time and Space Complexity
the algorithm computes pairwise conditional probabilities and tries to minimize the sum of the difference of the probabilities in higher and lower dimensions. t-SNE has a quadratic time and space complexity while applying it to data.

#### Physical Analogy
imagine map points are all connected with springs, the stiffness of the spring connecting points i and j depends on the mismatch between similarity of the two data points and the similarity of the two map points, that is $$p_{j|i} - q_{j|i}$$. now let the system evolve according to law of physics, if two map points are far apart while data points are close, they are attracted together, if they are nearby but data points are dissimilar, they are repelled. final mapping is where equilibrium is reached.

the physical analogy corresponds to minimizing **Kullback-Leiber divergence** between two distributions, $$p_{j|i}$$ and $$q_{j|i}$$.

$$$
KL(P || Q) = \sum_{ij}(p_{ij} * \log(p_{ij} / q_{ij}))
$$$

to minimize the score, we performm gradient descent, which can be computed analytically:

$$$
\frac{dKL(P || Q)}{dy_i} = 4 * \sum_j(p{j|i} - q{j|i}) * g(|x_i - x_j|) * u_{ij}
$$$
where $$g(z) = \frac{z}{1 + z^2}$$

where $$u_ij$$ is a unit vector going from $$y_j$$ to $$y_i$$. this gradient expresses the sum of all spring forces applied to map point $$i$$.

#### Why t-student distribution ?
it is well known that the volume of N-dimensional ball of radius r scales are $$r^N$$, where N is large, if we pick random points uniformally in the ball, most points will be close to the surface, and very few will be close to the center.

if we use Gaussian distribution for the map points as well, we would get an imbalance in the distribution of the distance of the point's neighbours. this is because distribution is of the distance is so different between a high-dimensional space and low-dimensional space. yet the algorithm tries to reproduce the same distance in two spaces. the imbalance would lead to an excess of attraction forces and unappealing map.

the t-distribution has much heavier tail than Gaussian distribution, which compensates the original imbalance. for given similiarity between two data points, the corresponding map points will be much further apart in order for their similiarity to match the data similarity.

#### Common Fallacies
few common fallacies to avoid when interpreting results of t-SNE

1. perplexity should be smaller than the number of points. perplexity value should be between 5 to 50
2. different runes with the same hyperparameter will produce different results
3. cluster sizes in any t-SNE plot must not be evaluated for standard deviation, dispersion or any other similar measures. this is because t-SNE expands denser clusters and contracts sparser clusters to even out cluster sizes.
4. distances between clusters may change because global geometry is closely related to optimal perplexity. In a dataset with many clusters with different number of elements, one perplexity cannot optimize distances for all clusters
5. patterns may be found in random noise as well, so multiple runes of the algorithm with different sets of hyperparameter must be checked before deciding if a pattern exist
6. different cluster shapes may be observed at different perplexity level
7. topology cannot be analyzed based on a single t-SNE plot, multiple plots must be observed before making any assessment

#### Python Example
obtain the **tnse** python package

```
## importing the required packages
from time import time
import numpy as np
import matplotlib.pyplot as plt
from matplotlib import offsetbox
from sklearn import (manifold, datasets, decomposition, ensemble,
             discriminant_analysis, random_projection)
## Loading and curating the data
digits = datasets.load_digits(n_class=10)
X = digits.data
y = digits.target
n_samples, n_features = X.shape
n_neighbors = 30
## Function to Scale and visualize the embedding vectors
def plot_embedding(X, title=None):
    x_min, x_max = np.min(X, 0), np.max(X, 0)
    X = (X - x_min) / (x_max - x_min)     
    plt.figure()
    ax = plt.subplot(111)
    for i in range(X.shape[0]):
        plt.text(X[i, 0], X[i, 1], str(digits.target[i]),
                 color=plt.cm.Set1(y[i] / 10.),
                 fontdict={'weight': 'bold', 'size': 9})
    if hasattr(offsetbox, 'AnnotationBbox'):
        ## only print thumbnails with matplotlib > 1.0
        shown_images = np.array([[1., 1.]])  # just something big
        for i in range(digits.data.shape[0]):
            dist = np.sum((X[i] - shown_images) ** 2, 1)
            if np.min(dist) < 4e-3:
                ## don't show points that are too close
                continue
            shown_images = np.r_[shown_images, [X[i]]]
            imagebox = offsetbox.AnnotationBbox(
                offsetbox.OffsetImage(digits.images[i], cmap=plt.cm.gray_r),
                X[i])
            ax.add_artist(imagebox)
    plt.xticks([]), plt.yticks([])
    if title is not None:
        plt.title(title)

#----------------------------------------------------------------------
## Plot images of the digits
n_img_per_row = 20
img = np.zeros((10 * n_img_per_row, 10 * n_img_per_row))
for i in range(n_img_per_row):
    ix = 10 * i + 1
    for j in range(n_img_per_row):
        iy = 10 * j + 1
        img[ix:ix + 8, iy:iy + 8] = X[i * n_img_per_row + j].reshape((8, 8))
plt.imshow(img, cmap=plt.cm.binary)
plt.xticks([])
plt.yticks([])
plt.title('A selection from the 64-dimensional digits dataset')
## Computing PCA
print("Computing PCA projection")
t0 = time()
X_pca = decomposition.TruncatedSVD(n_components=2).fit_transform(X)
plot_embedding(X_pca,
               "Principal Components projection of the digits (time %.2fs)" %
               (time() - t0))
## Computing t-SNE
print("Computing t-SNE embedding")
tsne = manifold.TSNE(n_components=2, init='pca', random_state=0)
t0 = time()
X_tsne = tsne.fit_transform(X)
plot_embedding(X_tsne,
               "t-SNE embedding of the digits (time %.2fs)" %
               (time() - t0))
plt.show()
```
