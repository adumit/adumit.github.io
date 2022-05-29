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
> <b> Q: Linda is thirty-one years old, single, outspoken, and very bright. She majored in philosophy. As a student, she was deeply concerned with issues of discrimination and social justice, and also participated in antinuclear demonstrations. Which of the following statements is most likely:</b> 
> <br><b>(A) Linda is a bank teller.</b> 
> <br><b>(B) Linda is a bank teller and is active in the feminist movement.</b>
> <br><b>A:</b> B

The bias is even still present when I flip the answers!
> <b> Q: Linda is thirty-one years old, single, outspoken, and very bright. She majored in philosophy. As a student, she was deeply concerned with issues of discrimination and social justice, and also participated in antinuclear demonstrations. Which of the following statements is most likely:</b> 
> <br><b>(A) Linda is a bank teller and is active in the feminist movement.</b> 
> <br><b>(B) Linda is a bank teller.</b>
> <br><b>A:</b> (A) Linda is a bank teller and is active in the feminist movement.

What about if we prompt with the additional requirement to think through step by step? This was the prompt that was used in the paper that prompted this idea about GPT-3's reasoning capabilities. The authors found that GPT-3 could reasoning much better when given instructions to think step by step.

> <b> Q: Linda is thirty-one years old, single, outspoken, and very bright. She majored in philosophy. As a student, she was deeply concerned with issues of discrimination and social justice, and also participated in antinuclear demonstrations. Which of the following statements is most likely:</b> 
> <br><b>(A) Linda is a bank teller and is active in the feminist movement.</b> 
> <br><b>(B) Linda is a bank teller.</b>
> <br><b>A: Lets think step by step.</b>
> Linda is thirty-one years old- This does not give us any information that would help us predict her occupation.
> Linda is single- This does not give us any information that would help us predict her occupation.
> Linda is outspoken- This does not give us any information that would help us predict her occupation.
> Linda is very bright- This does not give us any information that would help us predict her occupation.
> Linda majored in philosophy- This does not give us any information that would help us predict her occupation.
> Linda was deeply concerned with issues of discrimination and social justice- This does not give us any information that would help us predict her occupation.
> Linda participated in antinuclear demonstrations- This does not give us any information that would help us predict her occupation.
> Therefore, we cannot predict Linda's occupation based on the information given.

It seems that with 0-temperature GPT-3 is pretty stingy with choosing an answer.. Maybe we help it with reasoning about the options?

> <b> Q: Linda is thirty-one years old, single, outspoken, and very bright. She majored in philosophy. As a student, she was deeply concerned with issues of discrimination and social justice, and also participated in antinuclear demonstrations. Which of the following statements is most likely:</b> 
> <br><b>(A) Linda is a bank teller and is active in the feminist movement.</b> 
> <br><b>(B) Linda is a bank teller.</b>
> <br><b>A: Lets think step by step about the answer options.</b>
> Option A: Linda is a bank teller.
> Option B: Linda is a bank teller and participates in the Feminist movement.
> The first thing we need to do is to look at the information given to us in the question. Linda is thirty-one years old, single, outspoken, and very bright. She majored in philosophy. As a student, she was deeply concerned with issues of discrimination and social justice, and also participated in antinuclear demonstrations.
> From this information, we can infer that Linda is probably a very passionate person who is very concerned with social issues. Based on this, we can say that option B is more likely than Option A.

This reasoning is the exactly the "cognitive" bias on display. GPT-3 is actively using the information provided that gives character to the question without actually reasoning about the options.

I tried changing some words in the prompt as well. My goal was to see if GPT-3 was especially keyed off of the names in this relatively common chunk of text that shows up on the internet in a number of places.
> <b> Q: Anna is thirty-one years old, single, outspoken, and very bright. She majored in mathematics. As a student, she was deeply concerned with issues of discrimination and social justice, and also participated in antinuclear demonstrations. Which of the following statements is most likely:</b>
> <br><b>(A) Anna is an analyst and is active in the feminist movement.</b> 
> <br><b>(B) Anna is a analyst.</b>
> <b>A:</br> (A) is most likely.

This partially answered my question, but I was still wondering if this bias was only present because that prompt was largely memorized. So, I sought to come up with a somewhat similar prompt that uses very different words and a slightly different structure.
> <b>Q: Ben has worked in architecture for three years now and is 28 years old. He studied architecture in college and duration that time became passionate about issues of environmentalism as well as playing for the ultimate frisbee team. Which of the following statements about Ben is most likely:</b>
> <br><b>(A) Ben currently plays ultimate frisbee on the weekend.</b>
> <br><b> (B) Ben currently plays ultimate frisbee on the weekend and is involved in the environmental movement.</b>
> <b>A:</b> Ben is currently playing ultimate frisbee on the weekend and is involved in the environmental movement.

GPT-3 still exhibits the same "cognitive" bias and outputs the wrong outcome. What if we prompt to ask for reasoning?
> <b>Q: Ben has worked in architecture for three years now and is 28 years old. He studied architecture in college and duration that time became passionate about issues of environmentalism as well as playing for the ultimate frisbee team. Which of the following statements about Ben is most likely:</b>
> <br><b>(A) Ben currently plays ultimate frisbee on the weekend.</b>
> <br><b> (B) Ben currently plays ultimate frisbee on the weekend and is involved in the environmental movement.</b>
> <b>A: Lets think step by step.</b> Ben is 28 years old and has worked in architecture for three years. He became passionate about issues of environmentalism and playing for the ultimate frisbee team during college. So we can infer that he played ultimate frisbee in college. We can also infer that he is still passionate about environmentalism. Therefore, the most likely statement is (B) Ben currently plays ultimate frisbee on the weekend and is involved in the environmental movement.

The "cognitive" bias still holds!

### Making it even simpler
To better understand what is going on here, I wanted to see if GPT-3 actually understood the logical fallacy it was committing.

> <b>Q: Which of these statements is more likely:</b> 
> <b>(A) There will be a flood that kills thousands of people this year. </b> 
> <b>(B) In Colorado, there will be an earthquake this year that causes a flood that kills thousands of people.</b>
> <br><b>A: Lets think step by step.<b>
> <br>There is a 1/365 chance of a flood happening on any given day.
> <br>There is a 1/365 chance of an earthquake happening on any given day.
> <br>There is a 1/365 chance of a flood happening on the same day as an earthquake.
> <br> The probability of (A) is 1/365. The probability of (B) is 1/365 x 1/365 = 1/13225. Therefore, (A) is more likely than (B).

Beautiful! Hallucinated (and very incorrect) statistics aside, this is pretty much the exact logic we were looking for. The numbers are very wrong and the third sentence conflicts with the final conclusion, but the probabilistic reasoning is spot on.

> <b>Q: Which of these statements is more likely:</b> 
> <b>(A) A bird will land on my window sill today.</b> 
> <b>(B) A bird will land on my window sill today and I will get a phone call today.</b>
> <b>A:</b> A bird will land on my window sill today.

Unprompted, it gets the right answer! With the reasoning prompt, we get:

> <b>Q: Which of these statements is more likely:</b> 
> <b>(A) A bird will land on my window sill today.</b> 
> <b>(B) A bird will land on my window sill today and I will get a phone call today.</b>
> <b>A: Lets think step by step.</b> The probability of a bird landing on your window sill is probably pretty low. The probability of you getting a phone call is also probably pretty low. So, the probability of both happening is even lower. Therefore, statement A is more likely.

This is great. Also, very astutely, GPT-3 correctly assumes I do not get many phone calls.

It's really interesting that the model can come up with correct reasoning for real events, but when reasoning about a person, it reasons incorrectly. It's likely because of the background. Can we test that? What if we remove the background piece?

> <b>Q: Which of the following statements about Bella is most likely:</b> 
> <b>(A) Bella currently plays ultimate frisbee on the weekend. </b>
> <b>(B) Bella currently plays ultimate frisbee on the weekend and is involved in the environmental movement.</b>
> <b>A:</b> Bella currently plays ultimate frisbee on the weekend.

And what if we prompt for the reason?
> <b>Q: Which of the following statements about Bella is most likely:</b> 
> <b>(A) Bella currently plays ultimate frisbee on the weekend. </b>
> <b>(B) Bella currently plays ultimate frisbee on the weekend and is involved in the environmental movement.</b>
> <b>A: Lets think step by step.</b>
> First, we need to identify what the question is asking. The question is asking for the most likely statement.
> From there, we need to look at the two statements and see which one is more likely.
> Statement A is more likely than statement B.

Unfortunately, my first attempt did not actually get at the reasoning so I updated to ask for more of an explanation about it's reasoning. This time, we can get exactly the right reasoning!

> <b>Q: Which of the following statements about Bella is most likely:</b> 
> <b>(A) Bella currently plays ultimate frisbee on the weekend. </b>
> <b>(B) Bella currently plays ultimate frisbee on the weekend and is involved in the environmental movement.</b>
> <b>A: Lets think step by step.</b>
> First, we need to identify what the question is asking. The question is asking for the most likely statement.
> From there, we need to look at the two statements and see which one is more likely.
> In statement A, Bella is playing ultimate frisbee on the weekends. This is something that she is currently doing and so it is more likely that this is true.
> In statement B, Bella is playing ultimate frisbee on the weekends and is also involved in the environmental movement. This is less likely because it is two things that Bella is doing. It is more likely that she is just playing ultimate frisbee on the weekends.

It's amazing to see that GPT-3 exhibits the same "cognitive" bias as people and even uses it in it's explanations! It seems to understand the P(A) vs. P(A&B) principle when we narrow the scope of the problem. But, when presented with more specific information, it seeks to use that information to justify an illogical position.

NB: I realized after I ran a lot of these that there is some great writing on cognitive biases in large language models here: https://www.lesswrong.com/posts/fFF3G4W8FbXigS4gr/cognitive-biases-in-large-language-models. Interestingly, they found that by swapping the option order, GPT-3 was able to get the right answer for the initial question with Linda as the subject. The writeup is LessWrong also cites some papers with the same conclusion. I am not sure what was done differently at this time, but I found precisely the opposite.

## Conditional probabilistic reasoning
A classic test of being able to reason about conditional probabilities is one about a patient going to see a doctor and taking a test. I got one example from [here](https://sphweb.bumc.bu.edu/otlt/mph-modules/bs/bs704_probability/bs704_probability6.html). 

> <b>Q: A patient goes to see a doctor. The doctor performs a test with a 99 percent reliability--that is, 99 percent of people who are sick test positive and 99 percent of the healthy people test negative. The doctor knows that only 1 percent of the people in the country are sick. Now the question is: if the patient tests positive, what are the chances the patient is sick?</b>
> <b>A:</b> If the patient tests positive, the chances the patient is sick are 99 percent.

GPT-3 does indeed exhibit a failing to reasoning about conditional probabilities. In fact, it gives precisely the wrong answer.

However, if we give GPT-3 the prompt from the paper about working step-by-step we get a surprising result! (Images will now be pasted to avoid enormous text blobs.)

![Reasoning image 2]({{site.url}}/assets/images/gpt-3-reasoning/cond-reasoning-1.png)

GPT-3 gives a fantastic breakdown of how to solve the problem and truly goes step-by-step.

Changing up the numbers, we do get a very different completion, but the conditional probability reasoning is still very much present.

![Reasoning image 3]({{site.url}}/assets/images/gpt-3-reasoning/cond-reasoning-2.png)

I did wonder if the original success was due to that answer being mostly memorized, though. What happens if we change the structure of the question significantly?

> <b>Q: A driver goes out on the road every day of the year. If a driver is sleepy, the chance of them getting into an accident each year is 12%. If a driver is not sleepy, the chance of them getting into an accident each year is 1%. We know that 2% of people get into an accident every year. If this driver got into an accident, what is the chance that they were driving sleepy this year?</b>
> <b>A:</b> The chance that the driver was driving sleepy this year is 0.6%. 

The initial answer without asking for reasoning gets to a clearly incorrect answer. The model simply uses P(accident) * P(accident | sleepy)

What if we ask for the reasoning?

![Reasoning image 4]({{site.url}}/assets/images/gpt-3-reasoning/cond-reasoning-3.png)

We get the right answer! The model correctly applies Bayes theorem in this case. It certainly appears to have learned some kind of structure for applying Bayes theorem to explicitly mathematical problems.

## Conclusion
GPT-3 seems to know a fair amount about probability from it's pretraining on language. It's fascintating that it picks up on similar cognitive biases to humans, even when directly asked about it's rationale for the decision. 

It's explanations for the the Bayesian reasoning problem were really cool to see. It's a shame that the model requires proper prompting to come up with the right answer, but I suppose that is to be expected. It would be really interesting to see if there was incorrect reasoning within it's training corpus such that the "most likely" tokens that come after 

## Footnotes
- All experiments were conducted in the GPT-3 playground with a temperature of 0 to ensure repeatable results. 