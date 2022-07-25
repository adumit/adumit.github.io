---
layout: post
title:  "Paper read: Consequences of Misaligned AI"
date:   2022-07-25 16:00:00 +0200
categories: paper-reads
---

# Consequences of Misaligned AI
Link: https://arxiv.org/abs/2102.03896

## Goal
This paper makes the case that (almost) necessarily a reward function specified by a person for an ML algorithm is incomplete with regards to the true utility function that the human wants to be maximized. They use this to demonstrate what can go wrong and how the true utility can be negatively impacted by this partial specification. They then go on to describe how to go about designing reward functions in a dynamic and interactive way.

## The problem
The authors start with the analogy to the story of King Midas. He turns everything he touches to gold and so can no longer eat. This is analogous to autonomous systems. They give examples of automated recommendation systems as examples that have already caused significant harms. The engagement metrics are a poor proxy for what we (society) likely want to get out of it, which is enjoyment, truthfulness, and informative.

![Misaligned AI image 1]({{site.url}}/assets/images/misaligned-ai/image-1.png)

This problem is captured in the above image. In that image, a human user gives the machine a single utility function for all time. The authors argue that this <b>necessarily</b> leads to a loss of utility compared to before any optimization was done! This is true, at least, as long as there is any amount of misspecification of the true value. This could be leaving attributes out, misweighting attributes, or not formalizing the attributes in an appropriate way.

The authors introduce the problem in the principal-agent space, which comes from economics. The principal here is the designer (human) who is running the system and the agent, the machine, is acting.

They consider a resource constrained situation in which there are L attributes of a state that provide utility and we can specify J attributes, where J < L. The demonstrate that under this constraint, the principal loses utility from the agent without fixing the underlying alignment problem.

## Modeling the problem
The authors define the attribute space S in R<sup>L</sup> where L attributes define the support of the utility function. They impose a strictly increasing constraint on the utility with respect to the L dimensions. In other words, the human's utility, U, which maps R<sup>L</sup> -> a continuous value, along each L dimension can only increase and there is not an interaction in the utilities. Note: This doesn't specify whether there is interaction is how we obtain utilities, just that our utility gained from L<sub>1</sub> increasing doesn't impact any other Ls.

Given the above definition of the world, the human uses the robot to act on the world and increase their utility. The robot gets access to proxy attributes J, which are a subset of L, and a proxy utility function which maps over R<sup>J</sup>.  Further, the robot must act incrementally, meaning it works over time t to incrementally improve utility. And, finally, the robot goes through complete optimization -> the robot will reach the optimal state of proximal utility.

The human's task is to design a utility function that maximizes the true utility via the proxy utility.

As an example, the authors give content recommendation. For example, we could care about 4 attributes (L = 4), (1) amount of ad revenue, (2) engagement quality, (3) content diversity, and (4) overall community well being. The overall utility could be a sum of these, and we might model some constained environment (based on a user's limited attention): ![Misaligned AI image 2]({{site.url}}/assets/images/misaligned-ai/image-2.png)

With this constrained environment, we can easily measure ad spend and some engagement, but some other things are harder to measure, so we only give our agent 2 attributes to optimize.

In their example, they show what happens when you model any pair of attributes as well as what happens to proxy utility and real utility:
![Misaligned AI image 3]({{site.url}}/assets/images/misaligned-ai/image-3.png)

Essentially, in their toy environment, optimizing any pair of attributes decreases overall utility!

They use this toy example and extend it. They prove that overoptimization eventually occurs under two key criteria. First, they require that if you perturb the world too much in any direction, it with either be infeasible or undesirable. This seems reasonable at face value - think paper clip maximizer as an extreme example. Second, the lower bound of at least one unmentioned attribute needs to be sufficiently low. This second condition kind of proves their point though -> at least, this conditon is all that is required for their overoptimization to occur due to a feasibility constraint of not being able to increase something to it's maximum without decreasing something else.

Ultimately, it seems to come down to decreasing marginal utility at the extremes. Alternatively, this could be increasing opportunity cost, but decreasing marginal utility seems like the more likely case.

## How do we solve the problem?
![Misaligned AI image 4]({{site.url}}/assets/images/misaligned-ai/image-4.png)

In general, this image from Figure 1 of the paper proposes the dynamic loop of working with the robot. Try to improve the situation, see how it does, then add constraints or update the utility function to get closer to the real utility function. 

You could also try to ensure that the robot has no effect on the unobserved attributes from L that don't make it into J, but that actually seems more difficult - possibly only done through constraints. Some work is being done in the impact minimzation community to hopefully achieve this through minimizing impact on the state of the world. As an example, imagine you could send out surveys to Youtube watchers. Then your robot tries to keep the survey responses as constant as possible while maximizing ad revenue. This might have caught some of the negative community aspects, but (very critically) depends on the questions asked. Basically, it depends on the robot being given observations about the world that act as a proxy for "unobserved" attributes!

In the case where the human continuously updates the constraints, we're in a good place as well. Ultimately, we can get guaranteed improvements in performance if the human can watch for decreases in overall utility.

If the human can perform both the impact minimzation and utility updates, then we get the best of both worlds and we can converge to a global optimum. This version of the world requires the assumptions of both methods including the ability for the robot to minimize real impact and for the human can construct locally improving utility functions.

## Conclusion
This was a very interesting position paper. There is a lot to like here in terms of the framing. I often question how well papers like this meet the real world. The assumptions on human rationality and the fidelity of the world model we require to make any headway in real problems leaves quite a bit to be delivered. I'm not sure how much more this adds to the conceptual discussion beyond the "paper clip maximizer" parable.


