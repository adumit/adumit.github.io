---
layout: post
title:  "Paper read: Large Language Models are Zero-Shot Reasoners"
date:   2022-05-28 16:00:00 +0200
categories: paper-reads
---

# Large Language Models are Zero-Shot Reasoners
Link: https://arxiv.org/abs/2205.11916

## Goal
This paper takes a close look at prompt engineering for zero-shot tasks. Most notably, it finds an <b>extremely</b> simple prompt that significantly increases the accuracy on zero-shot tasks: "Let's think step by step". They add this prompt before the answers the model outputs. This feels like "Just ask for reasoning". 

## Method
The entire method is adding "Let's think step by step." to the answer prompt for GPT-3.

An example would be:
> Q: What is (4 + 3) * 7
> <br>A: <b>Lets think step by step</b>

That's it! It's beautiful in its simplicity and appears to be able to accomplish quite a bit.

## Some of the really interesting results
Increasing the accuracy on MultiArith from 17.7% to 78.7%  
and GSM8K from 10.4% to 40.7% with an off-the-shelf 175B parameter model

![Reasoning image 1]({{site.url}}/assets/images/zero-shot-reasoning/image-1.png)
The example they chose is great! In the few-shot example, a human has to come up with the steps, but what if the human doesn't know the steps? The zero-shot outcome shows a very promising direction.

![Reasoning image 2]({{site.url}}/assets/images/zero-shot-reasoning/image-2.png)
Some datasets definitely see a much bigger impact than others.

![Reasoning image 3]({{site.url}}/assets/images/zero-shot-reasoning/image-3.png)

The left-hand example in Table 4 is really interesting. Is there some Bayesian reason going on here? E.g. if a child's room is believed to be in a house, then house is actaully a more accurate answer because it's inclusive of all instances of a child's room. Perhaps it ignores the base rate though.

Different prompts create different outcomes:

![Reasoning image 4]({{site.url}}/assets/images/zero-shot-reasoning/image-4.png)

It still amazes me how such small semantic deviations in the input result in very different thinking for the model. It might be interesting to take a smaller language model (that researchers have access to) and look at how the slight changes in input affect the representations for downstream tokens.

## Conclusion
This was a really interesting paper! I really like how small of an idea they studied, but then took that and studied the idea very well. It's fascinating how much potential for reasoning and creativity is locked within GPT-3 that can be "coaxed" out by human creativity.