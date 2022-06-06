# Adaptive Attention Span in Transformers
Link: https://arxiv.org/abs/1905.07799

## Goal
Extend the context window of transformers by allowing it to be adaptive. This technique also allows for controlling the memory footprint and computational time.

## Where does the current model break down?
![Adaptive attention image 1]({{site.url}}/assets/images/adaptive-attention/image-1.png)
Figure 1 shows the attention values over the prior context for a next-character prediction. The above access pattern differs significantly from the assumed performance. Head A uses almost nothing from the early context and almost exclusively uses the latest context whereas head B uses much more of the early information. This means that Head A could be relatively equally performant given a much smaller context.

## How do they solve it?
They propose a soft mask function:
![Adaptive attention image 2]({{site.url}}/assets/images/adaptive-attention/image-2.png)

This function maps a distance to a value in the [0, 1] range. The function is parameterized by a value z, which is in the range [0, S], where S is the length of the attention span. The shape of the function is shown in Figure 2.

![Adaptive attention image 3]({{site.url}}/assets/images/adaptive-attention/image-3.png)

This function allows a dynamic mask to be computed, which allows for variable computation. The hyperparameter R controls for the average size. The attention weights can be computed with the masked span as follows:
![Adaptive attention image 4]({{site.url}}/assets/images/adaptive-attention/image-4.png)
Finally, to regularize the z-values, they add an l1 penalty on the parameters.

## How well does it work?
They experiment over test8 and enwik8, which both have ~100MM tokens. They are running character level models so they report bit per character as their metric.

![Adaptive attention image 5]({{site.url}}/assets/images/adaptive-attention/image-5.png)
They see reasonable increases in absolute performance while significantly decreasing the required FLOPS, which is a great outcome for what they are trying to achieve.

![Adaptive attention image 6]({{site.url}}/assets/images/adaptive-attention/image-6.png)
The results from enwik8 support the same conclusion as text8.

![Adaptive attention image 7]({{site.url}}/assets/images/adaptive-attention/image-7.png)
They are also able to demonstrate that the performance continues to increase as they increase the span limit and increase faster as well. 

Nicely for their method too, the average span doesn't actually increase much at all (middle figure). This demonstrates that there is some information far away, but not too much and certainly every head doesn't need to attend that far. 

Finally, one other intesting result from the paper is that early layers don't attend as far. The information is only combined in the later layers.
![Adaptive attention image 8]({{site.url}}/assets/images/adaptive-attention/image-8.png)
Figure 4 shows the length of attention spans as layers progress.

## Conclusion
This was a very concise paper with one clear idea that they were able to test on a sufficiently large dataset to make it convincing. This kind of work supports the fact that later versions of these large language models could have significantly larger context windows. I think this kind of adaptive attention is likely more realistic than the Linformer kind of variant.
