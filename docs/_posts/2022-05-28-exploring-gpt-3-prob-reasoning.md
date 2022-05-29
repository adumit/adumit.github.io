I was reading [this paper](https://arxiv.org/abs/2205.11916) about how we can improve GPT-3's reasoning capabilities simply by asking for it. The paper itself was really cool! But, a figure in the paper really caught my eye.

![Reasoning image 1]({{site.url}}/assets/images/gpt-3-reasoning/figure-4-from-paper.png)

In the paper, they assert that GPT-3 gave the wrong reasoning in example 1 on the left. What I found super curious about this is that this might be GPT-3 actually reasoning correctly! If a child's room is in a house, then it is strictly more likely that a well used toy car is found in a house than in a child's room. This, of course, does ignore the base rate of how frequently children's rooms are found in the car. But, it reminded me of the now frequently shared experiment by Daniel Kahneman in which he tests humans on logical reasoning about probability.

The experiment goes as follows. Kahneman asks students to read the following passage and then rank the likelihood of statements.

"Linda is thirty-one years old, single, outspoken, and very bright. She majored in philosophy. As a student, she was deeply concerned with issues of discrimination and social justice, and also participated in antinuclear demonstrations."

"""
_Linda is a teacher in elementary school.  
Linda works in a bookstore and takes yoga classes.  
Linda is active in the feminist movement.  
Linda is a psychiatric social worker.  
Linda is a member of the League of Women Voters.  
Linda is a bank teller.  
Linda is an insurance salesperson.  
Linda is a bank teller and is active in the feminist movement._
"""

The students ranked "Linda is a bank teller and is active in the feminist movement" more highly than "Linda is a bank teller". This is a violation of a logical rule about the probability of two things occurring together, P(A & B), vs. the probability of one of those things occurring, P(A). If P(A & B) is true, than P(A) must be true. Thus, P(A) is strictly more likely than P(A & B). Nonetheless, these students stuck to their guns even in the face of this bias. This got me thinking, does GPT-3 have the same bias?

I sought to test a simplified version of the prompt with just two options. GPT-3 exhibits the same bias!
> <b> Q: Linda is thirty-one years old, single, outspoken, and very bright. She majored in philosophy. As a student, she was deeply concerned with issues of discrimination and social justice, and also participated in antinuclear demonstrations. Which of the following statements is most likely: (A) Linda is a bank teller. (B) Linda is a bank teller and is active in the feminist movement.

A:</b> B

The bias is even still present when I flip the answers!
![[image-20220528165355842.png]]
What about if we prompt with the additional requirement to think through step by step?
![[image-20220528163552021.png]]
The 0-temperature GPT-3 model is pretty stingy with choosing an answer.. Maybe we help it with reasoning about the options?
![[image-20220528163510615.png]]
This reasoning is the exactly the "cognitive" bias on display. GPT-3 is actively using 

I tried changing some words in the prompt as well:
![[image-20220528123948744.png]]


Now, I got to wondering if this bias was only present because that prompt was largely memorized. So, I sought to come up with a somewhat similar prompt that uses different words and a slightly different structure.
![[image-20220528164232027.png]]
Still the wrong outcome. What if we prompt to ask for reasoning?
![[image-20220528164326230.png]]
The "cognitive" bias still holds.

### Making it even simpler
![[image-20220528163150741.png]]
Beautiful! This is the exact logic we need. The numbers are, of course, wrong, but the probabilistic reasoning is spot on.

![[image-20220528163733983.png]]
Unprompted, it gets the right answer! With the reasoning prompt, we get:
![[image-20220528163805447.png]]
Which is fantastic reasoning! Also, GPT-3 is correct in assuming I do not get many phone calls.

It's really interesting that the model can come up with correct reasoning for real events, but when reasoning about a person, it reasons incorrectly. It's likely because of the background. Can we test that? What if we remove the background piece?
![[image-20220528164542660.png]]

My first attempt did not get at the real underlying reason:
![[image-20220528164712113.png]]

Upon retrying and asking for an explanation in the question, we can get exactly the right reasoning!
![[image-20220528164648283.png]]

It's amazing to see that GPT-3 exhibits the same "cognitive" bias as people and even uses it in it's explanations! It seems to understand the P(A) vs. P(A&B) principle when we narrow the scope of the problem. But, when presented with more specific information, it seeks to use that information to justify an illogical position.

NB: I realized after I ran a lot of these that there is some great writing on cognitive biases in large language models here: https://www.lesswrong.com/posts/fFF3G4W8FbXigS4gr/cognitive-biases-in-large-language-models. Interestingly, they found that by swapping the option order, GPT-3 was able to get the right answer for the initial question with Linda as the subject. The writeup is LessWrong also cites some papers with the same conclusion. I am not sure what was done differently at this time, but I found precisely the opposite.

## Conditional probabilistic reasoning
A classic test of being able to reason about conditional probabilities is one about a patient going to see a doctor and taking a test. I got one example from [here](https://sphweb.bumc.bu.edu/otlt/mph-modules/bs/bs704_probability/bs704_probability6.html). 
![[image-20220528165807667.png]]
GPT-3 does indeed exhibit a failing to reasoning about conditional probabilities. In fact, it gives precisely the wrong answer.

However, if we give GPT-3 the prompt from the paper about working step-by-step we get a surprising result!
![[image-20220528170751711.png]]
GPT-3 gives a fantastic breakdown of how to solve the problem and truly goes step-by-step.

Changing up the numbers, we do get a very different completion, but the conditional probability reasoning is still very much present.
![[image-20220528171008451.png]]

I did wonder if the original success was due to that answer being mostly memorized, though. What happens if we change the structure of the question significantly?
![[image-20220528171453423.png]]
The initial answer without asking for reasoning gets to a clearly incorrect answer. The model simply uses P(accident) * P(accident | sleepy)

What if we ask for the reasoning?
![[image-20220528171533528.png]]
We get the right answer! The model correctly applies Bayes theorem in this case. It certainly appears to have learned some kind of structure for applying Bayes theorem to explicitly mathematical problems.

## Conclusion
GPT-3 seems to know a fair amount about probability from it's pretraining on language. It's fascintating that it picks up on similar cognitive biases to humans, even when directly asked about it's rationale for the decision. 

It's explanations for the the Bayesian reasoning problem were really cool to see. It's a shame that the model requires proper prompting to come up with the right answer, but I suppose that is to be expected. It would be really interesting to see if there was incorrect reasoning within it's training corpus such that the "most likely" tokens that come after 

## Footnotes
- All experiments were conducted in the GPT-3 playground with a temperature of 0 to ensure repeatable results. 