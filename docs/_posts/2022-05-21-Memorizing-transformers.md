---
layout: post
title:  "Paper read: Memorizing Transformers"
date:   2022-05-21 16:00:00 +0200
categories: paper-reads
---

# Memorizing Transformers
Link: [https://arxiv.org/pdf/2203.08913.pdf](https://arxiv.org/pdf/2203.08913.pdf)

## Goal
Standard transformers have no obvious way to actually store past inputs other than in their internal weights by means of "memorizing" the training data or by past inputs literally being in the context window. This is certainly a desirable quality if we ever want to use these models to converse with us or to be able to make use of new information that they are told about. This paper seeks to introduce a new architecture that can look over the embeddings of past tokens in "memory" and optionally attend to them.

## How does it work?
![MT image 1]({{site.url}}/assets/images/MT/image_1.png)
- The bulk of the model is a vanilla transformer decoder
- They use a causal attention mask
	- What is a causal attention mask? Seems like it is from: "Longformer: The long-document transformer." which has a sliding-window causal mask.
- Token embeddings of the last layer are used to predict the next token.
- Largely follow the Transformer-XL style in the following ways:
	- Uses the Transformer-XL style cache, which holds the keys and values from the prior training step when operating over long documents.

### kNN augmented attention layer
A transformer layer near the top of the stack has a kNN-augmented attention layer, which combines two forms of attention.
- It has a standard dense self-attention layer on the local-context, which is the input sequence of the current training step.
- And, it also does a kNN lookup on the external memory.
- Each head keeps a cache of the prior M (key, value) pairs, where M is the memory size.
- For each token in the input subsequence, the kNN process will return the top-k (key, value) pairs.
	- "As with standard dense attention, we first construct an attention matrix by computing the dot product of each query against the retrieved keys, then apply softmax, and finally return a weighted sum of the retrieved values. Unlike standard dense attention, the retrieved memories contain a different set of (key, value) pairs for each query."
- The dense self-attention on local-context and the attention over the KV pairs from retrieved memory are combined via a gate:
	- ![MT image 2]({{site.url}}/assets/images/MT/image_2.png)
	- The dot-circles are element-wise multiplication, which allows each head to choose how to attend to the local-context or memories.

##### Some finer details
- They do not use position bias for the retrieved memories. There's really no positional information.
- They do not use a global position encoding for tokens. A prior work showed it doesn't work well for long documents. For the local-context they use the T5 relative position bias.
- There are multiple documents in each batch and they are chopped up into subsequences:
![MT image 3]({{site.url}}/assets/images/MT/image_3.png)
- The memory is cleared between each document because it is not relevant between them.
- During training, the memorized KV pairs can quickly become stale. To account for this, they normalize the KV vectors in memory to at least deal with differing magnitudes.
- They use approximate kNN rather than true kNN due to speed issues.
- They found that training on a small memory and then finetuning on very large memory to lead to better performance.

## Results
They train on a variety of datasets that all involve fairly long corpra. English language books, source code from GitHub, a very large collection of documents from the internet, formal math, and arXiv Math)
![MT image 4]({{site.url}}/assets/images/MT/image_4.png)
They demonstrate that having memory improves the model's perplexity and it is further improved with the XL cache, which helps maintain a better local-context.

They further demonstrate that the proper way to achieve a larger memory is by pretraining on a medium-large memory and then finetuning for a smaller number of steps on an even larger memory.
![MT image 5]({{site.url}}/assets/images/MT/image_5.png)

This sentence about scaling up the base model drives the point home: "External memory provides a consistent improvement to the model as it is scaled up. Remarkably, we found that the smaller Memorizing Transformer with just 8k tokens in memory can match the perplexity of a larger vanilla Transformer which has 5X more trainable parameters."

They also demonstrate that they can augment a pretrained transformer with the external memory.. the idea is a good one!
![MT image 6]({{site.url}}/assets/images/MT/image_6.png)

And finally, they do a small study to see which tokens the model is attending to in memory:
![MT image 7]({{site.url}}/assets/images/MT/image_7.png)
They look for tokens where the loss is significantly different between two memory sizes. They find that the model is finding lemma definitions in it's memory and that 