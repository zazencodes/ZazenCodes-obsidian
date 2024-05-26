---
date: 2023-08-10
tags:
    - book
hubs:
    - "[[math-stats]]"
---
# Think Bayes
Author: Allen Downey

## Baye's theorem

```
p(H|D) = p(H) x p(D|H)
         --------------
             p(D)
```

Where H is the hypothesis and D is the data. In words - this means
- The **posterior** probability of a hypothesis H, given data D
- Is the **prior** probability of that hypothesis
- Multiplied by the **likelihood** of the data, given the hypothesis
- Divided by the probability of the data, under any hypothesis (**normalizing constant**)

## Beta distribution and conjugate priors
- Doing a bayesian update with a binomial likelihood function P(D|H) and a beta distribution prior P(H) will yield a beta distribution posterior P(H|D)

## Probability density functions
- Probability mass functions (PMFs) are discrete
- Probability density functions (PDFs) are continuous
e.g. Gaussian with mean 0 and std 1
```
f(x) = exp(-x^2/2)
       -----------
       sqrt(2 x PI)
```

## Poisson processes
- A process is a stochastic (having some kind of randomness) model of a physical system
- Poisson processes are continuous versions of Bernoulli processes
- Events can occur at any point in tie with equal probability (either it happens, or it doesn't, i.e. 1 or 0, like Bernoulli coin flips)
- Looking at charts below, take k = time
	- PDF shows how poisson distribution can model events that are likely to occur soon after each other (low lambda) or longer (as lambda increases)
	- CDF shows how the total probability of the event happening over time approaches 1, at a rate determined by lambda
- PDF

![[Pasted image 20230810095252.png]]
- CDF
![[Pasted image 20230810095308.png]]

## Observer bias
- Students think that classes are bigger than they are because more of them are in big classes
- Gym members think that everyone who goes to the gym is more fit because fit people spend longer at the gym

## Variability
- When comparing variability between groups, it's better to use the standard divided by the mean
- It's a dimensionless measure of variability relative to scale
- e.g. heights of men VS women
	- 178 +/- 7.7 and 163 +/- 7.3
	- men have more variation in absolute terms
	- but in normalized terms (dividing my the mean) the both become nearly the same: 0.04

## Population VS sample means
- For a normal process, if you plot the entire population then you'll get a Gaussian distribution with parameters *mu* and *sigma*
- If you draw *n* samples from this and get sample mean *m*, then the distribution of your samples will be a Gaussian with parameters *mu* and *sigma*/sqrt(*n*)

