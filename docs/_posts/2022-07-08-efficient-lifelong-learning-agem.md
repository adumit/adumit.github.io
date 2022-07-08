---
layout: post
title:  "Paper read: Efficient Lifelong Learning with A-GEM"
date:   2022-07-08 16:00:00 +0200
categories: paper-reads
---

# Efficient Lifelong Learning with A-GEM

## Introduction - Lifelong learning

A-GEM is a method to do better at lifelong learning. This particular setting is seen as a stream of tasks and data points in those tasks and is asked to acquire new skills and knowledge quickly with a small amount of data. The learner should not forget prior knowledge in this endeavor and must also be constrained in compute and storage required. In other words, the learner cannot simply store all previous knowledge and replay it and cannot run computations that cannot be done in real- or close-to- realtime.

## Vocabulary

Di = dataset for task i, and each dataset consists of a series of tuples (x, t, y) where x is the input data, t is the task descriptor, and y is the label of that point.

TCV  = Tasks that can be used for cross-validation 

TEV  = Tasks that are used for evaluation

F𝚹 = The function mapping from X x T -> Y - the network that maps our inputs, x and t to the label space, y.

## Claims

The authors claim that their work accomplishes two major innovations.

1.  A-GEM has a better trade-off than other algorithms between average accuracy and computational/memory cost.
    
2.  All algorithms improve their ability to learn new tasks quickly when provided with a compositional task descriptor.

## Learning protocol

Prior work relies on a learning protocol that closely matches supervised learning, but violates our assumptions of a lifelong learner. 

Prior protocols break the same sequence of T tasks into training, validation, and test sets. This breaks the learning protocol because we foreseeably have some data leakage about the structure of a task. Further, if we choose hyper-parameters based on a training set of data, we further the data leakage for evaluation time.

![A-GEM image 1]({{site.url}}/assets/images/AGEM/agem-1.png)

In essence, their new learning and evaluation protocol splits along the “task” axis instead of the dataset axis. In other words, we do not split up any dataset between training, and validation/test.

## Metrics for learning

### Task accuracy

ak, i, j (a at k, i, j) is a value between 0 and 1 that defines the accuracy on the test set of task j after the model has been trained on the i-th minibatch of task k. This gives us a way to communicate about the accuracy of the current task (when j == k) and prior tasks (when j < k). BK is the total number of mini-batches in DK.

### Average accuracy

![A-GEM image 2]({{site.url}}/assets/images/AGEM/agem-2.png)

### Forgetting Measure

![A-GEM image 3]({{site.url}}/assets/images/AGEM/agem-3.png)

In english, the forgetting measure tells us how much information has been forgotten by comparing the current accuracy based on training up to task k for task j compared to the maximal accuracy the model achieved at any time in the past on task j.

### Learning Curve Area (LCA)

![A-GEM image 4]({{site.url}}/assets/images/AGEM/agem-4.png)

This is a function of the number of mini-batches per task. In other words, a high Zb where b is low would mean that the model learns a new task very quickly (with few mini-batches of the task). This metric does not appear to take into account forgetting on previously seen tasks so should be contrasted with the forgetting measure when evaluating overall performance.

### A-GEM Method

GEM is the method that A-GEM is based on. In GEM, we store an episodic memory MK for each task k. We’ve seen the episodic memory previously, though the precise definition can differ. Based on the author’s repo [here](https://github.com/facebookresearch/agem/blob/45421499483b28935491251e9e821c55e8b3c089/utils/utils.py#L294), it appears they are simply randomly sampling from the entire task dataset for each task.

GEM treats the prior losses as inequality constraints on our new update where we want to update our parameters for our current loss such that the prior loss on prior tasks is less than or equal to what it was before. It is:

![A-GEM image 5]({{site.url}}/assets/images/AGEM/agem-5.png)

To accomplish that task, GEM checks the gradient on our current task against the gradient from each of our prior tasks by vectorizing the gradient across all parameters. If the angle between our current gradient and any of those prior gradients is >90 degrees, we then project our new gradient into the closest L2 norm gradient that keeps the angle in our bounds.

Formally, this optimization problem becomes: 

![A-GEM image 6]({{site.url}}/assets/images/AGEM/agem-6.png)

Basically, the inner-product of our new gradient and prior gradient should be >= 0.

Intuition:

As a small example to illustrate the point and build intuition, I show a small example below.

-   I ran a 2-parameter linear regression without a constant. This means there are 2 trainable parameters in this “network” and we can look in 2D space.
    
-   I have two examples in the dataset: [1, 0.1] and [0.1, 1] with labels 0 and 1 respectively. My loss function is MSELoss and I train with the Adam optimizer to take steps along the gradient as opposed to more straight forward optimization.
    
1.  The first step in training is to train on example 1 -- X=[1, 0.1] and y = 0. We obviously get very low loss and our gradient is [0.0193, 0.0019] for parameter index 0 and index 1 respectively. This serves as our episodic gradient memory for the first example (we then take an optimization step and achieve almost 0 loss).
    
2.  Now, for the second gradient step, when we run forward and compute the gradient we get gradient values of [-0.1614, -1.6141] respectively.
    
3.  Taking the inner-product between these two values, it is -0.0062, which violates our constraint.
    
4.  Visually, we can see the angle is >90 also.
    
5.  ![A-GEM image 7]({{site.url}}/assets/images/AGEM/agem-7.png)
    
6.  In the many gradients case, correction would be a quadratic program (from the authors), but in our single gradient case we can use the update in the paper.
    
7.  ![A-GEM image 8]({{site.url}}/assets/images/AGEM/agem-8.png)
    
8.  ![A-GEM image 9]({{site.url}}/assets/images/AGEM/agem-9.png)
    
9.  ![A-GEM image 10]({{site.url}}/assets/images/AGEM/agem-10.png)
    
10.  And… we get a new projected gradient as seen above!
    

Now, intuitively, what happened? Well, in our simple example, to keep the loss LTE what it was, we cannot take a negative step in the X direction since the gradient wrt the loss is positive in the X direction. However, for our new example, we need to take a large step in the Y direction. This new projection accomplishes both those things.

Interesting note: indeed, when we take a step along the projected gradient instead of the new gradient, our loss for example 1 goes up less. However, it does still go up some!


#### Back to A-GEM…

Now, in the above intuition section, we only had one other prior example. But, in a real world case we’d have many. GEM handles this via a large set of constraints. A-GEM simply averages all of the gradients in our episodic memory to construct a single vector, thus reducing it to the constraint shown in the intuition section.

This turns it into ensuring that the average episodic memory does not increase.

![A-GEM image 11]({{site.url}}/assets/images/AGEM/agem-11.png)

The authors say that the main difference here is the level of guarantees between GEM and A-GEM. GEM has better worst-case guarantees in terms of forgetting whereas A-GEM has better average guarantees.

#### Joint Embedding Model Using Compositional Task Descriptors

Typically we have our inputs X and labels Y. When we have a task description, we also introduce a new component T which is a per-task descriptor. The shape of each tk in T is Ck x A, where Ck is the number of classes in that task and A is the number of attributes per class.

Essentially, the goal is to learn a separate function that maps the class attributes to an embedding space, which can then be consumed for classification per task - meta-information per task that allows easier transfer of knowledge. In this paper, both this function and the one for the inputs are feed forward networks. The learned embedding for the task descriptor is, of course, shared across all examples in a task.

![A-GEM image 12]({{site.url}}/assets/images/AGEM/agem-12.png)

#### Results

The tasks, in brief, are to split up traditional datasets (CIFAR-100, CUB, or AWA) or to edit MNIST (permuted MNIST) and train on different splits of the data as though they were different tasks.

The joint-embedding piece that they add here are some high-level descriptors of the images (attributes of the classes such as type of beak or color of feathers for bird images) or integer task descriptors. The latter of these don’t actually make any sense.. From the code here: [https://github.com/facebookresearch/agem/blob/master/fc_permute_mnist.py](https://github.com/facebookresearch/agem/blob/master/fc_permute_mnist.py) it seems that the integer task descriptors are not used at all and there is no “joint embedding model” for the images without additional features. In fact, integer task descriptors sound just like the label.

We can also see from the results that they do not show the -JE models for MNIST or CIFAR.

![A-GEM image 13]({{site.url}}/assets/images/AGEM/agem-13.png)

![A-GEM image 14]({{site.url}}/assets/images/AGEM/agem-14.png)

The author’s claim about the trade-off between time/memory for accuracy appears to be supported. A-GEM shows either the best or close to the best outcomes on forgetting and accuracy, while requiring significantly less compute and memory on the tasks here.

Their second claim about learning new tasks more quickly with the JE models is evidenced in the second set of plots where the LCA10 is much higher for JE models meaning they more quickly learn the task.

![A-GEM image 15]({{site.url}}/assets/images/AGEM/agem-15.png)

We can observe the forgetting curves for CIFAR and MNIST in the first row. Lose a few points of average accuracy overtime compared to ~10% on CIFAR. Interestingly in CIFAR, the prog-nn does slightly worse overall, which I don’t really understand.

![A-GEM image 16]({{site.url}}/assets/images/AGEM/agem-16.png)

Finally, here, we see that the zero-shot performance of the JE models is hugely better than without speaking to the utility of the JE models and how well the information transfers from task-to-task.

#### Thoughts

The overall idea of GEM seems quite promising and A-GEM makes the problem reasonably computable so this seems promising.

I am absolutely not convinced that their JE models are anything special. It just seems like more data and very high quality data at that since it concretely describes the task. I would need to look into tasks where they use it more concretely to understand what is going on there, but glancing at their code, there were 85 features per task for the AWA task and 312 features for the CUB task. This seems like a pretty significant increase in useful information and adding this in via a separate NN body does not seem novel.