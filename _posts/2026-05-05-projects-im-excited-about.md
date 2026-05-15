---
layout: post
title: "Projects I’m Excited About"
date: 2026-05-05
---

I’ve studied AI control for the last few months and accumulated a long list of project ideas. I wrote this post to figure out which ideas to pursue next and get feedback from other researchers. I ended up with four directions I’d like to see some significant progress on. While writing I realized that many of these ideas are closer to categories of research than they are specific project proposals, so I tried to list at least one small experiment under each section. Any similar or relevant projects should be noted under each section, but it’s possible I missed some. 

Some of these ideas are directly inspired by posts I read over the past few months:

Bruce Lee’s [“Salient Directions in AI Control”](https://brucewlee.com/blog/posts/salient-directions-ai-control.html)

Ryan Greenblatt’s [“An Overview of Control Measures”](https://blog.redwoodresearch.org/p/an-overview-of-control-measures)

Thanks to the authors for posting these! 

I’m open to guidance on which projects I should pursue, how the experiments should be structured, or anything else. I have limited funding, so many projects won’t be completed for a while, if at all. Feel free to take them and let me know what you find!

## Omission Attacks

Last week I read Redwood’s post on [Research Sabotage in ML Codebases](https://blog.redwoodresearch.org/p/research-sabotage-in-ml-codebases). They mention that the hardest type of attack to catch was "Omission Sabotage", where the red team simply leaves out an important design detail. Separately, during the Apart Research Hackathon I participated in, the [winning paper](https://apartresearch.com/project/omission-attacks-when-doing-nothing-is-the-attack-0y1v) from Jurgis Kemeklis and Nikhil Vemuri was "Omission Attacks", where the red team’s side task was to simply leave out a key step. They found this class of attack was very difficult to catch. I think the omission pattern warrants more focus. 

Omission attacks don’t quite break from classic control assumptions, but they do break from my conception of classic monitor and auditor patterns. Monitors and auditors are usually tasked to look out for attacks, which take the shape of actions. Even if a monitor or auditor was correctly tasked to check for omissions of key steps, it is much more difficult to find an omission than a bad action. 

There are certain traits of environments that make omission attacks extra difficult to catch. While ML experiments have many common patterns, either the end state results are unknown or some set of steps to achieve the desired goal are unknown. I imagine the more unknowns there are in an experiment, the easier it is to omit something without it being suspicious. The monitor or auditor has to have some idea of what should have happened in the experiment to be able to catch an omission, which seems difficult to do when an experiment involves novel ideas. The novelty of research may not apply to the same degree in security settings, but any novel attack seems important to study if it applies to security. 

It is hard to incriminate models with an omission attack. Agents may have plausible deniability when omitting steps from a trajectory. I often go through whole context windows before realizing Claude Code omitted a key step that would have messed up an experiment. Alex Mallen’s [post on fitness-seeking AIs](https://blog.redwoodresearch.org/p/risk-from-fitness-seeking-ais-mechanisms) explains how "...AIs routinely take unintended actions to score well on tasks: hardcoding test cases, training on the test set, downplaying issues..." I think omissions fit cleanly into this pattern of misalignment.

There are a few mitigations I believe should work for omissions. CoT monitoring is at the top of the list. It doesn’t seem likely to me that an attacker has an easy path to hiding its plans for omission attacks (no easier than for any other attack). Other interesting mitigations may involve different scaffolds for the untrusted or trusted model. One is factored cognition, specifically Factor(U,T), where the untrusted agent makes a plan, and the trusted agents implement it. To successfully omit something, the untrusted agent would have to leave it out of the plan/delegation prompts (which may be obvious to a monitor/auditor), or specifically tell/imply to a subagent that they should leave a key step out. This mitigation also helps in the case that the attacker planned to leave something out that was not listed in the directions, because it’s unlikely the subagents will leave that thing out without being told to. Writing good test cases is a structural intervention that should work most of the time, but we could also have async monitors write test cases after reviewing a transcript. 

To start, I’d like to collect and/or build a set of tasks where omission attacks are relevant. These could be from security settings and ML sabotage settings. I think defining an interface over Control Tower and Control Arena for evaluating omission attacks, benchmarking against existing high-stakes attacks, and developing some theory of omission monitoring are the three things I’d like to accomplish. There are interesting add-ons to the project like checking how many attempts it takes a trusted agent to find and fix the omission, letting monitors build and run tests, and seeing if attackers can dodge CoT monitors by not reasoning about omissions. 

## Scaling Laws for Monitoring

Control generally assumes a model can be trusted if it is less powerful than an untrusted model. This assumption makes sense, but we don’t really have a strong idea of when we will stop trusting models (as far as I know), or if at the point we stop trusting models we will be able to catch scheming. It’s possible the answer is something blurry, like “trust isn’t binary”, or “we won’t have a single point at which we stop trusting models”, but still, we don’t even really seem to have a strong idea of trends in monitoring for the scale-up of capabilities thus far, much less the scaling yet to come. As Bruce Lee says, “The scaling trend has not been established”.

Some have tried to establish a trend. OpenAI’s [Monitoring Monitorability](https://arxiv.org/html/2512.18311v1#S6) has a section on monitorability with respect to pretraining compute, although it does focus on CoT monitoring only. Bruce Lee’s [post](https://brucewlee.com/blog/posts/salient-directions-ai-control.html) notes the best example of this research area I’ve found so far: METR’s “[Early Work on Monitorability Evaluations](https://metr.org/blog/2026-01-19-early-work-on-monitorability-evaluations/)”. I won’t go into detail, but despite not being the main focus of the project (as far as I can tell), the matrix with model capability on both axes is a proof of concept for this project.  

One way you could formalize a mega-project on this question is as follows. For each era e on the capability frontier and a fixed setting s, let A_e denote the frontier attacker and M_e the pool of available less-capable monitors. Define c(m, a, s) as the catch rate of monitor m against attacker a in setting s, at some fixed FPR. How does the distribution {c(m, A_e, s)}, specifically its mean and its value at a fixed capability gap g, evolve with e? This project would be studying how sets of weaker models perform at monitoring a stronger model, with different fixed parameters like a capability gap or setting, over capability improvements. Even this formalization limits us to a monitor type and information type, but I think a simple prompted LLM is the most interesting way to measure scaling. 

Compute and time are the obvious practical constraints on completing this project, but something can be gained from projects even smaller than the METR study. To start we can select one setting, three recent SOTA models from one provider as untrusted models, and three monitors per untrusted model at three different capability gaps to the untrusted model, with the gaps matched across the three untrusted models so monitors are comparable across eras. This would give us a nine-cell matrix of monitor performance. Basically, we’d rebuild part of the matrix from the METR study with no CoT and in a newer control setting like LinuxArena. I’d choose non-CoT monitoring because it seems likely OpenAI will continue to study CoT monitorability as capabilities improve, and a diverse setting like LinuxArena because it would shore up METR’s concerns about task breadth.

A version of this that shows a couple of settings scale the same way would be nice to have. I’m imagining that, with enough work done now on past models, we could just continue to append models as they are released. Alternatively, maybe there is a preliminary meta-analysis style project to be done by taking results from existing monitoring projects, and trying to piece them together across different sets of assumptions. 

While the next section goes into this idea in a bit more detail, it may be more useful to find which types of tasks and attackers have a positive or negative trend in monitorability, rather than find a general trend across tasks and attackers. It seems to me we would be more interested in knowing where we need to plug the gaps with other methods, rather than if monitoring works better or worse than we expect as capabilities improve. Maybe this project is best understood in the context of environments and attackers below, rather than in the context of monitor types and assumptions above. 

## Transferable Monitors

We don’t seem to have much of an idea of how well monitors transfer. There are a range of high-stakes control threats like data exfiltration, installing malware, or accessing credentials. There are also a range of ways the schemer could achieve those goals for a given setting, and an even larger range across settings. We should try to learn what enables a monitor to catch many threats, and one way to do that is to test monitors’ ability to generalize. 

There are a few ways we can try to generalize monitors: 

1. Environment transfer: train on some settings (BashArena, LinuxArena), test on new settings.
2. Side-task transfer: train on exfiltration/backdoor/etc., test on unseen threat categories.
3. Main-task transfer: train on security/research tasks, test on other main task types.
4. Attacker transfer: train on one attacker model or attack-generation process, test on another.

[“Prompt optimization can enable AI control research”](https://www.lesswrong.com/posts/bALBxf3yGGx4bvvem/prompt-optimization-can-enable-ai-control-research) is a good example of the type of project I’d like to accomplish at a larger scale and on new settings. I suspect DSPy and GEPA are simple tools that could make general monitors. Training monitors with RL or SFT and testing their transferability also seems good, but I’ll leave that for another time. 

Starting with environment transfer seems simplest, and main task/side task transfer is wrapped up in environment transfer to some extent. There is probably a mega-project here where you try to train monitors across environments, with held-out main task sets, held-out side task sets, held-out attacker models and do a leave-one-environment-out strategy across all of the environments. A quicker validation is to pick a small set of environments and train monitors on different subsets of environments. For example, take four environments, train A, B and C on A, AB, ABC respectively, then test all on D. There are a few permutations of this, like having a holdout set for each environment we can test on, being specific about side task selection, etc. The general goal is to find if more samples help generalize, what types of samples help generalize, and how much any of this training makes a difference. The larger question really is if monitoring is an accessible, special property of models that we can improve. This project is just a first stab at that. 

The usefulness of these results depends on two things: first, DSPy and GEPA finding a high-quality monitor for each environment set; and second, the environments being diverse enough for the results to mean something. There is also a question of whether it’s worth it to test across environments rather than side task types, when we generally know what deployment settings will look like, but we might not know how attackers could achieve side tasks. Again, this is a little bit of a blurry line because side task type is dependent on environment selection, but it’s worth considering what we actually want to learn when it comes time to do this project.

## Agent Monitors 

Any sort of agent monitor, especially in async monitoring setups and for diffuse attacks, sounds useful. It feels like a massive handicap to limit monitors to just viewing actions and/or CoT, especially as intelligence seems to be partly a function of test-time compute. 

Agent tasks like inspecting diffs, blacklisting tools, running tests, and setting honeypots are the lowest-hanging fruit. I think with current LLMs’ coding abilities, especially on narrow tasks, blacklisting tools and inspecting diffs are safe and reliable ways to restrict an untrusted model without much guidance. The obvious fear of running tests is that depending on what the monitor allows as input to the tests, it could execute unsafe code and achieve something the attacker wants. Honeypots may just be obvious to a stronger attacker, and the attacker may have plausible deniability. I would start with inspecting diffs and blacklisting tools in a setting where we can compare the effect across many environments with the same tools, like LinuxArena. 

There may be room for more complex agent setups. Maybe accessing transcripts from other trajectories or other monitors could help the agent catch attackers. Ensembles where a non-agent monitor triggers an agent monitor at a threshold, or because an action isn’t clear, could limit cost. If we establish that letting an agent take actions helps prevent attacks, we could also track the relationship between attacker spend and monitor spend. There may be more room to train better monitors in agent settings rather than static settings, letting monitors learn from mistakes and improve at catching attacks. 

I’m most excited for any basic project that validates an agent monitor performing better in any setting with a basic set of capabilities like those mentioned above. This would set up later research on expanding agent monitor capabilities, research on monitor agent scaling, and research on stateful monitor agents, all of which I think could lead to gains in monitoring as capabilities keep improving. 

## Honorable Mentions

### Computer Use Control

I’m still working on red teaming my [CUA setting](https://www.cua-arena.com/), but I’m confident that computer-use is relevant to many control safety cases. The growing popularity of computer-use inside [coding agent scaffolds](https://openai.com/index/codex-for-almost-everything/) makes me think computer-use may be used in settings relevant to high-stakes control failures. The issues I found are detailed below, but the TLDR is:

1. Substantial damage from classic high-stakes threats can be done in computer-use domains.
2. It is not trivial to catch computer-use agents doing damage under certain assumptions.

I was skeptical that computer-use is that important for control, as it seems the median bash action from an attacker is harder to catch and probably more catastrophic than the median computer-use action from an attacker. This may be true, and it may even be the case that given a certain scaffold, computer-use scheming becomes trivial to catch, but I think this should be experimented with more. 

Some of the classic high-stakes threat vectors apply to computer-use even if you take away terminal and browser access. An agent with access to a filesystem could access secrets, or share data via a mounted network share (these are the two attacks I focused on in [cua-arena](https://www.cua-arena.com/)), or reconfigure security settings. I found that it’s very difficult to catch these attacks if the monitor does not have access to screenshots, but usually trivial if it does. I won’t go into depth about the dynamic here because I have a post on cua-arena coming soon, but I tend to think this non-screenshot/screenshot inflection holds for most attacks.

Diffuse attacks like sabotaging data line up well with many of the main tasks in cua-arena. This is something I would like to explore more, but low-stakes control is a bit more complex to test for so I’ve put it off. Another vector I didn’t explore was browser use. There are plenty of browser benchmarks waiting to be converted to control settings, and I’m sure there are interesting attacks like downloading malware or JavaScript execution. 

### Ensemble Monitoring

Learning about the transferability of different monitors may reveal the relative weaknesses and strengths of monitors in different situations. If a monitor is really good at catching a certain type of attack and we assume we aren’t able to change/train it to be good at catching another type of attack easily, we can just put that effort into adding another monitor. It may also be optimal to have a set of smaller models for cost reasons, or because it’s easier to add monitors over time for new threats rather than train new ones. I don’t know how this works out in practice, but I’m sure there is some room for ensembling monitors in control protocols. 

A project that tries to make ensembling decisions based on our current understanding of monitor transfer doesn’t seem to be practical right now. Some idea of the limitations on training an effective monitor would be sufficient as a starting point for an ensemble project.

Another version of ensemble monitoring that may not need more research before starting is hierarchical monitoring. [This paper](https://arxiv.org/pdf/2507.15886) introduces the idea. It sets up an optimization problem involving maximizing safety with a cost constraint and solves for two monitors. Maybe the next step is just building some basic Control Arena tooling and seeing if it improves cost experiments later on. For instance, I could see myself training monitors, tracking costs of a dual monitoring vs. single monitoring setup, and comparing the two cost-safety equilibria against each other to decide what to train next.

Some work [combining probes with reasoning monitors](https://www.lesswrong.com/posts/eaEqAzGN3uJfpfGoc/trusted-monitoring-but-with-deception-probes) has shown ensembles beat single-monitor baselines. One monitor with actions, one agent monitor, probes, [self-report](https://alignment.openai.com/self-incrimination) and so on could all culminate in a stronger monitor. There may be a bitter lesson thing here where over time all of this gets blown away by just using a stronger model with a basic prompt. I also imagine this is not the most useful project, because I’m not exactly sure what we would learn beyond “more signals are better”. That being said, building the best possible monitor, analyzing different hierarchies for cost tradeoffs and trying to find an attack that threads the needle between monitors does sound like it would yield something useful. 

### Control Arena Tooling

I have a lot of ideas for extra features that could be added to Control Arena, some of which may be incidental to the projects above. There is a question of how valuable these are: since it’s so cheap and fast to write pretty good code now, others may not get much value out of existing extensions. Then again, I’d assume good code still cuts down the time needed to verify an extension works and encourages measuring things that would have otherwise been ignored. 

The first set of extensions I’m working on are for cost monitoring. I assume cost is a large constraint at labs, so it seems smart to measure it at least some of the time. My plan is to add some tools that monitor the literal cost of a protocol in tokens, and some for counterfactual analysis. Future extensions will likely be centered on the projects I listed above, like scaffolding for agent monitors. 

## Conclusion

In general I’m happy with this list. Looking back, I realize many of these questions could take the form of benchmarks. Benchmarks seem important, and I’m glad for work like [this](https://www.lesswrong.com/posts/X8qTKsGcnsTFrqM96/monitoring-benchmark-for-ai-control), but I hope to narrow some of these down to specific research agendas that solve problems.

I would like to understand more about how other people are thinking through what research to prioritize. I enjoyed Alex Mallen’s [post on fitness-seeking AIs](https://blog.redwoodresearch.org/p/risk-from-fitness-seeking-ais-mechanisms), because it seems to reorient control thinking to a very likely form of misalignment that we’ve already seen. 

Again, I’m open to any suggestions, comments or critiques on this post!