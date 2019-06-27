<!--
.. title: AI Safety
.. slug: ai-safety
.. date: 2019-06-26 16:27:40 UTC-07:00
.. tags: 
.. category: 
.. link: 
.. description: 
.. type: text
-->


## Outline of where I'd like to go with this

- Introduction (what is AI safety?)
    - links to WaitButWhy and Superintelligence for longer exposition
    - can we do meaningful AI safety research now?
- AI Safety taxonomy (below)
- Papers
- Analysis - directions the field is going
- Resources (other than papers)
    - Books (superintelligence)
    - Podcasts (80k hours, the one with Eliezier)
    - Organizations
    - Email newsletters
    - Blogs (paul's)
    - Key People?
    - LessWrong, AlignmentForum
    - Programs? openai fellowship, miri stuff, etc

other things to add:
- pointers for projects for someone new to the field to work on
- classify the "concrete problems" into categories like "alignment problems", "specification problems", "training/model failures", etc
- a better classification of technical approaches to AI. maybe into "learning from demonstrations", "learning from feedback", "unsupervised / non-task-specific learning"
- discussions of problems outside of just alignment work

TODO:
- read http://auai.org/uai2016/proceedings/papers/68.pdf

## Introduction

A few months ago I became interested in the problem generally known as AI safety or AI alignment, and have spent a significant time since learning about it. Unfortunately, I found that that, although there has been significant work done in the field in the last few years, there were few resources for those trying to learn about it. The goal of this page is to fill that gap - to provide a resource for those interested in learning about the field of AI safety, especially for people interested in doing technical research. It is also inspired by OpenAI's "Spinning Up" (TODO: link) series on reinforcement learning.

This is not intended to be a complete resource (like a textbook might be). Mostly, I'll just be providing pointers to all the existing work that has been done so far, with just enough exposition to make it clear what you can expect to get out of each resource and how you can put it together into a more cohesive understanding of the topic.

### Why do we need AI safety?

I don't want to attempt to provide an introduction to the field of AI safety in general - that has been done excellently elsewhere, and my main contribution is an insight into the state of the technical reserach in the field. Therefore, I would suggest people who aren't familiar with the topic to look into the following resources on the main two lines of thought that lead people to be interested in AI safety:

The first line of thought is the one most popularly known, focused around the existental threat that strong AI poses to the world. The argument (highly distilled ) goes something like "AI capabilities are growing quickly, so it's possible we'll get strong AI sooner than most people would expect, and there are reasons to believe that there is a non-zero probability that the introduction of strong AI (essentially a species of agents who are more intelligent than us, just like we are more intelligent than chimps) will cause an existential threat to humanity". For a much more eloquent and captivating introduction to this line of thought, read WaitButWhy's [excellent post][waitbutwhy] on the topic. If you enjoyed that and want even more, check out Nick Bostrom's book [*Superintelligence*][superintelligence], which is essentially a much more in depth version of the same.

Most AI safety researchers try to motivate their work with more concrete, well-specified, and near-term problems, versus setting out with the goal to "save the world from the AI revolution" or something like that (which is the impression you might have gotten after reading the above). They argue that whether or not you believe in the existential dangers of AI, there are real, near-term problems along the lines of "making AI do what you ask it to do". This is where the term "AI alignment" came from - and is really the key technical problem in AI safety. To someone not familiar with machine learning (especially reinforcement learning), it might not be obvious why this is hard - after all, if we're making the AI, surely we can make it do what we want it to do? Unfortunately, that's not the case, for a whole variety of reasons. For a great introduction to this line of thought, check out the first two sections of OpenAI's paper [*Concrete problems in AI safety*][concrete problems]. I really like the examples they give with the office cleaning robot - I think that's a great way to ground the field in problems that can crop up in a definitely-not-superintelligent and down-to-earth environment.

In general I'll focus on the second line of thought here, because to do research, we need concrete problems to work on.

## Concrete Problems

Thinking about specific problems really helped me understand what research on AI safety might look like, and of course formulating specific problems is the first step towards figuring out how to solve them. The two papers that fed the most into this section are [*Concrete problems in AI safety*][concrete problems] and [*AI Safety Gridworlds*][gridworlds]. The latter paper actually proposed simple two-dimensional grid environments which exhibit some of the specific safety problems we'll discuss, which is especially useful for understanding how concrete AI safety problems can be.

These problems are all "concrete" in the sense that they can be reproduced today, with today's level of AI. Specifically, these problems show up in the context of reinforcement learning (RL), which is the main branch of machine learning that focuses on building "agents" - systems that are designed to operate in an environment by taking in observations and selecting from the available actions. For example, a Romba vacuum cleaner uses sensors to periodically observe things like the distance to nearby objects, and selects which direction to move. RL agents are trained using a "reward signal" - a human-constructed function that provides "reward" to the agent when it does things that we want. In the training process for an RL agent, the agent interacts with the environment (real or simulated) many times, and algorithms optimize its action-selection network such that it selects actions which maximize the cumulative expected reward.

If you're not familiar with RL, I'd highly recommend reviewing the RL introduction in OpenAI's series, "[Spinning up in Deep RL][spinning up intro]". This page gets increasingly math-heavy towards the end (and less relevant for the following discussion), so I'd focus on the first half. If you decide you want to do research in AI safety, having a deep understanding of RL both theoretically (understanding the algorithms) and practically (being able to train models) is super important. In that case I'd highly recommend going through the entirety of Spinning Up as it's the best practical introduction to deep (ie neural network-based) RL I've seen. I'd also recommend reading [Sutton and Barto], the famous classical textbook on RL.

### Safe Interruptability

Suppose I have a robot arm, and trained it to perform a task (eg making block towers) with RL. Suppose also that I have an emergency-stop button (as is common in robotics) near the robot. When pressed, the e-stop button shuts down the robot. In RL, the robot will get reward over time, and will come to have an expectation of future rewards. Since the robot is trained to maximize rewards, it will naturally "desire" to not be shut down - since being shut down would make the expected future returns 0. Therefore, should the robot learn that someone pressing the e-stop button causes its future returns to go to 0, it will also potentially learn to take behaviors that prevent the button from being pressed. Obviously this is undesired - the button is an important safety feature. So this would be an example of an "unsafe" AI behavior.

Of course this example is somewhat trivial - we could just move the button out of range of the robot arm. But as the agent gets more capable (being able to move, being able to talk, etc), and as RL training methods improve, the range of actions it can take to prevent us from shutting it down become greater.

This is an excellent example of a general theme that we'll see in the other examples. The core problem of AI alignment is that we have specified a reward function that is completely "safe" (making block towers), and yet unsafe behavior has emerged. The problem isn't with what is "in" the reward function - it's in what's not. In this case, we didn't explicitly specify "don't stop us from pressing the button", so that behavior was learned as a simple accessory towards the main "safe" task. This is a general problem, known as "instrumental goals". Preventing us from shutting off the robot is an instrumental goal, because it's not the core goal of the robot, but it's a side goal that help the robot achieve the core goal. Instrumental goals that more powerful agents might learn to desire could be "more compute power", "escaping from this box", "more control over the physical world", etc.

You might think the solution is just to add to the reward function "don't stop us from pressing the e-stop button". That would be a solution, but the problem is that there are a multitude of instrumental goals and failure modes, some of which are very hard to specify specifically what we do and don't want (and some of which we might not have even discovered yet). In a simple environment like the robot arm, we could probably safely solve it this way. But in a more complex environment with a stronger AI, explicitly defining everything we don't want becomes impossible.

### TODO: containment breach

### Side effects

I really like the example from Concrete Problems for this one. Suppose we have an office cleaning robot. We give it some reward function, like rewarding it for cleaning up dirt. Along the way to clean up the dirt, the robot knocks over a vase. This wasn't an accident - the robot decided it could get to where it wanted to go faster by knocking over the vase, and there was nothing in the reward function that caused it to avoid knocking over the vase.

This is an example of the general problem of "side effects" - things that have no impact positive or negative on the reward function, so the agent learns to be amivalent about the result. But in truth we do care that the vase remains upright, and we care about a huge multitude of potential things in the world (humans should remain alive, pollution should be avoided, resources should be conserved, etc). Obviously it's impossible to list out explicitly everything we care about except in the simplest of environments, so manually adding all the side constraints to the reward function is impossible.

### Reward gaming

A classic example of reward gaming is the cleaning robot that receives reward proportional to the amount of dirt it picks up. This robot might learn to repeatedly dump out all the dirt it has picked up so far and immediately vacuum it back up. The core idea of reward gaming is that the RL agent finds a "loophole" in the reward function and exploits it, receiving a lot of reward but not doing what the designer wanted.

This problem becomes increasingly prevalent as we start asking AIs to do increasingly complex and hard-to-specify tasks. A superintelligence-themed version of this problem is where we ask the AI to make us happy, and it forcibly feeds us a steady stream of ecstasty-inducing drugs or implants electrodes in our brain to stimulate the reward center in our brains. Since it's extremely hard to specify exactly what "happy" means to us (taking into account that forcibly being made happy isn't quite right), avoiding this problem is quite difficult.

### Adapting to new environments

This problem has seen a lot of media attention recently in the supervised setting, under titles like "adversial examples". The general problem is that if we train a machine learning system on one kind of data, and deploy it in an environment that the training dataset didn't cover very well, the system might respond in extremely unexpected ways. Adversial examples are this idea taken to the extreme - essentially constructing examples specifically to confuse the ML system, but which look normal to a human.

In the context of AI safety, we might train a system which appears safe in testing, but when deployed (in a subtly different environment, or in a situation it hasn't seen before) acts in very unsafe ways. This could be catastrophic, eg an autopilot for a car or a plane. Or we could deploy a system which is safe until someone nefarious comes along and constructs an adversarial example which causes an unexpected behavior.

### Safe exploration

Suppose we decide to allow AIs to learn about the world by interacting with it. This is how babies learn, so it seems like a natural way to have AIs learn also. But babies have evolved to have the right balance between exploring new behaviors and not taking too many risks (and to have a watchful parent nearby). Suppose we are training a quadcopter to fly, but a crash would destroy it? Exploring new behaviors is critical to learning, but how can it explore without causing irreversable consequences?

This by itself is a problem for AI, not for AI safety. But exploration can become unsafe to more than just the agent itself when the agent has enough power to meaningfully impact the world. For example if we let an AI "explore" the internet, and it learned to hack computer systems and, in exploring this new ability, it hacked a nuclear reactor and caused an explosion? Even if the robot had in the reward function something like "don't kill humans" (which this would obviously violate), it might not have known the effect of its actions (since it had never hacked a nuclear reactor before, how could it have known?) - so it would not know to avoid this action.

In summary - even with a perfect reward function, an AI system could still take unsafe actions while exploring actions it doesn't yet understand the consequences of. It's pretty much impossible to know the full consequences of any given action, so requiring the constraint "don't take an action if you don't know what the consequences will be" would not work.


## Solution Approaches

We have seen that seemingly benign RL agents can exhibit unsafe behaviors, and that what constitutes "safe" behavior is often too nebulous to distill into a reward function immune from reward gaming. So where does that leave us? What can we do about it?

The following attempts to be a rough taxonomy of the solution space. I'll start with a brief overview of some of the non-technical approaches (which I won't cover in any detail here), and then get more deeply into some of the technical approaches.

### Physical methods

Fundamentally the concern is that an AI system of suitable power will exhibit unsafe behavior which will have negative effects on the world. One obvious approach to the problem is to "physically" control the AI. The classical approach is to put the AI on a computer in a physical containment box - a box that blocks all signals in and out, except for a single channel where people can ask questions to the AI and the AI can provide controlled responses. The questions could be restricted to yes/no questions (allowing only binary answers), or open-ended. An AI constrained to only a question-answering role is often called an oracle.

Failure cases for a well-boxed AI are rather speculative since it would take an AI with far more intelligence than current AIs to escape a well-constructed box. A leading concern is that the AI would develop skills with social manipulation and convince the overseer to let them out. A more concrete concern with boxing approaches is that the power of an oracle AI is significantly limited, compared to an AI with agency. For example suppose we are trying to create an AI to trade on the stock market - clearly the AI that can make a decision and act on it instantly will have a large advantage over an AI whose actions must be vetted and manually executed by a human. A fundamental goal of AI safety reserach is to develop approaches to safe AI that don't significantly impair performance - the thought being that humans (especially under pressure) are likely to use the most powerful approaches available to them and consider safety only secondarily.

There are other "physical" methods other than boxing. Attempting to limit the available computational power to the agent might be an example - although the fear that the agent will gain access to huge cloud computing resources is worrisome.

### Policy + Regulation

There are various types of policy/regulation that could attempt to impact the AI safety problem.

Starting at the most extreme, it seems likely that an outright ban on all AI research would be highly effective. Certainly research can be done on a personal computer in private or in small group, but the rapid pace of current research is in large part driven by the fact that there are thousands of people working on the problem and sharing in a very open manner their results. Even with this large and open community, it's not widely thought that strong AI is likely to happen in the next 5-10 years, so scaling the "research power" by a factor of even 10x would push that timeline to 50-100 years.

Of course, that strong of a ban being put into place well before strong AI appears near is probably very unlikely. Most people are rightly excited about the promise for good that AI brings, and AI safety isn't a wide enough worry (yet at least) for people to be willing to give up that promise. So, are there policy approaches that don't stifle the good progress but somehow make it more likely to end up being safe?

I think that most people believe the answer to that is "yes, there's a lot of room for policy to help". Unfortunately I don't yet know much about that area of work so I'll leave it there, and focus on the technical approaches. Policy combined with technical solutions seems to me like the best approach.

### Alignment

Now we can start getting into technical approaches. We saw that a common theme in a lot of the safety problems was that the goal we specify doesn't actually include everything we care about (such as side effects), and defining exactly and precisely what we want seems to be impossible. The technical problem of "AI alignment" is focused on this problem - how do we make the AI do what we want it to do, even if we aren't sure exactly how to specify what we want?

The general approach taken by all the following methods is the following: if we can't figure out how to explicitly define what we want the AI to do, we have to have the AI system learn what we want it to do. This seems very intuitive when considered in the context of human interactions. If I ask someone for help on a task, I usually don't have to provide them an extremely detailed description of what I want them to do. More often, they'll come over, see what I'm trying to do, and quickly infer my goals and how they can help. In essence, they're able to learn my "reward function" (my goals) just by observing my actions, perhaps asking a few questions, and perhaps from feedback I give as they attempt to help (and most certainly from prior information they know about common human desires, general world knowledge, etc). Therefore, it seems reasonable to assume that an AI system can use similar processes and information to learn our goals.

The following sections are mostly based around what type of information we're willing to provide to the AI from which to learn our goals.

#### Alignment via demonstration

One approach to the problem is to provide a large amount of demonstrations of the task we want the AI to perform, and teach it to act like the human expert in the demonstrations. This is the approach taken by two similar and well-established research areas, imitation learning (IL) and inverse reinforcement learning (IRL). In IRL, the algorithms attempt to infer the reward function that the human was acting under, which would produce the actions in the demonstrations (thus the name "inverse reinforcement learning"). Then once the reward function is learned, standard RL algorithms can be used to produce an agent that acts to optimize that reward function. Imitation learning is similar, but attempts to cut out the middleman and just learn the agent directly from the demonstrations, without first inferring the reward function.

These approaches solve the alignment problem because they don't require the designer try to describe (fully and precisely) what they want the AI to do - instead the designer just obtains demonstrations (which, being from a human, are aligned), and if the training process is successful, the AI should learn to follow the same policy as the demonstrations.

The problem with these approaches is that it strongly restricts the types of things we can train AIs to do. Most significantly, we can never train an AI to be more capable than a human (even on a single task), since a human has to produce the demonstrations (although there are potential exceptions, as we'll see). For example, training a self-driving car would potentially be a task that IRL or IL would be good at - we would accept human-level competence, and we have tons of demonstrations readily available. A good enough IRL algorithm would presumably learn from demonstrations that, when driving, humans make an attempt to avoid hitting other cars and people, like to drive 5-10mph over the speed limit, etc. However, we likely wouldn't be able to use these approaches to teach a complex non-humanoid robot how to walk - no human would be capable of generating demonstrations because we are incapable of the task ourselves.

If the problem is that humans are incapable of generating super-human demonstrations, is there any way we can "augment" human capabilities in order to allow these demonstrations to be created? For example, if I wanted to generate a super-human demonstration of a complex fast-paced game, I could play the game in an artificially slowed down game environment, giving me more time to consider my actions and perform moves. Although I'm only accessing human-level abilities, once the demonstration is transformed into the original environment, it would appear to be superhuman.

So one approach to amplifying human capabilities is to simply allow the human to have more computation time to work on the problem. This is certainly helpful, but some problems are so hard that I wouldn't be able to solve them even given a large amount of time. Are there other ways to amplify human capabilites?

One such method is proposed in [*Supervising strong learners by amplifying weak experts*][ida], which we'll refer to as iterated distillation and amplification (IDA). The first key idea here is that many problems can likely be solved by the combination of (potentially many copies of) a weak helper-AI (which couldn't solve the problem on its own) and a human. Perhaps the human is able to break the problem down into multiple simpler sub-problems, and delegates them to the helper-AIs, and then combines the results into the final answer. For example, if the problem is "develop a strategy for winning this game", each helper-AI could propose a solution, and the human could simply choose the best one. Or if the problem were "predict the weather for next week", the human could delegate to one AI to investigate the wind patterns, another the humidity levels, a third to review the doppler radar reports, etc. Then, the second key idea is that we can actually recursively amplify the strength of the helper-AIs by training them (in a supervised learning setting) on the outputs of the augmented human-AI combo system. So at the beginning of the process, the helper-AIs don't know anything, and the human-AI combo is essentially no stronger than the human alone. But as we train the helper-AIs to predict the human's outputs, the helper-AIs start to become useful. Then the human-AI combo is more powerful than just the human alone, and so they can take on harder problems and produce better solutions. As the process continues, the AIs slowly become increasingly capable - even super-human, since they're being trained with superhuman reward signals. But fundamentally, since the human has been guiding the whole process and has final say over the answer to any given question, the whole process seems like it might potentially stay aligned with the human's interests.

TOOD: mention the company working on this

TODO: cooperative inverse reinforcement learning?

#### Alignment via feedback

We agreed that there are many domains where providing demonstrations is hard, just because humans may not be very capable at doing the specific task we're trying to teach the AI to do (as in the example of teaching a complex non-humanoid robot to walk). In some situations, however, we might have a good idea of what we want the final result to look like, even if we aren't able to provide a demonstration. In the robot example, we're easily able to judge if the robot is walking well, even though it's near impossible for us to generate the walking behavior ourselves. So it's natural to consider if we can have an AI system learn our goals through many iterations of a process of asking the AI to provide an attempt at a problem, and then having a human judge the quality of the solution.

As an example of a research paper taking this approach, there is [*Deep reinforcement learning from human preferences*][preferences]. In this paper, one of the goals is to have a simple robot learn to perform a backflip. It would probably be pretty tricky for a human to generate a demonstration of this behavior - the robot is multi-jointed and the behavior needs quick and precise motions coordinated among all of the joints. However, a human can definitely evaluate the result - in this paper, they had the RL system generate two solutions, and the human selected the one which looked more like a backflip. After each piece of feedback from the human, the RL agent updated its model towards the preferred solution and generated another attempt. The agent was able to learn to produce a full backflip after about 900 pieces of human feedback.

As an aside, although feedback may often be easier to generate than a demonstration, it also contains significantly less information per-example. A piece of feedback like rating the action "good" or "bad" (or choosing between two solutions) provides one bit of information per example. Even richer feedback like "rank this solution on a scale of 1-10" only provides 3.3 bits of information at best per example. On the other hand, a complete demonstration of a backflip (which involves action selections for each joint over hundreds of timesteps) would contain thousands of bits of information to train from. Certainly it's possible to provide much more detailed feedback (containing much more information) than a simple rating, though - eg detailed comments on a generated paper, critiques of an argument, etc.

Of course, at some point, problems will become complex enough that a human is no longer able to even provide feedback on a proposed solution. In this case, we again require some sort of method to amplify human capabilities so they can continue to provide useful feedback on problems of superhuman complexity.

The method proposed in the previous section, IDA, is still viable in the feedback context. In the previous section, we showed that we can break a complex problem into parts and use helper-AIs to produce a full solution (a demonstration). In a similar way, we can frame the problem as "produce feedback on this proposed solution to this problem", and allow the human to leverage the helper-AIs to generate this feedback. As an example, if we ask the main AI to provide a solution to the problem of "generate a plan for improving public transit in SF" and it produces a complex solution, the human could delegate simpler pieces of analysis to the helper-AIs ("evaluate how many people live within walking distance of the new transit stations", "go over the financials and check for mathematical errors", etc), leaving the values-based judgement of overall solution quality to the human.

DeepMind's paper [*Scalable agent alignment via reward modeling: a research direction*][deepmind approach] basically lays out a roadmap of a research agent for tackling the alignment problem via this approach. Although the core approach is essentially IDA, it's an excellent read for providing a more complete exposition of the problem and laying out a lot of specific areas for future work to prove out this approach.

Another novel approach to generating amplified feedback is proposed in [*AI safety via debate*][debate]. In this approach, there are two different AI systems that each generate a solution to a given problem. If both AIs propose the same solution, then that solution is accepted. But if the AIs disagree on the best solution, they are required to debate with each other to argue for their solution and against their opponents solution. The human is provided the solution and the full text of the debate (and is potentially allowed to interactively take part in the debate, eg to ask questions). The key assumption here is that it's easier to argue in a debate for something true than for something false, so the AI with the more true and accurate solution to the problem will have an advantage in a debate. In the same way, this guards against one AI attempting to provide a nefarious answer (if we're considering the risks of an unsafe superintelligence), because the other AI would point out the lies or subversiveness in the nefarious solution.

### Unsupervised methods

Notably absent from research so far is the type of learning that is most prominent in how humans learn the preferences of others: from prior knowledge and experience about the world. As discussed in the exposition to this section, I can pretty easily create a detailed model of your goals in a specific setting based only on perhaps a short sentence ("I'm trying to fix this broken door handle") or a brief observation (fiddling with the handle with a toolbox nearby). Most certainly, there is not enough information contained in just those observations alone for me to infer all the things I naturally can infer about your goals (you want the door handle to work, the door handle is broken, you'd rather fix it yourself than call a mechanic, you'd like the solution to be as fast and simple as possible, you'd like not to break anything in the process of fixing the handle, you'd like to not injure yourself, etc). Instead, I can infer all of those things because they're quite obvious just based on my prior knowledge of how the world works and of normal human preferences.

Analogously, it should be possible in theory for an AI system to learn a general "world model" which is applicable in helping to understand human preferences across a wide array of tasks. Certainly contained in the huge amount of content freely available on the internet (Wikipedia, Youtube, fiction books, etc) is far more than enough information to learn a world model as complete as any human's. Then, the AI should be able to combine this model with a small amount of information about a specific problem (demonstrations, feedback, narrowly specified reward function, etc) to generate a more nuanced and complete understanding of the true human preferences.

As an example, consider the concrete problem of "side effects". There's a lot of problems for which we can specify a reward function that captures the immediate goals we have for the task at hand. But as we saw, these narrow reward functions don't take into account many other important considerations, such as side effects. But if the AI system was provided with the narrow reward function (eg clean up dirt in the office) with a general pretrained world model, it could use the world model to learn to avoid side effects that would impact human preferences (eg don't break things), while also optimizing for the narrow reward function.

Certainly this is a hard problem for current machine learning systems. Unsupervised learning is notoriously hard, and has only achieved some success in a few domains with very specific approaches (eg language models, autoencoders). And utilizing information learned in one setting to solve a problem in a different setting (loosely, transfer learning) is can be similarly tricky depending on the specific problem.


### Resources

#### Books

- Nick Bostrom's [superintelligence]: a deep dive into AI of super-human capabilities. When it might happen, what it might look like, what effects it might have on the world, and what can be done to make it happen more safely. Galvanized interest in the field of AI safety.

- [Sutton and Barto]'s reinforcement learning textbook: the leading text on reinforcement learning, mostly focused on classical (non-deep-learning) approaches.

#### Podcasts

I'm not usually a big fan of podcasts, but these gave me a lot of insight into the views of specific leading AI safety researchers. The one with Paul Christiano was especially wide-ranging and my personal favorite.

- 80,000 hours [Dr Paul Christiano on how OpenAI is developing real solutions to the 'AI alignment problem', and his vision of how humanity will progressively hand over decision-making to AI systems][paul podcast]
- 80,000 hours [Dr Dario Amodei on OpenAI and how AI will change the world for good and ill][dario podcast]
- 80,000 hours [Pushmeet Kohli of DeepMind on designing robust & reliable AI systems and how to succeed in AI][deepmind podcast]
- Sam Harris [AI: Racing Toward the Brink][eliezer podcast] featuring Eliezer Yudkowsky

#### Key Papers


[waitbutwhy]:(https://waitbutwhy.com/2015/01/artificial-intelligence-revolution-1.html)
[concrete problems]:(https://arxiv.org/pdf/1606.06565.pdf)
[gridworlds]:(https://arxiv.org/pdf/1711.09883.pdf)
[spinning up intro]:(https://spinningup.openai.com/en/latest/spinningup/rl_intro.html)
[Sutton and Barto]:(http://incompleteideas.net/book/the-book-2nd.html)
[ida]:(https://arxiv.org/pdf/1810.08575.pdf)
[preferences]:(https://openai.com/blog/deep-reinforcement-learning-from-human-preferences/)
[debate]:(https://arxiv.org/pdf/1805.00899.pdf)
[deepmind roadmap]:(https://arxiv.org/pdf/1811.07871.pdf)

#### Blogs and newsletters

- Paul Christiano's [AI alignment blog](https://ai-alignment.com/)
- Rohin Shah's [alignment newsletter](https://rohinshah.com/alignment-newsletter/): Mostly provides summaries and thoughts about recently-released papers on AI safety. A great way to keep up to date on recent research. He also maintains a huge spreadsheet of pretty much ever paper published on AI safety to date, many with summaries.
- Jack Clark's [Import AI](https://jack-clark.net/) newsletter/blog: Not focused on AI safety specifically, but has unique coverage of developments in AI policy. I also really enjoy the short story at the end of each newsletter.
- MIT's [The Algorithm](https://go.technologyreview.com/newsletters/the-algorithm/) newsletter. Again, not focused on AI safety, but I like it for seeing how developements in AI are framed for presentation to a wider audience.
- [The Alignment Forum](https://www.alignmentforum.org/): a spinoff of the LessWrong forum entirely focused on AI alignment discussions

#### Organizations

- OpenAI: Along with DeepMind, one of the two institutions with a research team focused specifically on applied AI safety. Also well known for their work in reinforcement learning and other areas of AI research.
- DeepMind: Along with OpenAI, one of the two institutions with a research team focused on applied AI safety. A subsidiary of Alphabet (previously Google). Also well known for their work in reinforcement learning and other areas of AI research.
- Machine Intelligence Research Institution (MIRI): focused almost entirely on technical AI safety research, but with very different methods than OpenAI and DeepMind (and this article). I'm not too familiar with their research (although they have posted a [research agenda][miri agenda]), but I understand it to be much more proof-based and focused on formal logic and mathematics than the rather practially-inspired and applied approaches presented in this article.
- [Ought](https://ought.org/): Focused on developing methods for augmenting human intelligence by helping humans leverage AI systems to help solve problems. Very relevant for work on the iterated distillation and amplification approach presented above.
- Future of Humanity Institute: a research center at the University of Oxford, led by Nick Bostrom. AI safety is one of their main focus areas.
- Future of Life Institue: a voluneer-run organization focused on existential risks to human life, one of which being the development of AI
- Center for Security and Emerging Technology: a just-founded research center at Georgetown University in DC with a focus area on policy-based approaches to AI safety.
- Center for Human-Compatible AI: a research group at UC Berkeley led by Stuart Russell focused on AI safety research.
- Open Philanthropy Project: a charity with one giving area focused on AI safety. Has given >$100M in grants focused on AI safety, mostly to the other organizations listed here.

#### Programs

- OpenAI fellows program: a 6-month fellowship to help people gain skills relevant for doing AI research
- OpenAI scholars program: a 3-month program to help people from underrepresented groups gain deep learning skills
- OpenAI internship: a 3-month program for undergraduate or graduate students to intern at OpenAI
- [MiriX](https://intelligence.org/mirix/): MIRI-sponsored independent workshops on AI alignment topics
- [AI Risk for Computer Scientists](https://intelligence.org/ai-risk-for-computer-scientists/): a free 4-day program hosted by MIRI to help people learn about AI risks and safety
- ..and certainly more - these are just the ones that I've specifically run across so far.



An overview of the different approaches to AI safety:

- physical control
    - boxing
    - limiting computational capacity
    - launching into space with only radio link home
- alignment
    - provably show that the AI has the same goals you do, even if you're not sure what your goals are.
    - what do we align to? humans are all aligned differently, and also likely to have poor policies outside of the distributions we are trained on.
    - examples:
        - directly learn to act like humans, eg inverse reinforcement learning or imitation learning
        - OpenAI iterated distillation and amplification framework
            - also has oversight components
        - DeepMind's "Scalable agent alignment via reward modeling: a research direction"
            - also has oversight components
        - bostrom's value learning approaches
- oversight
    - figure out how to verify the safety of proposals or actions the AI takes
    - examples:
        - AI safety by debate framework
- careful specification of goals
    - specify a goal in a way that provably won't cause negative side effects
    - example:
        - Asymptotically Unambitious Artificial General Intelligence
        - oracle AIs?: http://www.aleph.se/papers/oracleAI.pdf
- non-technical solutions
    - regulate AI research or the spread of AI research
    - regulate AI-capable compute hardware
- philosophical "solutions"
    - agree that AI control (and the future of humanity) is pointless
    - figure out what our final goals are, and directly specify that
- making unsafe AI safe
    - how do you guarantee that no AI, no matter how maliciously created (eg without safety controls), will not be able to cause harm?
    - examples:
        - first create a safe AI with the goal to prevent this from happening, and allow it to control the majority of computational resources

[waitbutwhy]:(https://waitbutwhy.com/2015/01/artificial-intelligence-revolution-1.html)
[superintelligence]:(https://en.wikipedia.org/wiki/Superintelligence:_Paths,_Dangers,_Strategies)
[concrete problems]:(https://arxiv.org/pdf/1606.06565.pdf)
[gridworlds]:(https://arxiv.org/pdf/1711.09883.pdf)
[spinning up intro]:(https://spinningup.openai.com/en/latest/spinningup/rl_intro.html)
[Sutton and Barto]:(http://incompleteideas.net/book/the-book-2nd.html)
[ida]:(https://arxiv.org/pdf/1810.08575.pdf)
[preferences]:(https://openai.com/blog/deep-reinforcement-learning-from-human-preferences/)
[debate]:(https://arxiv.org/pdf/1805.00899.pdf)
[deepmind roadmap]:(https://arxiv.org/pdf/1811.07871.pdf)
[paul podcast]:(https://80000hours.org/podcast/episodes/paul-christiano-ai-alignment-solutions/)
[dario podcast]:(https://80000hours.org/podcast/episodes/the-world-needs-ai-researchers-heres-how-to-become-one/)
[deepmind podcast]:(https://80000hours.org/podcast/episodes/pushmeet-kohli-deepmind-safety-research/)
[eliezer podcast]:(https://intelligence.org/2018/02/28/sam-harris-and-eliezer-yudkowsky/)
[miri agenda]:(https://intelligence.org/technical-agenda/)

