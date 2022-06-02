---
layout: post
title:  "Paper read: Linformer: Self-Attention with Linear Complexity"
date:   2022-06-01 16:00:00 +0200
categories: paper-reads
---

# Linformer: Self-Attention with Linear Complexity
Link: https://arxiv.org/abs/2006.04768

## Goal:
Reduce attention complexity from O(n<sup>2</sup>) to O(n). The main slowdown in modern transformers comes from multiplying two `n*d` matricies, the queries and keys, together where N is the sequence length and d is the dimensionality of the embedding. 

## Method overview
- Project the original attention into multiple smaller attentions through linear projections.
- Thereby forming a low-rank approximation of the attention mechanism.
- This is acceptable because they will seek to prove that attention is a low-rank operation thus can be well approximated by a low rank factorization.
- This accomplishes the following relative speed up:
	- ![Linformer image 1]({{site.url}}/assets/images/linformer/image-1.png)

## Demonstrating that attention is low-rank
Their method relies on the fact that attention is a low-rank operation so the commit a substantial amount of their paper to demonstrating this.

![Linformer image 2]({{site.url}}/assets/images/linformer/image-2.png)

The above plots detail the cumulative eigenvalues for the 512 dimensions. They draw lines at the 128th largest dimension and show that ~>90% of the distribution is captured. The right-most plot shows a heatmap of the different layers and heads and also shows that >85%+ of the value is captured by the cumulative eigenvalue at index 128.

## Method in detail

![Linformer image 3]({{site.url}}/assets/images/linformer/image-3.png)

Ultimately, it comes down to running the K and V matricies through the projection matricies. These matricies are `n*k` and they can project down from `n*d` to `k*d` after the multiplication. By choosing a small k, which they show through the Johnson–Lindenstrauss lemma is acceptable, they show that they can remove the dependence on n.

They also allow for a few additional efficiencies via parameter sharing. The sharing happens within the projection matricies. They can share across all heads in a layer, they can just share in key and values, or headwise sharing. They experiment with these variants.

## Results

![Linformer image 4]({{site.url}}/assets/images/linformer/image-4.png)

Their first result is the pretraining validation perplexity of different Linformer variants (different context windows), compared to the standard transformer, and different sharing variants.
- The sharing variant seems to have minimal effect.
- Surprisingly, the larger context window does not seem to do much good.
- The Linformer does very well compared to the standard transformer.
- A larger k seems to make little difference. (Would have been nice to try significantly smaller K's too!)

![Linformer image 5]({{site.url}}/assets/images/linformer/image-5.png)

On downstream tasks, their model performs just about as well as a standard transformer. They do test a ton of different small deviations in their model so it's hard to say that the times when they "win" it's actually due to the model and not just the hyperparameters.

Nonetheless, comparable performance coupled with decreased inference time is great! It does seem like they should have tested significantly increased context windows if the Linformer reduces space and time complexity as much as they claim.

![Linformer image 6]({{site.url}}/assets/images/linformer/image-6.png)

They ultimately speed up inference time by 1.5x and allows for a 1.7x larger maximum batch size. As we increase the context window, the speed-up and memory savings are even more dramatic.

## Conclusion
This was a concise paper with one really clear idea to test. I wasn't able to find any folks that tested scaling this up significantly. It feels like this would be another great way to try out really large context windows.

Another idea that came out of reading this - could you apply a low-rank factorization and trainable parameters to a pretrained transformer? Allow it to simply learn how to project it's keys and values down? Probably a significant amount of effort because all the parameters are dependent on N. So, perhaps it always takes training from scratch.