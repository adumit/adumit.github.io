---
layout: post
title:  "Paper read: MultiSWAG - Bayesian Deep learning and a probabilistic Perspective of Generalization"
date:   2022-05-08 12:00:00 +0200
categories: paper-reads
---

# MultiSWAG: Bayesian Deep learning and a probabilistic Perspective of Generalization

#### Marginalization

“The key distinguishing property of a Bayesian approach is marginalization, rather than using a single setting of weights.”

What is marginalization?
-   It is the process of using all settings of parameters, weighted by their posterior probabilities.
-   This process stands as opposed to the frequentist point of view in which we use only a single set of parameters of our weights.

![Marginalization-integration]({{site.url}}/assets/images/MSWAG/MSWAG-post-figure-1.png)

-   In mathematical terms, we are integrating over all of the possible weights to get the final distribution of our outputs given the inputs and the data.
-   This is also known as a Bayesian Model Average (BMA)
    

#### The two axes of generalization

“From a probabilistic perspective, we argue that generalization depends largely on two properties, the support and the inductive biases of a model.”

Inductive biases: the probability we would generate a dataset if we were to randomly sample over our prior space. Essentially, how likely is our data given our model, or, in another view, how well does the data fit the world our model creates?

Support: the range of possible datasets for which our model can conceive of.

![Figure 2]({{site.url}}/assets/images/MSWAG/MSWAG-post-figure-2.png)

-   Figure (2a) shows three different models that cover 3 quadrants on the two axes of generalization.
-   For the linear model (blue curve) we see that all of the probabilities over its parameters gather weight around one version of the data in the wrong area. It gathers in one location because it is a simple model and thus its prior hypothesis space is very small (it can only consider a few cases well). And it is in the wrong space because its biases do not fit the data well.
-   For the MLP model, it spreads it’s hypothesis space out very widely because it is highly parameterized and thus has a very large prior hypothesis space. However, it is not concentrated usefully for this data because it has no useful biases towards a particular kind of data.
-   The CNN model has stronger inductive biases towards a certain kind of data (images) and thus concentrates closer to image datasets in this hypothetical space.
-   In 2b, c, d we see what happens when we train a model and update it’s parameters to the posterior distribution. The stronger inductive biases allow us to more easily narrow our hypothesis space to a tighter bound around the true model. But, if the true model does not lie in our hypothesis space at all, we will never get there.

Note: The authors use the above two-axis generalization scheme to explain why Gaussian Processes (GPs) tend to produce simpler models. Though they have large support, they have inductive biases towards simple, linear models even with a complex kernel.

### The MultiSWAG approach

#### Introduction to SWA: 

Post can be found [here]({{site.url}}/paper-reads/2022/05/08/SWA.html)

#### Introduction to SWAG (Stochastic Weight Averaging-Gaussian or SWA-Gaussian):

From: [https://arxiv.org/pdf/1902.02476.pdf](https://arxiv.org/pdf/1902.02476.pdf)

SWAG is essentially a higher-order method of SWA in which we keep track of a low-rank approximation of the covariance between the weight matrices introduced in SWA to induce a better final approximation.

If each epoch of an SGD training scheme is viewed as drawing from the distribution of weights that could parameterize this space then another view of SWA is to think of it as drawing from the distribution of N(weights_sgd, 0). In this view we can only draw the mean distribution of weights over SGD iterates. 

SWAG approximates the variance of this normal distribution of weights to 

![Figure 3]({{site.url}}/assets/images/MSWAG/MSWAG-post-figure-3.png).

This approximation can be written as:

![Figure 4]({{site.url}}/assets/images/MSWAG/MSWAG-post-figure-4.png)

Note: this is the unbiased estimator of the covariance of a distribution. It is the sum of squared deviations from the mean.

The only difference in SWAG is that they use a low rank approximation of this final covariance matrix by keeping D columns.

![Figure 5]({{site.url}}/assets/images/MSWAG/MSWAG-post-figure-5.png)

In terms of uncertainty estimation, this gets us an even better version than SWA because we can now draw many samples of the weights with slight deviations using our D matrix. Thinking back to the loss surface discussed in the SWA paper, this lets us marginalize over that entire loss surface as opposed sampling a single point - the SWA mean (that was better than any individual trained point).

#### Introduction to Deep Ensembles:

From: [https://arxiv.org/pdf/1612.01474.pdf](https://arxiv.org/pdf/1612.01474.pdf)

This paper has two primary contributions: demonstrates using deep ensembles scalably and provides criteria under which they should be conducted. The two criteria are using proper scoring rules and utilizing adversarial training. 

The important piece to understand for MultiSWAG is that each component of a Deep Ensemble is itself a DNN trained from scratch on the dataset and then the final model is averaging the predicted probabilities.

#### MultiSWAG:

![Figure 6]({{site.url}}/assets/images/MSWAG/MSWAG-post-figure-6.png)

##### MultiSWAG combines SWAG with the multiple model concept from Deep Ensembles.

-   We can see in the above picture what MultiSWAG is doing. It trains multiple models from scratch and then does a SWAG (or SWA for MultiSWA!) approximation for each one.
-   This means we almost certainly hit multiple basins of attraction.

##### How does this fit into the picture of marginalization?

-   In order to get the best approximation to the marginal distribution, we need to consider as many possible models and especially as many different basins of attraction since those are the locations of the highest density of our posterior probability.
    
#### Returning to neural network priors and generalization

“A prior over parameters p(w) combines with the functional form of a model f(x;w)to induce a distribution over functions p(f(x;w))”

-   The authors argue this case by pointing to recent results on image networks where a fully untrained CNN still is able to do a good job predicting images using only a simple learned model on top. The CNN has good inductive priors for images.
    
-   This is further supported by taking a simple, untrained CNN, and running images of a particular class through it. They show that the logits outputs across instances of that class are more correlated than with other classes, again showing us that CNNs have useful priors.

Towards generalization, we can now understand it as a 2-dimensional concept. This generalization concept also applies to other model classes, such as Gaussian processes.

#### Defeating Double Descent

![Figure 7]({{site.url}}/assets/images/MSWAG/MSWAG-post-figure-7.png)

We’ve previously read about a phenomenon called double descent where a network will first get better at test error in the “classic” regime and then get worse, “overfit”, but then return to getting better.

-   This was especially apparent in any instance with corrupted labels.
-   The authors demonstrate that SWAG demonstrates this phenomenon to a lesser extent and MultiSWAG appears to completely eradicate it.
-   This is specifically the case when we can increase the number of SWAG models sampled from.
    

#### Temperature applications and Bayesian models

The standard Bayesian posterior distribution is normalized by a factor, Z.

![Figure 8]({{site.url}}/assets/images/MSWAG/MSWAG-post-figure-8.png)

For Bayesian deep learning we have a temperature scaled distribution:

![Image 9]({{site.url}}/assets/images/MSWAG/MSWAG-post-figure-9.png)

T < 1: cold posteriors in which we more strongly concentrate around high likelihood values - upweight more likely weights.
T = 1: Standard Bayesian posterior
T > 1: warm posteriors where we allow the prior to “speak” more and affect the posterior more strongly - this is due to the p(w) term not being affected by T comparably.

#### Temperature and model misspecification

“Are we in a misspecified setting for Bayesian neural networks? Of course. And it would be irrational to proceed as if it were otherwise. Every model is misspecified.”

“Moreover, for virtually any realistic model class and dataset, it would be highly surprising if T= 1 were in fact the best setting of this hyperparameter. Indeed, as long as it is practically convenient, we would advocate tempering for essentially any model, especially parametric models that do not scale their capacity automatically with the amount of available information.”

-   Since we cannot pretend as if the model is not misspecified, tempering helps us better reflect our real beliefs instead of pretending the model is not misspecified.
-   Example from diagnosis agent work - optimal temperature encouraged a warm posterior because the model is certainly misspecified for interview data since it was only trained partially on that type of data. Thus, our real beliefs should be substantially less confident.
-   “For parametric models such as neural networks, it is to be expected that the capacity is particularly misspecified.”

#### Effects of the prior

-   This discussion concerns what the prior function, p(w), should be and what effect it has on the posterior. The possible functions under consideration are:
-   p(w) =N(0,I)
-   p(w) =N(0,α2I)
-   Each of these describes how we sample the prior weights of our networks.
-   The authors point out that for a large range of values of alpha, we do not collapse into a particular class distribution, which was a problem discussed in a discussed paper.
-   Figure 9 and 10 demonstrate this phenomenon and what happens to the posterior.

![Figure 9]({{site.url}}/assets/images/MSWAG/MSWAG-post-figure-10.png)

-   First, figure 9 shows what varying the value of alpha does to the prior distribution. Each of these plots demonstrate the predicted probabilities when varying how we sample those weights. The problem discussed in the paper is evidenced here by the fact that the sqrt(10) value of alpha causes a high prediction value in one class (far right plots).    
-   There is a sweet spot for this value that can be found via cross-validation.

![Figure 10]({{site.url}}/assets/images/MSWAG/MSWAG-post-figure-11.png)

-   However, even though figure 9 may be worrying because we need to then cross-validate this additional parameter, figure 10 shows us that this prior is not very strong and as we see additional data points the effect of this particular aspect of the prior is not too strong.
-   All of the above plots are post-training on the number of data points under the same prior (from the appendix). The predictions are then averaged across the dataset.

#### Performance of high performing functions in loss-landscape space

![Image 12]({{site.url}}/assets/images/MSWAG/MSWAG-post-image-12.png)

-   Figure 11 shows us the loss landscape of the posterior on 4 independently trained models on the same set of data. We can see that each model fits into the single low-loss valley.     
-   This is confirmation of earlier papers and experiments around why SWA/SWAG work.
-   Taking the average of these models or possibly even just the weights here will provide better uncertainty estimate on out-of-distribution data as well as do better on within distribution data because we will find a better location in the loss landscape to sit.
-   We can visualize the better out-of-distribution sample as the blue part of the curve that shows uncertainty.

#### Performance in the presence of noise and data corruption  

![Image 13]({{site.url}}/assets/images/MSWAG/MSWAG-post-image-13.png)

Figure 5 demonstrates this phenomenon on one type of noise and as we increase the number of models.

Figures 14 - 19 further demonstrate the performance across these 3 ensembling techniques on various types and levels of noise and corruption.
-   We can plainly see from these plots that the ensemble models do better and MultiSWAG appears to outperform the other techniques.    
-   It especially does better than the single model and even just two models already provides substantial benefit.
    
How well do these approaches approximate true BMA?

![Image 14]({{site.url}}/assets/images/MSWAG/MSWAG-post-image-14.png)

Figure 4 shows that Deep Ensembles do a better job of approximating true BMA as the number of models increases.