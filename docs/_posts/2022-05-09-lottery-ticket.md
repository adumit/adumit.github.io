---
layout: post
title:  "Paper read: The Lottery Ticket Hypothesis - Finding Sparse, Trainable Neural Networks"
date:   2022-05-09 12:00:00 +0200
categories: paper-reads
---

# The Lottery Ticket Hypothesis: Finding Sparse, Trainable Neural Networks
Link: https://arxiv.org/abs/1803.03635

## Summary 
This paper seeks to understand the theory of what makes pruning networks so hard and to provide a plausible explanation for why overparameterization is so helpful. The central hypothesis, the lottery ticket hypothesis, is that large networks contain subnetworks that equally easily reach the high test accuracy that we would otherwise expect from the entire network. The paper covers a range of experiments that slowly build up this hypothesis from very simple networks to more complex ones.

## The lottery ticket hypothesis in action
**The Lottery Ticket Hypothesis:** A randomly-initialized, dense neural network contains a subnet- work that is initialized such that—when trained in isolation—it can match the test accuracy of the original network after training for at most the same number of iterations.

![LT image 1]({{site.url}}/assets/images/LotteryTicket/image-20220508120833776.png)

Figure 1 shows the lottery ticket hypothesis in action. For each of the architectures, we can see that it takes longer to train to a point of early stopping, but also that accuracy can remain very high for very sparse networks! Even at 7% sparsity the accuracy is comparable to the full network. And, from 60 - 80% or so sparsity, it appears that most of the networks actually improve in accuracy.

## Initialization: what makes the lottery hypothesis work
The most important component of the lottery ticket hypothesis ultimately turns out to be the initialization of the subnetworks. The authors demonstrate this by finding the subnetworks that perform well and then randomly reinitializing those subnetworks. When they do this, the network no longer trains well.

This leads to their process for identifying winning tickets:

![LT image 2]({{site.url}}/assets/images/LotteryTicket/image-20220509162550236.png)

The above method is 1-shot - just do the pruning once. The authors ultimately cover the one-shot case as well as the iterative case, which seeks to perform the above procedure iteratively and continually making the network more sparse.

## Simpler results from the paper
![LT image 3]({{site.url}}/assets/images/LotteryTicket/image-20220509164047314.png)

Figure 3 shows the accuracy on Lenet, a relatively smaller and simpler architecture. What's really amazing here is that a network with just ~1/5th of the original parameters actually improves accuracy and trains faster than the original network!

![LT image 4]({{site.url}}/assets/images/LotteryTicket/image-20220509164428160.png)

In figure 4 we can see the comparison with the random re-initialization method as well as iterative vs. oneshot pruning. Compare the blue and green lines, which are the oneshot and iterative methods, to the orange and red lines. The blue and green lines clearly beat out the random initialization. However, it's interesting that the random re-init of the subnetworks actually doesn't do all that bad!

## More complex results
To make the case more compelling, the authors move onto more complex architecutes than Lenet. Here, I skip over a few of the smaller convnets.

![LT image 4]({{site.url}}/assets/images/LotteryTicket/image-20220509164748164.png)

In figure 7, we can see a larger network, VGG-19, which was a big winner in terms of architecture in vision for some time.

And finally, we get to a Resnet-18.

![LT image 5]({{site.url}}/assets/images/LotteryTicket/image-20220509164846595.png)

In both cases above, it's amazing to see how well the lottery tickets perform. Especially compared to the random re-initialization.

## Interesting nuances
- The initialization is the key, but it's not because they are close to their final values. One might think that it's almost like the network is already trained! In fact, it's the opposite. The weights in the subnetwork actually move further away!
	- ![LT image 6]({{site.url}}/assets/images/LotteryTicket/image-20220509165614562.png)
- The winning tickets do seem to encode some amount of positive inductive bias towards the task at hand. In other words, the structure of the winning ticket does encourage good performance on the task.
- Winning tickets do seem to generalize better!
- The lottery ticket hypothesis overall is a possible explanation for why enormously over-parameterized networks are easier to train. In fact, the authors cite a work that shows that overparameterized two-layer relu networks converge to global optima for some problems when trained with SGD.
	- Could this be why the bigger and bigger models we're training today are doing better? Are they simply just getting more tickets?