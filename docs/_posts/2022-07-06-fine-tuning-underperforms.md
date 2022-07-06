# Fine-Tuning can Distort Pretrained Features and Underperform Out-of-Distribution
Link: [https://openreview.net/pdf?id=UYneFzXSJWh](https://openreview.net/pdf?id=UYneFzXSJWh)

# Goal

Demonstrate that full fine-tuning does a worse job at generalization than “linear-probing” (which is only tuning the final head layer). Additionally, they seek to demonstrate part of why they suspect this phenomenon exists. The main challenge is that work on full fine-tuning is more difficult than linear probing because there is simply much more going on.

# How do they test their hypothesis?

The authors tested on ten distribution shift datasets to specifically test the OOD performance of fine-tuning and linear probing. Their high level results are summarized in Figure 1.

![Fine tuning underperforms 1]({{site.url}}/assets/images/ft-up/image-1.png)


The authors test 3 variants:

1.  Fine-tuning: begin with a randomly initialized head and a pretrained model and fine-tune end-to-end. This is the variant they claim “distorts” the pretrained features.    
2.  Linear probing: begin with a randomly initialized head and freeze the body of the model.
3.  LP-FT: linear-probing and then fine-tuning. In this variant, they start by training only the head and then fine-tune the entire model. They claim this helps with the feature distortion.

# Key intuition
How does fine-tuning reduce the OOD accuracy?
- Essentially, it comes down to fine-tuning making updates that are “orthogonal” to the OOD data. It does this because the ID data is, by definition, partially orthogonal to the OOD data and thus the updates nearly necessarily update in some negative ways.
- In the example below, fine-tuning updates the feature extractor along the x-dimension. This means that while both the linear-probing and FT variants get the right answer (y), on the OOD data, fine-tuning has distorted the features.

![Fine tuning underperforms 2]({{site.url}}/assets/images/ft-up/image-2.png)

# Theory

![Fine tuning underperforms 3]({{site.url}}/assets/images/ft-up/image-3.png)

This theorem gives the lower bound of the OOD error as fine-tuning iterates. The main points to consider are:
-   The larger the difference between the initial pretrained feature extractor space (R0) and the S|, the larger the OOD error. This is very intuitive: if the features of the OOD data are substantially different than the ID data (from the pretrained model), then we’ll generalize even worse as we finetune.    
-   This bound is also impacted by the head alignment error, which is denominated by phi.

## Proof Sketch

So, what does the above theorem actually mean for our fine-tuning process? The proof in their paper seeks to establish that we can improve that lower bound via solely linear probing or (by extension) LP+FT. Their sketch follows as such:
- Start from the understanding that when we randomly initialize a new head, the weights of that head are very far from the optimal set of weights.
- To achieve good error, we know that the set of weights on the head will have to approach the optimal set of weights at some time t in the optimization.
- During that approach, while we are fine-tuning, the weight updates are coupled between the head and the body of the model.
- This means that the further our initial point is, the further the pretrained features will shift during training. This invariably leads to poor OOD performance because we have shifted the subspace of our pretrained features meaning we do worse for whichever subspace we no longer perform on.

Question:
- Does this implicitly assume that the ID data is further from the OOD data than the pretrained data? E.g. if the ID data is somehow a better subspace fit than the pretrained data, we might do worse.
	- Perhaps this is alleviated with a more specific definition of ID and OOD.

## How does LP+FT resolve from the above proof?
If the head and body updates are coupled, then simply improving the head performance before starting fine-tuning will naturally cause less disturbance to the base model features.

# Experiments
They test on 10 datasets that explicitly model distribution shifts in the training and test sets.
## In-distribution experiments

![Fine tuning underperforms 4]({{site.url}}/assets/images/ft-up/image-4.png)

## Out-of-distribution experiments

![Fine tuning underperforms 5]({{site.url}}/assets/images/ft-up/image-5.png)

## More challenging situations
- Worse pretrained feature extractor -> fine-tuning is better than linear probing.
	- When the quality of the pretrained feature extractor goes down, both fine-tuning and linear probing do worse. In this case though, fine-tuning is better.
	- They used a worse version of a model trained on a standard dataset as their pretrained model. They then updated that model.
- When ID ~= OOD, fine-tuning > linear probing, but LP-FT is better than both.
	- They compared a model trained on CIFAR (ID) on data collected in CIFAR (10.1), which is a dataset that followed the same methodology as CIFAR.

# Conclusion

- All of their experiments were on image data. Results may be somewhat different for NLP or other tasks. However, their theory should be generalizable.
- If you’re going to look into fine-tuning a model, definitely try linear-probing and then fine-tuning! You have almost nothing to lose it seems.
- The fact that LP+FT ultimately results in improved OOD performance seems a little counterintuitive given the proof they describe.
	- If this is the case, then it seems like there is some benefit of ID -> OOD performance.
	- It points more towards a concept like overfitting the ID data. If we always truly perturbed the body features negatively by fitting ID data, then wouldn’t LP always win?