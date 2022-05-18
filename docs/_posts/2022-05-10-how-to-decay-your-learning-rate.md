---
layout: post
title:  "Paper read: How to decay your learning rate"
date:   2022-05-10 12:00:00 +0200
categories: paper-reads
---

# How to decay your learning rate

Paper link: [https://arxiv.org/pdf/2103.12682.pdf](https://arxiv.org/pdf/2103.12682.pdf)

## Goal

Create a learning rate scheduler that automatically adjusts according to the empirical phenomenon discovered by the authors. That phenomenon is named weight norm bouncing and the authors observe that we can automate the process rather than handcrafting schedules that adjust the learning rate implicitly based on this realization.

## The problem

In some problems in DL, the authors observe the problem for weight norm bouncing. This is defined as a monotonic decrease in weight norm followed by a monotonic increase in the weight norm.
  
![ABEL image 1]({{site.url}}/assets/images/ABEL/image_1.png)

In this figure we can see the weight norm bounce prior to some of the vertical lines.

The vertical lines specifically indicate points at which the learning rate is decayed. The authors make the observation here that complex schedulers tend to decay the learning rate when the weight norm bounces.

Why does weight norm bouncing occur? The authors postulate that it bounces due to the L2 regularization that is in place in most modern CV NNs. With L2 regularization, the weight norm initially decreases and then increases as the penalization becomes less prominent.

Ultimately, the authors claim that the presence of weight bouncing is a necessary, but not sufficient condition for complex schedulers to have an improved performance over simple ones (where we simply decay the learning rate once near the end of training).

![ABEL image 2]({{site.url}}/assets/images/ABEL/image_2.png)

Where t is time at the end of an epoch.

## The proposed solution

Algorithm 1 defines the ABEL scheduler. Essentially, it tests the weight norm bouncing condition in the top if-statement and then decays the learning rate by a decay factor until it reaches a minimum learning rate.

Thus, the only three parameters in the algorithm are the decay factor and the maximum and minimum LR. Empirical evidence!

![ABEL image 3]({{site.url}}/assets/images/ABEL/image_3.png)

Table 1 shows the primary results for comparing ABEL to other methods.

In general, ABEL does about as well as the hand-tuned Stepwise decay functions. Though there are instances where Cosine annealing does better, ABEL still performs quite well and comes with other benefits, which will be discussed later.

ABEL is also very robust to its hyperparameters, which is a nice trait.!

![ABEL image 4]({{site.url}}/assets/images/ABEL/image_4.png)

This table shows training for many epochs (90 for decay factor != 0.5 and 120 for 0.5) and how the final accuracy of a model trained with ABEL changes over those scales.

### Training curves for ABEL
  
![ABEL image 5]({{site.url}}/assets/images/ABEL/image_5.png)

We can see that ABEL tends to approximate the shape of the hand crafted stepwise functions without requiring hand tuning. 

Further, it ultimately achieves similar accuracy, which we see here and saw in table 1.

## Additional benefits of ABEL

-   Doesn’t require a fixed train budget. Other competitive LR schedulers such as cosine/linear decay depend on how much total training epochs you are willing to allocate. ABEL only depends on the weight bounce.    
-   As seen above, ABEL is more robust to its hyperparameters.

## For what problems is this helpful?

### What happens when there is no weight bouncing?

-   The authors ran experiments in the NLP and RL settings testing both simple and complex decay schedules.
-   In both of these settings, there is no weight norm bouncing and the authors use this as evidence that complex schedules are only helpful when there is weight norm bouncing.

![ABEL image 6]({{site.url}}/assets/images/ABEL/image_6.png)
![ABEL image 7]({{site.url}}/assets/images/ABEL/image_7.png)

Then, the authors run CIFAR-10 experiments without data augmentation. Without data augmentation, the model does not exhibit weight norm bouncing. And, even a simple learning rate schedule can reach 0 train error and competitive test error.

Ultimately, the authors conclude that weight norm bouncing is a necessary, but not sufficient condition for complex schedulers.

### Improving understanding of weight norm bouncing

-   Here is the changing weight norm equation under SGD updates:
    
![ABEL image 8]({{site.url}}/assets/images/ABEL/image_8.png)

-   eta is the learning rate and lambda the L2 regularization coefficient and gt is the gradient with respect to the loss.
-   The authors provide a simplification which allows us to discard the second to last term involving gt and eta.
-   With this simplification, we can see that the updates are determined by the gradient and the prior weight norm.
-   At the start of training, we expect the weight norm term to dominate, which leads to the decreasing weight norm.
-   Then, as the weight norm becomes small, we can expect the gradient term to dominate, leading to the bounce.
-   Though the above simplification does not always hold in practice, the authors state that any layer before a batch norm is scale invariant and so the term involving gt and wt will be on a smaller scale than the prior two terms.
-   The only necessary condition for weight norm bouncing is that L2 regularization is present and that the learning rate is large enough. In the figure below we can see that when the learning rate is not large enough, there is no weight norm bounce and the performance is worse (compared to a good scheduler + bounce).
    
![ABEL image 9]({{site.url}}/assets/images/ABEL/image_9.png)

-   The authors tentatively say that weight norm bouncing allows the model to explore more of the loss landscape, but they qualify this as requiring further research.
-   Decaying too early comes with a large negative effect since the weight norm never has a chance to bounce, thus reducing the space explored.
-   ![ABEL image 10]({{site.url}}/assets/images/ABEL/image_10.png)
-   And, alternatively, we can see that trying to initialize away from having a bouncing weight norm at all hurts performance. In figure 5b (above) we can see that the weight norm always bounces until the norm is so small it hurts performance.
-   A natural question is if all layers exhibit this behavior or only some layers do? The authors are using the weight norm of all layers together to decide when to decay the LR, so if only some layers are showing this behavior and others a different behavior, maybe something else is at play?

![ABEL image 11]({{site.url}}/assets/images/ABEL/image_11.png)

-   In this figure, we see that the weight norm is dominated by only 10 layers. In this sense, using the weight norm of the entire network suffices as these layers all seem to display some aspect of weight norm bouncing.
-   Note however, the most significant layer (in light yellow) has a weird behavior in the right plot, which tracks the weight norm at epoch t compared to that layer’s weight norm at initialization.

## Conclusions

-   In vision tasks with L2 regularization, try ABEL.
-   In other tasks, check if your model exhibits weight norm bouncing. If so, maybe try ABEL. If not, possibly best to try a simple learning rate schedule with a single decay step towards the end of training.
-   Adam still exhibits weight norm bouncing.
-   This phenomena only seems to occur in “hard” datasets, so the process for continued theoretical progress is difficult here. See figure below for difficult vs simple tasks. The simple task is CIFAR-10, difficult is CIFAR-10 with data augmentation.

![ABEL image 12]({{site.url}}/assets/images/ABEL/image_12.png)

## Thoughts
    
-   The authors do not study warmup, but simply include it where it is useful. Empirically, I have found warm up super useful and almost necessary for my own models, so probably keep that in. According to the authors, warmup is fairly independent of the learning rate schedule anyways.
- The overall strain of research here is an interesting direction. The authors appeared to have observed a phenomenon when training networks themselves and sought to recreate it. What is perhaps challenging here is that they chose problems which exhibited bouncing and constructed a decay mechanism that recreated the decays when bouncing so ultimately their method was self-serving. There is some evidence that it works for new problems, but I'm not sure it's studied widely enough within this paper to definitively prove that they did not just identify a spurious phenomena.
