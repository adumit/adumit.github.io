---
layout: post
title:  "Paper read: A Causal Approach for Business Optimization: Application on an Online Marketplace"
date:   2022-07-09 16:00:00 +0200
categories: paper-reads
---

# A Causal Approach for Business Optimization: Application on an Online Marketplace

Link: https://arxiv.org/abs/2207.01722

## Goal
Create a causal policy that estimates when it is beneficial for account executives to reach out to potential customers. Their model attempts to take into account that not all outreach is beneficial. Instead, using causal inference, they try to create a policy that determines when an account executive should reach out for maximal benefit.

## Background
In B2B sales, the sales cycle can be very complex and involve many interactions over a long time horizon. Often, teams will use a lead scoring approach and prioritize customers based on that score. However, these scores are often composed of metrics and heuristics that aren't completely contextualized. It can lead to situations where outreach can be wasted on "sure things", or where outreach has a negative effect "sleeping dogs".

A policy, pi(x) maps x -> A, where x is the input and A is the action space. They model each account executive as an agent operating in this space.
- Importantly, leads are assumed to be independent, which seems like a relatively safe assumption.
- Their goal is to find the optimal policy over sales:
	- ![Causal business understanding image 1]({{site.url}}/assets/images/causal-business-marketplace/image-20220709123913811.png)
- Given the action of outreach is binary, we can model the condition average treatment effect (CATE) as:
	- ![Causal business understanding image 2]({{site.url}}/assets/images/causal-business-marketplace/image-20220709124110700.png)
- Then, simply take action A=1 if the CATE is positive, otherwise, take no action, A=0.

## Setting
The online marketplace in question is a jewelry marketplace for used jewelry. Their system will essentially create a score for leads that have a used piece of jewelry they may want to get evaluated and then sold via Worthy.com, the site this group is working on.

Most critical to understanding the setting is the following series of steps:
1. A user will register an item on the site.
2. An AE will reach out to that user within a day.
3. The subsequent contact is then scheduled based on the predictive lead score.
4. Additionally, the lead may receive automated communication in between.

They primarily focus on the first contact decision as all follow up decisions are based on the first one.

The authors had data from 3 years: April 2018 -> May 2021. If a lead was not contacted for an entire day, that was defined as "no contact". Otherwise, it's defined as the lead was "contacted" that day. If a lead initiates contact, then that interaction was excluded from the dataset.

### Features
- Lead characteristics: state they reside in and age
- Item characteristics: value estimation among others
- Item Worthy stage: internal stage to the website indicating progress
- Communication history: all messages and phone calls prior to the decision
- Submission time/date/season

## Estimation pipeline
### Cate estimation
- They select for the 50 most relevant features for uplift.
- They excluded items with a propensity score < 0.01 or >0.99 (which excluded <1% of the data). This was to maintain positivity.
- They trained the effect estimator.
	- They trained an ensemble of 30 random forest uplift models using the CausalML implementation and then averaged over them for the final predictor.
- They evaluated their model
	- Binned predictions and computed calibration plots.
	- They created a Qini curve and computed the Qini coefficient.

### Going from model to policy
![Causal business understanding image 3]({{site.url}}/assets/images/causal-business-marketplace/image-20220709125407584.png)

- To estimate the value of the policy, they used a self normalizing importance sampling method. To get this, for each threshold value, they estimate the value of the policy.
- ![Causal business understanding image 4]({{site.url}}/assets/images/causal-business-marketplace/image-20220709125534749.png)
- This lets them compare a random policy, to their new policy, which creates a relationship much like an ROC. Further, we can see that the existing policy was quite suboptimal.

Finally, as an important business step, they construct a simple decision tree that roughly follows their recommended policy. This was important for giving to the AEs and for getting feedback from the domain experts.

## Results
- In comparison to the existing policy, their policy recommended contact ~66% of leads on the first day! This is ~3x increase compared to the current policy, which recommended only ~22%.
- Following the calculation of their policy, they ran an A/B test on their contact policy.
	- They demonstrated a 22% relative increase for the delivery rate of items, an important first step in the process. This was a statistically significant result.
	- They also measured how frequently the AEs followed the policy recommendation. They only followed it 54% of the time!
	- Further, they measured whether the delivery rate was significant only among those that were actually contacted. I.e. did they just see an increase because they increased the pool of contacts?
		- The answer, they found, was no! Even those that their policy recommended not to contact ended up with a higher rate of delivery.

## Discussion
- Pretty interesting small paper! it's nice that they published their work on what is potentially a very proprietary effort. Seems reasonable that other places could replicate this is they have a B2B sales operation and significant enough data.
- I found the result about the policy also being able to recognize when NOT to contact users was very compelling. It definitely makes a much stronger case that something significant was learned from the data.