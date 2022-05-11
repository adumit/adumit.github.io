---
layout: post
title:  "Paper read: A survey on self supervised contrastive learning"
date:   2022-05-11 16:00:00 +0200
categories: paper-reads
---

# A Survey on Contrastive Self-Supervised Learning

Link: [https://arxiv.org/pdf/2011.00362.pdf](https://arxiv.org/pdf/2011.00362.pdf)


## Goal
-   Self-supervised learning is popular because you can use unlabeled data.
-   Contrastive learning is a branch of self-supervised learning that has seen a significant amount of success.
-   The objective of contrastive learning is to find a useful embedding space of the data inputs by grouping similar inputs together and dissimilar inputs further apart.
-   This paper covers commonly used tasks for pre-training as well as a performance comparison across methods.
    
## What is contrastive learning?

![SSCL survey image 1]({{site.url}}/assets/images/Survey-SSCL/image_1.png)

At its core, contrastive learning is the simple idea that examples that are “similar” should be close together in embedding space compared to those that are “dissimilar”. For images, this tends to take the form of data augmentation methods that distort or alter a base image to some degree and then compare that to the original image as a similar example. We can then use images from other classes as “negative” examples that should be dissimilar.

![SSCL survey image 2]({{site.url}}/assets/images/Survey-SSCL/image_2.png)

Prior to contrastive learning, generative models in general were becoming more popular as pre-training methods. GANs could be used to improve task performance by generating labeled data or by simply swapping in the generator as a pre-trained model. However, training them was unstable and hard to do well.

![SSCL survey image 3]({{site.url}}/assets/images/Survey-SSCL/image_3.png)

As the field has gotten better at contrastive learning, the best methods are now competitive with supervised alternatives!

## How do we accomplish contrastive learning?

### Pretext tasks

Pretext tasks are the pre-training tasks that do the bulk of the work of contrastive learning. They typically take the form of some kind of data alteration or augmentation. The most common ones for images are color transformation and geometric transformation. For some tasks, there are context-based tasks, and then there are cross-modal tasks as well for images/text, for example.

#### Color transformation

![SSCL survey image 4]({{site.url}}/assets/images/Survey-SSCL/image_4.png)

#### Geometric transformations

These transformations include: scaling, flipping, random cropping, etc.

![SSCL survey image 5]({{site.url}}/assets/images/Survey-SSCL/image_5.png)

#### Context-based: Jigsaw puzzle

![SSCL survey image 6]({{site.url}}/assets/images/Survey-SSCL/image_6.png)

You apply the jigsaw-ing to both the base image and the variant. But, the variant is the one that is scrambled. Then, a different image with jigsawing applied would be the negative image.

#### Context-based: Frame ordering

For any data that extends through time, you could consider a re-ordering of the frames a way to do data augmentation.

#### Context-based: Future prediction

Predicting future or missing information is another strategy to create positive samples. The goal is not to predict full future inputs, but to predict high-level future states. This means you compress the original input down and then try to compare near and future states together.

#### View prediction (Cross modal-based)

![SSCL survey image 7]({{site.url}}/assets/images/Survey-SSCL/image_7.png)

View prediction is typically used when you have multiple views of the same scene. You want views at the same point in time to be similar and future views to be different.

#### NLP: Center and Neighbor word prediction

This is a pre-text task for self-supervised NLP. This is basically predicting a word from context or predicting a masked word in a sentence.

#### Next and Neighbor sentence prediction

Same as above except now the goal is to predict the entire next sentence or the sentence before.

#### Auto-regressive language modeling

This is predicting the next word in the sequence over and over. This could be longer than a sentence and could continue for long sequences of text.

#### Sentence permutation

Take a series of sentences and permute them randomly. The model’s goal is to predict the original ordering.

### How do you identify the right pre-text task?

It’s important to identify the right pre-text task because that is what your model will learn to be invariant to. For example, applying rotation could be useful in some settings, but would be a very poor choice for detecting which way is up in a photograph. Further, in vision, some pretext tasks produce artifacts in how the data is learned.

Short answer: it’s an art.

## Architectures

![SSCL survey image 8]({{site.url}}/assets/images/Survey-SSCL/image_8.png)

The four main architectures are depicted above.

### End-to-end learning

Any system where all of the modules are differentiable is an end-to-end system. These systems tend to prefer larger batch sizes to see more negative examples (all examples that aren’t the model and its augmentation are considered negative) as well as longer training times. SimCLR was a standout model in this space.
![SSCL survey image 9]({{site.url}}/assets/images/Survey-SSCL/image_9.png)

To the left, we see SimCLR’s performance with a linear classifier trained on the learned representation. Increased batch size and more epochs clearly contribute positively to outcomes.

A large challenge with these models is actually training them effectively with larger batch sizes as well as a problem with large mini-batch optimization problems.

### Using a memory bank

A memory bank serves as a way to accumulate a large number of feature representations from samples that can be used as negative examples. These are smaller in memory than the full inputs and also take less time to compare since you don’t need to do any forward passes.

![SSCL survey image 10]({{site.url}}/assets/images/Survey-SSCL/image_10.png)

The main challenge with a memory bank is that the representations go out-of-date very quickly. After just a few batches it can be possible that the representations are no longer comparable.

### Using a momentum encoder

A momentum encoder is a copy of the encoder network that uses a moving average (based on momentum) of the query encoder and acts as the key encoder. Only the query encoder is now updated via backprop. The advantage here is one of space and time. Space because you only have to train one encoder and time for the same reason. You also get the advantage of having the current and prior states of the saved encodings be relatively similar.

### Clustering feature representations

Clustering feature representations makes use of the concept that higher level features should be similar and grouped together. The challenge with prior contrastive work is that if you have a cat image, there might be cat images in the same batch that would otherwise be seen as negative examples. With a CFR approach, the goal would be to also have those instances be similar because they share many of the same features.

![SSCL survey image 11]({{site.url}}/assets/images/Survey-SSCL/image_11.png)

## Encoders

The decision of what encoder to use is important for any self-supervised method. For vision tasks, ResNet-50 is the most popular encoder - it has a good balance between size and capacity. Occasionally, some methods further project the dimensionality of the final representation of an encoder to smaller dimensions and take the contrastive loss in that space.

## Training

Overall, training takes the form of trying to push similar items closer together and dissimilar items further apart. Cosine similarity tends to form the basis of most contrastive learning setups.

![SSCL survey image 12]({{site.url}}/assets/images/Survey-SSCL/image_12.png)

Contrastive learning typically uses the Noise Contrastive Estimation (NCE) as the basis for comparison:

![SSCL survey image 13]({{site.url}}/assets/images/Survey-SSCL/image_13.png)

Here, q is the original sample, k+ is the positive sample and k- is the negative sample. Tau is a hyperparameter that is often called the temperature coefficient. However, when you use many negative examples, which is very common, then the above loss is not sufficient and instead we use InfoNCE:

![SSCL survey image 14]({{site.url}}/assets/images/Survey-SSCL/image_14.png)

Basically, this just combines a number of negative examples, but weights them, in total, equal to the positive example.

## Performance

The way the performance of these methods is typically measured is by taking the learned representations and training a linear or non-linear classifier on-top of them while either freezing or fine-tuning the encoder.

![SSCL survey image 15]({{site.url}}/assets/images/Survey-SSCL/image_15.png)
![SSCL survey image 16]({{site.url}}/assets/images/Survey-SSCL/image_16.png)
![SSCL survey image 17]({{site.url}}/assets/images/Survey-SSCL/image_17.png)

### How can we visualize what these are learning?

We often look to feature maps or visualizing kernels as well as nearest neighbors. Nearest neighbors are simply to understand if the model is learning intuitive relationships between neighbors.
![SSCL survey image 18]({{site.url}}/assets/images/Survey-SSCL/image_18.png)

### Performance in NLP
![SSCL survey image 19]({{site.url}}/assets/images/Survey-SSCL/image_19.png)

## Limitations and future directions

Recent research has shown that contrastive methods may miss out on key components of images or videos. One study found that a couple methods missed out on viewpoint, which is crucial for some downstream tasks.

### Lack of theoretical foundation

The architectural design and sampling techniques have a profound effect on performance. This makes the method highly dependent on the pretext tasks that are chosen during training. Thus, we need more theoretical foundations to build on to make these selections less error prone.

### Proper negative sampling during training

“Easy” negative samples are ones that are very clearly different from the target. Most negative samples are easy. When you mostly train on easy samples, the negative side of the loss does not contribute meaningfully to the total loss. 

There has been some work to do “hard” negative mining from the dataset, but doing so introduces a huge number of new parameters, hyperparameters which are all specific to a particular dataset and may not or can not generalize to other datasets.
