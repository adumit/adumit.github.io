---
layout: post
title:  "Paper read: Improving Contrastive Learning by Visualizing Feature Transformation"
date:   2022-05-17 16:00:00 +0200
categories: paper-reads
---

# Improving Contrastive Learning by Visualizing Feature Transformation

Link: https://arxiv.org/abs/2108.02982

## Goal
This paper tries to find a way to improve contrastive learning processes by visualizing the distribution of positive and negative examples and scoring them relative to each other. They then use this method to enable novel feature transformation methods that ultimately lead to improved performance on downstream tasks. The intend to show that this works by finding harder positives and more diverse negative examples to improve the learned embeddings.

## How does it work?
![FT-SSCL image 1]({{site.url}}/assets/images/FT-SSCL/image_1.png)

Figure 1 shows the score distributions of positive examples. Their intention is to demonstrate that harder positive examples (as indicated by the lower score) take longer to converge. In (c) when they apply the method from (b) to decrease the scores they see  improved performance.

### Defining the method
The authors define a feature transformation process as manipulations on the latent embeddings produced by the encoder embeddings h<sub>q</sub> and h<sub>k</sub> to improve contrastive learning. In , !\[my previous post on contrastive learning\](), this appears to map to the basic setup of contrastive learning and directly learning on the embedding space. Figure 2 from the paper demonstrates this visually: 
![FT-SSCL image 2]({{site.url}}/assets/images/FT-SSCL/image_2.png)

The score they use for the feature transformation process and to get to their intended outcome is simply the cosine similarity between the l2 norms of the two feature vectors produced by the model.

### Visualizing what it does

![FT-SSCL image 3]({{site.url}}/assets/images/FT-SSCL/image_3.png)

Figure 6 visually demonstrates how the negative interpolation and positive extrapolation work. For negatives, they basically interpolate between two negatives on the unit sqhere. This ultimately takes the form of: 
![FT-SSCL image 4]({{site.url}}/assets/images/FT-SSCL/image_4.png)

For positives, they push them away from each other via the projection process defined as: 
![FT-SSCL image 5]({{site.url}}/assets/images/FT-SSCL/image_5.png)
where the z's are the transformed features. The only constraint is that they want the score of the z_hats to be > than the score on the original to make more difficult positives. To ensure this, they only require that lambda<sub>ex</sub> >= 1.

Both lambda<sub>ex</sub> and lambda<sub>in</sub> are sampled via Beta distributions that they parameterize with equal alpha and beta parameters. Lambda<sub>ex</sub> is 1+the beta distribution though to ensure it is >=1.

## Why do we need this method?
The authors makes the case that current self-supervised contrastive learning (SSCL) approaches rely on human intuition to create the augmentations. This has the downside of being limited in our ideas and not make full use of what the model could tell us about how to improve its learning. The authors believe that from the score distributions, we can trace backwards through the training process and uncover new types of feature augmentations.

Figures 3, 4, and 5 demonstrate how the model can go awry during the training process.
![FT-SSCL image 6]({{site.url}}/assets/images/FT-SSCL/image_6.png)
Figure 3 shows the negative and positive score distributions over the course of training for different momentum values for the MoCo model. The authors make the observation from figure 3 that the constancy of the mean and variance of the positive and negative scores is an important attribute of a good learner. They draw this conclusion because the low values of the momentum (that have the significantly spikier graphs) perform poorly.

![FT-SSCL image 7]({{site.url}}/assets/images/FT-SSCL/image_7.png)
Figures 4 and 5 continue to contribute to this picture with a bit more detail. Figure 4 shows how the gradient becomes untrainable for the lower values of momentum and that performance significantly degrades even when the model is trainable (m=0.6).

Figure 5 shows again the same views as figure 3, but with a little more precision because it is only for the highest momentum values, which perform well. Figure (5c) especially demonstrates the ultimate principle the authors want to communicate: that the lower momentum creates easier positive examples. That is, the lower momentum creates positive examples with higher scores -> greater similarity -> less information passed to the model. This is because a lower momentum means we get more of the latest parameters of the model when we update it. The authors liken this relationship to the InfoMin principle: "Raising the view variance between zq and zk+ corre- sponds to increasing the mutual information for contrastive learning, which forces the encoder learns a more robust em- bedding and thus improves the transfer accuracy".

## How much do these changes help?
### Positive extrapolation and negative interpolation
They demonstrate why they choose extrapolation for positives instead of interpolation in Table 3 below: 
![FT-SSCL image 8]({{site.url}}/assets/images/FT-SSCL/image_8.png)
Ultimately, interpolation would actually make the positives easier, which they claim would make less robust features because the positives would be easier.

For the negative interpolation, they appear to find that the values they choose to parameterize the beta distribution doesn't matter all that much beyond a point.
![FT-SSCL image 9]({{site.url}}/assets/images/FT-SSCL/image_9.png)

### How large of a queue to keep?

![FT-SSCL image 10]({{site.url}}/assets/images/FT-SSCL/image_10.png)
Prior work had shown that keeping a queue of negative examples is helpful. However, it's costly. The authors use a union of two queues via interpolation to achieve the final performance. The FT here stands for feature transformation, which are queues with feature transformations within the queue rather than just the base images. The only FT here is interpolation and because we only see a very small improvement by including the original queue, we can actually keep the size down and improve our runtime by using the FT queue.

### When should you add in the transformations?
Right at the beginning!
![FT-SSCL image 11]({{site.url}}/assets/images/FT-SSCL/image_11.png)

### How well does it do?
![FT-SSCL image 12]({{site.url}}/assets/images/FT-SSCL/image_12.png)

Tables 7 and 8 show adding in pieces of the experiment for 7 and training for longer in table 8. Adding all of the components in is definitely an improvement. And, interestingly, training for longer actually narrows the improvement of the method over other methods.

And finally, Table 9 shows performance across a whole range of tasks and across other models:
![FT-SSCL image 13]({{site.url}}/assets/images/FT-SSCL/image_13.png)
This is a great result for thinking about applying this in new scenarios!