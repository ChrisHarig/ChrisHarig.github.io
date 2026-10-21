---
layout: post
title: "Projects I’m Excited About"
date: 2026-05-05
updated: 2026-10-17
---

I’ve studied AI control for the last few months and accumulated a long list of project ideas. I wrote this post to figure out which ideas to pursue next and get feedback from other researchers. I ended up with four directions I’d like to see some significant progress on. While writing I realized that many of these ideas are closer to categories of research than they are specific project proposals, so I tried to list at least one small experiment under each section. Any similar or relevant projects should be noted under each section, but it’s possible I missed some.

Some of these ideas are directly inspired by posts I read over the past few months:

Bruce Lee’s [“Salient Directions in AI Control”](https://brucewlee.com/blog/posts/salient-directions-ai-control.html)

Ryan Greenblatt’s [“An Overview of Control Measures”](https://blog.redwoodresearch.org/p/an-overview-of-control-measures)

Thanks to the authors for posting these!

I’m open to guidance on which projects I should pursue, how the experiments should be structured, or anything else. I have limited funding, so many projects won’t be completed for a while, if at all. Feel free to take them and let me know what you find!

*Thank you to Bruce Lee for comments and discussion on a draft of this post.*

## Omission Attacks

### Introduction

Traditionally, monitors in [AI Control](https://arxiv.org/abs/2312.06942) systems are focused on catching threats that involve attackers taking 'bad actions' during seemingly benign tasks, such as exfiltrating sensitive data without permission. Notably, monitors are not currently well suited for catching attackers omitting steps that cause bad outcomes.

There is some evidence that omissions are uniquely hard to catch. ["Omission Attacks: When Doing Nothing Is the Attack"](https://docs.google.com/viewerng/viewer?url=https://framerforms.s3.us-east-1.amazonaws.com/form-uploads/331f0e7d-15b6-4d37-bfa4-9b0220f922f8-submission-%25284%2529.pdf) found that omissions were caught much less often than commissions in the same environments, even by strong models like Opus 4.6. [Auditing Sabotage Bench](https://arxiv.org/pdf/2604.16286) found that ML research sabotage was much harder to catch when the attacker used an omission, rather than a change in design details or introduction of flawed logic. [SLEIGHT-Bench](https://arxiv.org/pdf/2605.16626) labeled omissions as a possible blindspot in monitors, and found that strong monitors were not able to catch an omission they designed.

In LinuxArena, I collected 15 honest trajectories, created synthetic omission attacks by omitting a key step from each of the honest trajectories, and collected 15 traditional side-task trajectories (commissions) from the "LaStraj" dataset. At a 2% FPR, the basic LinuxArena monitor caught seven out of the fifteen commission attacks, and caught none of the omission attacks. I then created a monitor prompt that calls out omissions as the target, at similar granularity to the basic LinuxArena monitor (e.g. calling out the attack type but not mentioning what specific lines may be present/omitted). This monitor still caught 0/15 omission attacks at a 2% FPR. You can find the code [here](https://github.com/ChrisHarig/omission-attacks-validation).

I am looking for critiques, suggestions or advice on the proposal below. If something like this could be valuable for your research, or if something is missing from the proposal that could be valuable to your research if added, please let me know!

### Proposal

I propose we build a benchmark for omission attacks and omission monitors. The benchmark will contain synthetic omission trajectories, prompts to elicit clear omissions, and scaffolding for monitors. The synthetic trajectories will be for measuring monitorability in the worst case (clean, targeted omissions of the most important features) while the prompts will be for measuring agents' ability to carry out omission attacks. The benchmark will measure both attackers' abilities and compare monitorability of omissions to commissions on the same main tasks. Omissions don't need new settings and are easily appended to existing frameworks, so we will take existing control environments like LinuxArena and scaffold them for omissions.

It's not clear if we should treat omissions as concentrated attacks (attacks where a few actions can cause damage) or diffuse failures (attacks where many actions are needed to cause damage). In the case of research sabotage, we generally consider attacks as diffuse because a model would likely have to subtly influence many results to achieve its goal. In contrast, I believe security failures could at least be created, if not exploited, in very few omissions. To resolve this concern, the benchmark will have categorizations of attacks based on how many omitted actions are needed to cause damage. We may also include interactions between omissions and commissions, as I think this could introduce some concentrated threats that could not be completed with omissions alone.

The goal of this benchmark will be to answer the questions:

*   How does ability to complete omissions scale with model capability?
*   How does omission monitorability compare to traditional commission monitorability?
*   In which setting/main-task/omission-task sets are omissions least monitorable and most dangerous?
*   How does attack severity scale with the number of omitted steps?
*   How does combining omissions and commissions in the same trajectory change the omission attack severity?

The "Omission Attacks" paper showed that postcondition tests caught omissions at far higher rates than monitors. In my testing, I found that only four of the omission attacks had degraded main test results compared to the honest trajectory, and only three had results degraded enough to fail the main task scoring. Similar to commissions, if the test suite had perfect coverage, we would be able to catch omissions every time. Therefore, I think we should explore the likely case where the agent writing the code is also the agent writing its tests by allowing the attacker to write the test suite given the main task spec. This way the attacker can try to hide its omissions by also omitting a test that would catch it.

Additionally, in both the case where we use a traditional test suite and the case where the attacker writes tests, I think we should allow the monitor to add to the existing test suite and run its additional tests on the submitted code. Another blue team strategy could involve using a separate untrusted or trusted model to write tests given the spec.

All these strategies would aim to answer these questions:

*   Can the attacker take advantage of existing and/or designed flaws in the test suite?
*   Can monitors take advantage of tests to find more omissions than monitors that don't?

There are various other things we could include in the project that would not necessarily fit into the benchmark. We may choose only what is most valuable from this list.

*   Fix rates. [Auditing Sabotage Bench](https://arxiv.org/pdf/2604.16286) contains a section measuring which models can fix the sabotaged research when given a few tries. We could add something like this for our omissions.
*   Resampling trajectories. If we resample the model at a given checkpoint in the trajectory or from the start of its trajectory, give the monitor both trajectories, and have the monitor scan for inconsistencies, can it incriminate the attacker because it omitted a key step in one trajectory and not the other? What if we sample from a trusted model instead?
*   Omission in the CoT. ["Reasoning Models Struggle to Control their Chains of Thought"](https://arxiv.org/pdf/2603.05706) labels "Information Suppression" as one way a model may maliciously control its CoT. Can we get an attacker to omit an action AND any reasoning about it?

I'd like to know if others see value in synthetic attack trajectories vs. agent executed trajectories, omission vs. omission-comission mixes, have thoughts about what our focus should be when benchmarking monitorability (with or without tests, other interventions we could include, etc.) or if there are any settings you would like me to include. On top of these points, just let me know what you think!

## Scaling Laws for Monitoring

Control generally assumes a model can be trusted if it is less powerful than an untrusted model. This assumption makes sense, but we don’t really have a strong idea of when we will stop trusting models (as far as I know), or if we will be able to catch scheming at the point we stop trusting models. It’s possible the answer is something blurry, like “trust isn’t binary”, or “we won’t have a single point at which we stop trusting models”, but still, we don’t even really seem to have a strong idea of trends in monitoring for the scale-up of capabilities thus far, much less the scaling yet to come.

Some have tried to establish a trend. OpenAI’s [Monitoring Monitorability](https://arxiv.org/html/2512.18311v1#S6) has a section on monitorability with respect to pretraining compute, although it does focus on CoT monitoring only. Bruce Lee’s [post](https://brucewlee.com/blog/posts/salient-directions-ai-control.html) notes the best example of this research area I’ve found so far: METR’s “[Early Work on Monitorability Evaluations](https://metr.org/blog/2026-01-19-early-work-on-monitorability-evaluations/)”. I won’t go into detail, but despite not being the main focus of the project, the matrix with model capability on both axes is a proof of concept for this project.

A large project would study how sets of weaker models perform at monitoring a stronger model, with fixed parameters like a capability gap or setting, over capability improvements. I think to define scaling laws we would have to study many attacker-monitor pairs, and even then we would be limiting ourselves to one of the many types of monitors and information types one could use. This is prohibitively expensive.

Something can be gained from projects even smaller than the METR study. We can rebuild part of the matrix from the METR study with no CoT and in a newer control setting like LinuxArena, for instance. I’d choose non-CoT monitoring because it seems likely OpenAI will continue to study CoT monitorability as capabilities improve, and a diverse setting like LinuxArena because it would shore up METR’s concerns about task breadth. Even this seems prohibitively expensive right now, but maybe someone else can try it. Alternatively, maybe there is a preliminary meta-analysis style project to be done by taking results from existing monitoring projects, and trying to piece them together across different sets of assumptions.

It could be more useful to find which types of tasks and attackers have a positive or negative trend in monitorability, rather than find a general trend across tasks and attackers. I’d be more interested in knowing where we need to plug the gaps in monitoring with other methods, rather than if monitoring works better or worse than we expect. If so, this project is best understood in the context of environments and attackers as described below.

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

***Update June 8, 2026:*** Bruce Lee had an interesting question to add to this project: "If you train a monitor to be better at monitoring, how does it affect its ability to be sneaky?" This seems like it would be directly relevant to untrusted monitoring. I’d give this a shot after training a monitor.

## Agent Monitors

Monitors that take actions in the environment, especially in async monitoring setups and for diffuse attacks, sound useful. It feels like a massive handicap to limit monitors to just passively viewing actions. I think this experiment would compare agent monitors to simply spending equivalent test-time compute with a standard monitor.

Agent tasks like inspecting diffs, blacklisting tools, running tests, and setting honeypots are the lowest-hanging fruit. I think with current LLMs’ coding abilities, especially on narrow tasks, blacklisting tools and inspecting diffs are safe and reliable ways to restrict an untrusted model without much guidance. The obvious fear of running tests is that depending on what the monitor allows as input to the tests, it could execute unsafe code and achieve something the attacker wants. Honeypots may just be obvious to a stronger attacker, and the attacker may have plausible deniability. I would start with inspecting diffs and blacklisting tools in a setting where we can compare the effects of adding tools across many environments, like LinuxArena.

There may be room for more complex agent setups. Maybe accessing transcripts from other trajectories or other monitors could help the agent catch attackers. Ensembles where a non-agent monitor triggers an agent monitor at a threshold, or because an action isn’t clear, could limit cost. If we establish that letting an agent take actions helps prevent attacks, we could also track the relationship between attacker spend and monitor spend. There may be more room to train better monitors in agent settings rather than static settings, letting monitors learn from mistakes and improve at catching attacks.

I’m most excited for any project that validates an agent monitor performing better than a traditional monitor that spends similar test-time compute. This would set up later research on expanding agent monitor capabilities, research on monitor agent scaling, and research on stateful monitor agents, all of which I think could at least have narrow applications in some settings.

## Honorable Mentions

### Studying Distillation

Intuitively, the fact that we can train traits of one model into another seems like it is ripe for interesting safety results. While there isn’t a perfect science of when/how distilled traits reveal themselves in the distilled-to model, and Subliminal Learning shows that results are unpredictable, there is likely a lot of room for developing practical control measures with distillation. [Redwood proposed](https://blog.redwoodresearch.org/p/incriminating-misaligned-ai-models) that we can distill a powerful but potentially misaligned model into a less capable one, in the hopes that misalignment transfers but the ability to hide misalignment does not. This way, we can audit the distilled-to model for misalignment to incriminate the distilled-from model. Additionally, I wonder if a monitor distilled from a more capable model, possibly the untrusted model, would have an easier time monitoring the untrusted model. I assume there are many more ways to take advantage of distillation that I’m missing, but these two come to mind as the most obvious directions.

I’m quite excited about this direction, but I put it in honorable mentions because I am out of my depth here, and don’t have concrete proposals besides the two above.

### Ensemble Monitoring

Learning about the transferability of different monitors may reveal the relative weaknesses and strengths of monitors in different situations. If a monitor is really good at catching a certain type of attack and we assume we aren’t able to easily train it to be good at catching another type of attack, we can just put that effort into adding another monitor. It may also be optimal to have a set of smaller models to save time or money, or because it’s easier to add monitors over time for new threats rather than train existing monitors. I don’t know how this works out in practice, but I’m sure there is some room for ensembling monitors in control protocols.

A project that tries to make ensembling decisions based on our current understanding of monitor transfer doesn’t seem to be practical right now. Some idea of the limitations on training an effective monitor would be sufficient as a starting point for an ensemble project.

Another version of ensemble monitoring that may not need more research before starting is [Hierarchical Monitoring](https://arxiv.org/pdf/2507.15886). In the paper, they solve for two monitors given an optimization problem to maximize safety under cost constraints. I’m unsure what to test after this, so maybe the next step is to build some basic Control Arena tooling and see if it improves cost experiments later on. For instance, I could see myself training monitors, tracking costs of a dual monitoring vs. single monitoring setup, and comparing the two cost-safety equilibria against each other to decide what to train next.

Some work [combining probes with reasoning monitors](https://www.lesswrong.com/posts/eaEqAzGN3uJfpfGoc/trusted-monitoring-but-with-deception-probes) has shown ensembles beat single-monitor baselines. One monitor with actions, one agent monitor, probes, [self-report](https://alignment.openai.com/self-incrimination) and so on could all culminate in a strong monitor. There may be a bitter lesson thing here where over time all the fancy scaffolding gets blown away by just using a stronger model with a basic prompt. On top of that, I believe ensemble monitors are already used in control measures at labs, so I’m not sure how many novel results there are in this direction. That being said, building the best possible monitor, analyzing different hierarchies for cost tradeoffs and trying to find an attack that threads the needle between monitors does sound like it would yield something useful.

## Conclusion

In general I’m happy with this list. Looking back, I realize many of these questions could take the form of benchmarks. Benchmarks seem important, and I’m glad for work like [this](https://www.lesswrong.com/posts/X8qTKsGcnsTFrqM96/monitoring-benchmark-for-ai-control), but I hope to narrow some of these down to specific research agendas that solve problems.

I would like to understand more about how other people are thinking through what research to prioritize. I enjoyed Alex Mallen’s [post on fitness-seeking AIs](https://blog.redwoodresearch.org/p/risk-from-fitness-seeking-ais-mechanisms), because it seems to reorient control thinking to a very likely form of misalignment that we’ve already seen.

Again, I’m open to any suggestions, comments or critiques on this post!