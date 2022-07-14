---
layout: post
title:  "Paper read: Coordination among Neural Modules Through a Shared Global Workspace"
date:   2022-07-16 16:00:00 +0200
categories: paper-reads
---

# Coordination among Neural Modules Through a Shared Global Workspace
Link: https://openreview.net/pdf?id=XzTtHjgPDsT

## Goal
Improve coordination among different neural modules through a learned shared workspace. This is intending to mimic a cognitive science theory of mind, which proposes the shared workspace for various brain functions. In this paper, they implement it as a limited bandiwdth communication channel. They claim that this imposes limit encourages specialization and compositionality and facilitates the synchronization of otherwise independent specialists.

## Benefits of this model
#### Distributed specialists
- Desireable scaling properties: we can add more specialists relatively easily and they can be integrated into the global system seamlessly.
	- If we add additional modules in a traditional transformer architecture, we run into quadratic scaling since each module attends to every other one. With a shared workspace, we've essentially moved towards a linear cost.
- Increased robustness: we can remove to pull out individual specialists without losing general performance.
- Efficiency: information is processed locally for the most part, which enables efficient computation and avoids expensive coordination.

#### Shared workspace
- Follows a cognitive science theory called the Global Workspace Theory (GWT). In GWT, there is an idea of a "blackboard" that all of your neuro-modules write to and read from.
	- All of the modules also write in a shared language or representation of external and internal phenomena.
- This shared communication channel creates the need for shared representation since all of the specialists need to be able to interpret it.
	- In this paper, they train a system where the specialists are trained in coordination, which should lead to the shared language we desire.

## Implementing the global workspace
![Neural modules image 1]({{site.url}}/assets/images/coordinating-neural-modules/image-1.png)
The model is implemented within a transformer paradigm and uses key-value attention to establish the relationship and coherence between the different submodules.

The primary difference is, of course, the shared workspace. This means that instead of each submodule attending to every other one, certain submodules are active and write to the shared workspace. In the subsequent layers, other submodules can read from the shared workspace.

#### Adding information to the global workspace
They implement it as a kind of cross-attention mechanism for the global workspace. The queries are a function of the shared global workspace projected into query-dimension by a weight matrix. Then, the keys and values are a function of the specialists broadly. Then, they take a softmax over the representations to get the updated memory matrix. The softmax here creates a soft competition among the modules to only write the most useful things into the shared context.

The above steps can be summarized as:
![Neural modules image 2]({{site.url}}/assets/images/coordinating-neural-modules/image-2.png)
In this equation, Q is ![Neural modules image 3]({{site.url}}/assets/images/coordinating-neural-modules/image-3.png)

#### How specialists attend to the global workspace
They again utilize an attention mechanism. In this case, the specialists generate the queries and the keys and values are from the global workspace representation.

After receiving the broadcasted information from the shared workspace, each module independently updates it's own step. The authors liken this to a single step in an LSTM or a GRU or a feed-forward layer like in a transformer.

## Experiments
They choose a variety of tasks to test their model on including simple tasks: image recognition for equilateral triangles, relatively simple question answering; and more complex tasks: object tracking, atari video game transfer.

![Neural modules image 4]({{site.url}}/assets/images/coordinating-neural-modules/image-4.png)
Figure 3 and Table 1 show relative performance compared to prior work. Mostly they are transformer architectures with some aspect of sparsity. Their method shows modest gains over the current best architecture(s).

![Neural modules image 5]({{site.url}}/assets/images/coordinating-neural-modules/image-5.png)
Figure 4 shows that their model converges much faster and ultimately ends up with a slightly higher accuracy. This task is a combination of vision and questions about that image that are relational or non-relational. E.g. questions about a single object or relations between objects.
- The authors argue that their method is better at dealing with useful sparsity and that is the main improvement they are realizing.

To support the sparsity claim, they share Figure 5.
![Neural modules image 6]({{site.url}}/assets/images/coordinating-neural-modules/image-6.png)
Figure 5 shows the activation maps for their version (b) and the prior model (a). This variant is slightly different than a typical transformer, but the important part is the sparsity of the activation maps.

## Conclusion
This paper presents a very interesting idea for the global workspace. I liked that they brought in an idea from cognitive science and implemented it in a way that seems to make sense with the theory.

Their experiments were on some non-standard tasks, but perhaps the body of research in this area tends to use those tasks because the models perform well relative to other kinds of models. E.g. sparsity in activations and attention is a key feature.