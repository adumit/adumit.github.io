# Language Modeling Via Stochastic Processes
Link: https://openreview.net/forum?id=pMQwKL1yctf

## Goal
The authors was to improve long-text coherence. Current large language models do a poorer and poorer job maintaining consistent tone and content the longer text they are asked to generate. The authors propose a stochastic process based method to improve long term coherence by attempting to make sure it is consistent with a latent plan.

## Method in brief
The authors propose a method they call Time Control. This method assumes a simple, fixed dynamics model with goal-conditioned generation.
- They assume that text generated without a goal can be represented as Brownian motion in latent space.
- Brownian motion will ensure that sentences close to each other are similar, but sentences far apart are likely very different.
- We can introduce goal-directed behavior by conditioning on a fixed start and end point. With this goal, the motion of the sentences becomes a Brownian bridge and the latent trajectories abide by closed-form dynamics.
- They derive a contrastive objective for learning the latent space with these bridge dynamics.
- Their method, Time Control, then plans the bridge ahead of time in latent space and then the model will generate text conditoned on that bridge, which allows it to maintain local and global coherence.

## Method details
![Language modeling via stochastic processes image 1]({{site.url}}/assets/images/lm-via-sp/image-1.png)
Figure 1 shows a visual deptiction of their method. They use triplets of points as a dataset. The start and end of a conversation act as "anchors" in latent space (x<sub>0</sub> -> z<sub>0</sub>) and (x<sub>T</sub> -> z<sub>T</sub>). Then, z<sub>t</sub> should be embedded within the green oval shape and closer to mu<sub>t</sub> (the expected projection of our point to a spot in latent space between the two end points) than a random sentence from another conversation x'. 

### The brownian bridge process
![Language modeling via stochastic processes image 2]({{site.url}}/assets/images/lm-via-sp/image-2.png)
The above probability describes the brownian bridge equation. It is effectively a noisy interpolator where the point z<sub>t</sub> should be closer to z<sub>0</sub> at the start of the time and closer to z<sub>T</sub> at the end of the time.

### The loss they optimize
![Language modeling via stochastic processes image 3]({{site.url}}/assets/images/lm-via-sp/image-3.png)
Given a brownian bridge (read start and end points) and a point x<sub>t</sub> and negative examples x', we can optimize the process via a contrastive loss. Basically, we want the points in the document to better map to our brownian bridge and points outside the conversation to map away from it.

### How do they train?
They start from a pretrained autoregressive large language model. In this paper's case, they use GPT-2.
- They map all the sentences in the dataset to the learned latent space using the pretrained encoder.
- This automatically gives them the brownian bridge because they have the start and end points from the document and all of the middle points.
- They then finetune the decoder to generate text that is conditioned on the past context and the latent plan.
	- To accomplish this step, they simply require the model to predict the next token using all previous tokens (exactly the normal autoregressive task), as well as predict the sentence embedding z<sub>t</sub>. This is a form of reconstruction, but a little more difficult.
	- The contrastive loss objective comes into play here for the reconstruction objective.

### Finally, how do they generate new text?
![Language modeling via stochastic processes image 4]({{site.url}}/assets/images/lm-via-sp/image-4.png)
Figure 2 shows the overall ideas of the process. The biggest challenge is that we often do not know our end point! To get around this they take the following steps:
- First, encode a set of sentences that are start and end points from the training data.
- Second, fit a gaussian to the end points conditioned on the start points.
- Third, given a new start point, we can sample from the gaussian fit to the end points.

## How well does it do?
### On a discourse coherence task
![Language modeling via stochastic processes image 5]({{site.url}}/assets/images/lm-via-sp/image-5.png)
Table shows the quantitative results from three different datasets that measure discourse coherence. There are large standard errors here from only 3 runs so they have wide margins and bold as such.

### Can the model generate locally coherent text?
They measure this outcome by taking two sentences as start and end sentences. The job of the model is to generate a sentence that fits in between the two and maintains coherency.

![Language modeling via stochastic processes image 6]({{site.url}}/assets/images/lm-via-sp/image-6.png)
![Language modeling via stochastic processes image 7]({{site.url}}/assets/images/lm-via-sp/image-7.png)
Table 2 shows the BLEU scores for ground truth sentences vs. the generated sentences on a variety of models. Table 6 shows the human ratings for those generated sentences. In general, their model appears to perform well enough on the local coherence tasks.

### Global coherence
This is what this model really cares about. They measure this is a few ways.
- First, they take a dataset composed on Wikipedia sections and try to generate new sections.
	- They test whether the new sections are of appropriate length by assessing whether they are similar length to other sections in that document.
- Second, they force the model to generate long text on a conversation dataset.
	- This is a bit of a weird task to compare, but can lead to interesting results. Essentially, they disallow the model from predicting a stop token.
- Third, they measure whether the ordering in forced long text generation makes sense.

![Language modeling via stochastic processes image 8]({{site.url}}/assets/images/lm-via-sp/image-8.png)
Table 4 shows the percentage of deviation of the generated sections from other sections in the text. Lower is better because that means the generated section better matches the length of the real sections. Their method performs quite favorably here.

Table 7 shows the human evaluation results on the forced long text generation:
![Language modeling via stochastic processes image 9]({{site.url}}/assets/images/lm-via-sp/image-9.png)

And table 5 shows the evaluation of coherent ordering of forced long text generation.
![Language modeling via stochastic processes image 10]({{site.url}}/assets/images/lm-via-sp/image-10.png)

Table 8 shows an example of how current models go wrong and their model goes right:
![Language modeling via stochastic processes image 11]({{site.url}}/assets/images/lm-via-sp/image-11.png)

## Conclusion
Overall, the paper was interesting and I very much like the idea of coherence that they proposed using the Brownian bridge. Their methods of evaluation strike me as a little weird. Their method will miss out on, or at least wasn't evaluated on, global coherence across entire documents in terms of content.

The forced long-text generation task was also a little strange. If a typical language model predicts a stop token, then it's done. What we want is that if it starts generating again, it maintains good coherence as well, in my opinion. That would take the form of saying that pieces of text outside the context window are coherent as well, which this method does not account for.