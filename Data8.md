## Data Science 8
1. explore data, visualize, identify patterns
2. statistical inference, quantify patterns are reliable, use randomization
3. prediction using machine learning

#### Association and Causation
comparison between treatment group and control group to establish association. two groups but be as similar as possible aside from the treatment. If there is other systematic difference between the two groups, it could be difficult to identify causality. e.g. observational studies could create selection bias. there is underlying source that could lead to wrong conclusion, they are called **cofounding factors**.

association can be accurate as causality if there is no cofounding factors. randomization help reduce cofounding factors, called randomized controlled experiment.

random != haphazard

#### Probability
* probability of all outcome add up to 1
* P(A) = number of outcome that make A happen / total number of outcomes
* multiplication rule: P(A and B) = P(A) * P(B | A)
* addition rule: P(A) = P(A1) + P(A2) where A can happen in exactly one of 2 ways: A1 or A2

#### Sampling
* determinisitic sampling, doesn't involve chance
* probabilistic sampling, before sample is drawn, need to know the selection probability
  1. uniform random sample, each individual has an equal chance of being selected
  2. simple random sample is uniform and without replacement

#### Probability Distribution
a probability distribution describes all possible values of the quantity, and probability of each of those values

#### Law of Averages
if a chance experiment is repeated many times, independently and under the same conditions, then the proportion of times that event occurs gets closer to the theoretical probability of the event.

the empirical distribution of a sample resembles the distribution of the population given large sample size with high probability.

#### Statistics
probability is a branch of mathematics, where if you make assumption on how the world works, you can compute what will happen when running an experiment. statistics is taking the result of experiment and reason about the world. understanding through observation. sampling is look at the result of the experiment, and reason how would the result come out differently.

statistical inference is making conclusions based on data in ramdom samples. for example, use data to guess the value of an unknown constant. the estimate of the constant is based on the sample of the unknown quantity.

#### Probability Distribution of Statistic
value of the statistic vary because random samples vary. there is also a sampling distribution, or probability distribution, of the statistic, which shows all possible values of the statistic. the probability distribution of the statistic can be hard to calculate, either have to do the math, or have to genreate all possible samples and calculate the statistic based on each sample. 

we instead use the empirical distribution of the statistic that we get by simulating the estimation over and over again. this way we avoid the difficulty mentioned regarding obtaining the probability of the distribution of statistic.


#### Empirical Distribution of the Statistic
empirical distribution ofthe statistic is based on simulated values of the statistic, which consists of all the observed values of the statistic, and the proportion of times each value appeared. 

good approximation of the probability distribution of the statistic is obtained if the number of repetitions in the simulation is large.

#### Testing Hypothesis by assessment
if we can simulate data according to the assumptions of the model, we can learn what the model predicts. if the real data and the models predictions are not consistent, that is evidence against the model.

#### Comparing Distributions
Total Variation distance between 2 statistical distributions is the sum of absolute difference in each category divide by 2. This is the way to compare the difference between 2 distributions

#### Null Hypothesis
a well defined chance model about how the data were generated; we can simulate data under the assumption of this model, "under the null hypothesis"

#### Alternative Hypothesis
a different view about the origin of data. result came from outside of chance.

#### Test Statistic
the statistic that we choose to simulate, to decide between 2 hypothesis. question to choose the statistic:
1. what values of he statistic will make us lean towards the null hypothesis?
2. what values will make us lean towards the alternative? preferably the answer should be just high or just low, try to avoid both high and low

#### Prediction Under Null Hypothesis
simulate the test statistic under null hypothesis, draw historigram of the simulated values; display the empirical distribution of the statistic under null hypothesis. it is a prediction about the statistic made by the null hypothesis to show all the likely values of the statistic, and show how likely they are.

probabilities are approximate, because we can't generate all the possible random samples

once we have the prediction statistic, we can compare the observed test statistic and its empirical distribution under the null hypothesis. if the observed value is not consistent with the distribution, then test favors the alternative and rejects the null hypothesis.

Result is considered statistically significant if the observed result is within the 5% of the tail area of the distribution of the random sample. second convention is less than 1%, it is considered highly statistically significant. this is the `p value` of the test.

P-value is the chance, under the null hypothesis, that the test statistic is equal to the value that was observed in the data, or even further in the direction of the alternative.

statistical test don't always come to the correct conclusion. the cutoff p-value is an error probability. if the cutoff if 5%, and the null hypothesis happens to be true, then there is about a 5% chance that your test will reject the null hypothesis.

#### A/B Testing
two random samples, samples look different, trying to explain why. could this due to chance or sample came from 2 different population.

hypothesis is about whether samples coming from different population due to characteristic separation differ due to chance alone, or is somewhat related to the dividing characteristic.

the test statistic is based on the observation that if null hypothesis is true, then the label does not matter, thus rearrangements of the 2 groups e.g. randomly drawing samples from both groups at random would produce similar statistic. this is called **permutation test**.

process of permutation test:
1. shuffle the labeled population (originally divided by the label)
2. assign some from group A and rest from group B, maintain the two sample sizes
3. find the difference between the averages of the two shuffled groups
4. repeat

the average from the permutation test should be around 0.

#### Causality
testing of the statistic does not prove causality, only confirm association.

#### Percentile
definition vary. simply definition is the value in a aset that is at laest as large as a percentage of the elements in the set. for percentile that does not exactly correspond to an element, take the next greater element.

#### Inference: Estimation
make a informed guess on an unknown parameter. 

#### Variability of an Estimate
tells how accurate an estimate is. `estimate = parameter + error`

#### Bootstrap
way to reuse the same sample to determine the variability of the sample without having to do resampling. bootstrap assumes the sample has the same distribution as the original population.

bootstrap draw at random from first sample with replacement as many values as the original sample contained. size of the resample has to be the exact same as the original sample.

bootstrap is used to compute the 95% confidence interval, interval of estimates of a parameter based on random sampling. **confidence interval** tend to be at the level between 90 to 95.

when not to use confidence interval:
* estimate very high and very low percentiles, the minimum or the maximum, e.g. the exact 99 percentile of the population
* trying to estimate any parameter that greatly affected by rare elements of the population
* probability distribution of the statistic is not roughly bell shaped
* original sample is very small. fewer than 20 elements

confidence interval and hypothesis test is related when we want to use hypothesis test to determine the value of a statistic using only a sample.

method:
1. construct a 100-p% confidence interval for the population average
2. if x is not in the interval, reject the null
2. otherwise cannot reject the null

#### Fact about Confidence Interval
* the chance the statistic of the population is in the confidence interval is not the same as the confidence interval level. in fact the confidence interval of a statistic based on a sample does not mean the true statistic of the population is within the confidence interval at all
* confidence interval of example 95% only mean if sampling is done repeatedly, and confidence interval of the statistic is computed for each sample, then approximately 95% of the confidence intervals contain the statistic of the true population

#### Average
* the average of a dataset does not need to be a value in the dataset

#### Median
the exact data point that is in the middle if fully ordered
* if the distribution is symmetric about a value, then that value is both the median and the average
* if data distribution is skewed, then the mean is pulled away from the median in the direction of the tail

#### standard deviation (variance)
variability around the mean. help understanding where the data are

#### Chebyshev's Bounds
mean +/- 2 SD contains 75% of the data
mean +/- 3 SD contains 88.88..%
mean +/- 4 SD contains 93.75%
mean +/- 5 SD contains 96%

#### Normal Distribution
`z = (value - mean) / SD`
in standard normal distribution, mean = 0 and s.d. = 1

formula: `phi(z) = (1 / sqrt(2 * pi)) * e^((-1 / 2) * z^2)` for `-infinity < z < +infinity`

the function is symmetric about 0

for normal distribution:
mean +/- 1 SD contains about 68% of the data
mean +/- 2 SD contains 95%
mean +/- 3 SD contains 99.73%

#### Central Limit Theorem
if sample is large and is drawn at random with replacement, then regardless of the distribution of the population, the probability distribution of the sample sum (or sample average) is roughly normal distribution. that is, the probability distribution of the average of large samples (not just one sample) taken randomly with replacement is a normal distribution.

this only apply to sum and average, not others such as product.

#### Standard Units
converting data points from its original unit to be described by the mean and standard deviation

#### Correlation Coefficient
measures linear association based on standard units. the value is `-1 <= r <= 1`. when r = -1 or 1 it means there is perfect straight line, when r = 0 then the data points are uncorrelated linearly (could still be associated by other relationships)

r is the mean of the products of standard units (product of x, the input measurement in standard unit, and y the output measurement in standard unit).

correlation does not mean causation. it's possible to have cofounding factors in between.

the correlation coefficient does not do a good job representing how many outliers there are. also does not show whether correlation is linear or not. outliers have serious effect on correlation.

it is easy to produce correlation on average (clustered) data, but those correlations are artificial because the data points are not from the original population, but the mean of a sample.

#### Regression
example data set: Galton's data on height of children vs the parent.

**nearest neighbor regression** : predicting a numerical variable y given value of x
* identify group of points for which the values of x are close to the given value
* prediction is the average of the y values for the group

if input and output does not have any regression relation, then the correlation coefficient has value 0.

a simple regression estimate can be produced using standard units (using mean and stdanrd deviation) between input x and output y.

then multiply by r using the formula `estimate of y = r * x`

then convert the estimate to the original unit of y

the regression estimate is an average. on average, the value of y at a fixed x are closer to the mean than x is.

the error between predicted y and actual y is measured in **root mean squared error**. the regression line minimizes the root mean squared error. it is also called the least squared line.

#### Residuals
the error in regression estimates. `residual = observed y - regression estimate of y = observed y - height of regression line at x`

residuals can be used to identify if there is linear relationship. e.g. if a residual graph shows lower value and higher value of x have only positive residual and medium x values have negative residual then this is unlikely to have linear relationship.

the residual of a linear relationship should have roughly equal number of positive and negative residuals on all values of x

the mean of residual is always 0, regardless of original data

`variance of residuals / variance of y = 1 - r^2`
`variance of fitted values / variance of y = r^2`
`variance of y = variance of fitted values + variance of residuals`

when r = 1, then the regression line gives everything about the relationship and residual variance is 0

thus |r| = SD of fitted value / SD of y

#### Confidence interval for prediction of regression
* bootstrap the scatter plot
* get a prediction for y using the regression line that goes through the resampled plot
* repeat many times and draw the empirical histogram of all predictions
* get middle 95% interval. this is the confidence interval

confidence interval of regression depends on the x position, some position will have bigger confidence interval for y and some less depending on whether the number of input data around the area. the more input data for some certain area of the input would mean re-sampled regression line would vary less around the area and as a result having smaller confidence interval (more confident).

confidence interval of the slope the value for slope through bootstrap resampling. this is based on the assumption there is a slope, or a correlation between input and output. if the confidence interval of slope contains 0, then it is not possible to reject the possibility that there is no relationship.

we use confidence interval to hypothesis testing. the null hypothesis is the slope of the true line is 0. and alternative hypothesis is that it is not. if onfidence interval contains 0, then it is not possible to reject the null hypothesis.

### Nearest Neighbor Classification
from the entire dataset, find one historical data point that's closest to the new data point to make prediction based on it

### K nearest neighbor
find the K closest data point to input data point and determine the label

### Supervised Learning
dataset is dividied into:
* training set
* test set
* validation set

#### Source of Data
(US Census)[https://www2.census.gov/programs-surveys/]
