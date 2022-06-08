---
layout: post
title:  "Paper read: Transformer Language Models without Positional Encodings Still Learn Positional Information"
date:   2022-06-07 16:00:00 +0200
categories: paper-reads
---

# Transformer Language Models without Positional Encodings Still Learn Positional Information
Link: https://arxiv.org/abs/2203.16634

## What are they studying?
This paper studies whether the positional embeddings we pass to our language models are definitively helpful or not. Further, they seek to show that models trained without those embeddings still learn positional information. They study these effects through probing the attention outputs in a causal attention setting.

## How do position encodings work?
The positional encodings they test fall into one of the following possibilities:
1) Learned
	- These embeddings are used in both masked language models and large autoregressive models (e.g. GPT-3). They are learned per position of the context window.
2) Sinusoidal
	- These are constant vectors computed by a function. In this case, sine and cosine functions of differing frequencies. These were introduced in the original transformer.
3) ALiBi
	- This is a newer embedding that tries to introduce relative as well as absolute information. It adds in linear biases via a negative bias to attention scores that grows linearly with between pairs of tokens that are further away.

## How do they measure outcomes?
- WikiText-103 as their baseline dataset
	- WikiText-103 is a typical language model task with high quality.
- They use ThePile as a large scale dataset.
	- The Pile is an 800GB english text dataset from that includes Common Crawl as well as 22 other sources.
	- They use this dataset to test scaling effects on the model.

## Results
![Pos encodings image 1]({{site.url}}/assets/images/adaptive-attention/image-1.png)
- On perplexity across the two datasets, the no positional embedding model does comparably well.

![Pos encodings image 2]({{site.url}}/assets/images/adaptive-attention/image-2.png)
- The same story is true for different model scales. The same scaling effect seems to apply to the model without any positional information.

![Pos encodings image 3]({{site.url}}/assets/images/adaptive-attention/image-3.png)
- And the sequence length also seems to have little change as far as the relative improvement between models!

Overall, it seems like the model without positional encoding performs very similarly to the learned embedding model and exhibits the same behavior across the tested dimensions. The embeddings are only helping very marginally it seems.

They then examine if this model learns positional information within the transformer weights to compensate. The answer seems like a yes! They test this by training a 2-layer ReLU network to predict the position of tokens in a sequence.

![Pos encodings image 4]({{site.url}}/assets/images/adaptive-attention/image-4.png)
Figure 3 shows the predictions of that network from a random 64-token sequence. Overall, the model does a very good job at understanding position. Even though some of the predictions are wrong, only a few are moderately far off.

Finally, they show how the perplexity changes when you train with a masked-language model kind of task:
![Pos encodings image 5]({{site.url}}/assets/images/adaptive-attention/image-5.png)
In this version of a language model, there is very little innate positional information because the model is bi-directional. Thus, the model does terribly.



