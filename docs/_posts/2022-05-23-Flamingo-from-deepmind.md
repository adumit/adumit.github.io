---
layout: post
title:  "Paper read: Flamingo: A Visual Language Model for Few-Shot Learning"
date:   2022-05-23 16:00:00 +0200
categories: paper-reads
---

# Flamingo - A Visual Language Model for Few-Shot Learning

Link to paper: [https://storage.googleapis.com/deepmind-media/DeepMind.com/Blog/tackling-multiple-tasks-with-a-single-visual-language-model/flamingo.pdf](https://storage.googleapis.com/deepmind-media/DeepMind.com/Blog/tackling-multiple-tasks-with-a-single-visual-language-model/flamingo.pdf)

### Goal of the paper
#### What are they trying to improve upon or what question are they trying to answer?
The authors are trying to create a general purpose visual-language model that is a few-shot learner. They are trying to be able to use independently trained vision-only and language-only and fuse them together to then fine-tune on only a small set of text-image pairs. They can then use this to answer really interesting prompts that combine both images and language.

### Methods
#### What is the method they are using/improving upon?
Prior methods would only produce a similarity score over text and video or need to be fine tuned on a very large amount of that data. The Flamingo model is able to fuse LLMs with video models - this fusion is where the novel architecture is. There isn't exactly a prior method here although they are certainly using constrastive Learning.

They are expanding prior work in the visual-language model space. There is actually quite a bit of prior work here so this isn't a brand new idea by any stretch. Part they took from elsewhere:
- Start with a pretrained LM
- Transformer based mapper between vision encoder and frozen LM
- Training of cross attention layers interleaved with the frozen LM layers

#### What is the main advance in their method?
What are the challenges with multimodal generative modelling?
- Taking a pretrained LM and allowing it to handle images somehow.
	- To handle this, they introduce new layers that handle the multimodal input and cross attention layers that can learn to attend to those new inputs along with current text. They then introduce some new gating mechanisms as well to learn how to bring that input in.
- Difficult to support images and video
	- They use a Perceiver style architecture for the video/image input (only or for everything?) that can limit the latent space and parameters needed for something as complex as video.
- How to obtain heterogeneous training data to induce good generalist capabilities
	- They've created a new dataset that scrapes text and images from webpages, which they combine with standard image/text and video/text datasets already out there.

Creating better data to have a very large multimodal dataset seems like it is probably the biggest advancement here.

Ultimately, three main contributions:
- New architecture along with training strategies. These support the video + image + text modalities.
- SoTA in many visual-language tasks.

Overall architecture:
![Flamingo image 1]({{site.url}}/assets/images/Flamingo/image-1.png)

###### Process for training
- Start with pretrained models.
	- For the LM, they can use a model already out there.
	- For the vision encoders, they separately train that with contrastive text-image approach. The goal of the vision encoder is to extract relevant semantic features like color, shape, nature, positions, etc.

###### What are the perceiver resample modules doing?
![Flamingo image 2]({{site.url}}/assets/images/Flamingo/image-2.png)
The perceiver resampler is basically a way to better handle video most importantly. It uses the variable length video along with time embeddings to turn a video into a single latent vector.

###### What about the gated attention layers?
![Flamingo image 3]({{site.url}}/assets/images/Flamingo/image-3.png)
These are the layers that help integrate the visual side of the world. The "gated" part means that they have a parameter that says how to linearly combine the cross-attention and self-attention (frozen layers from the LLM). These parameters start at 0 such that the output is initially exactly what the LLM would produce. Then, over the course of training they smoothly transition to integrate the image/video inputs.

When they are handling video or multiple images in the text, they ensure that the text side of the model can only attend to the latest image. Each image still has dependence on prior images, but this allows generalization to any number of prior images rather than a fixed look-back window.

### Results
#### What are the main results they use to justify their work?
Many different examples of few-shot learning including SoTA compared to fine tuned models. They cover quite a few different tasks here. 9 image related tasks and 9 video related tasks!

#### Do you believe these results are convincing?
Very much so. I think the diversity of tasks is quite impressive and the demos are fantastic.

### Discussion
#### Where could this be applied?
- I really like the idea of inserting untrained attention layers that can integrate a new mode of data. That seems like a relatively straight forward way to incorporate a new kind of data.
- Further, just the concept of starting from a pretrained model and augmenting it seems quite powerful as a technique. I wonder if that would work well for answer predictor by just adding a pretrained head?

#### What kinds of problems would this work well for?
- It seems like a really interesting idea to incorporate new sources of information via a cross-attention layer that starts as a gated layer so it doesn't immediately break a pre-trained model. It sounds like a very safe way of augmenting.

#### Interesting notes:
- When they had multiple datasets, they found that accumulating the gradients and taking 1 step was much better than a round-robin approach.
- They prepend their text sequences with a space 50% of the time because the tokenizer can tokenize words differently depending on if there is a space in front of it.