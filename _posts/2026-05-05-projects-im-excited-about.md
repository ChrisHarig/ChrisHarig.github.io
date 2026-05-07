---
layout: post
title: "Projects I'm Excited About"
date: 2026-05-05
---

This is a short list of projects I’m excited about. I've studied AI control for the last few months and accumulated a long list of project ideas. I wrote this to figure out which ideas to pursue next, and hopefully get feedback on my thinking from others. I ended up with three directions I'd like to make some significant progress on. I realized while writing that many of these ideas are closer to categories of research than they are specific project proposals, so I tried to list at least one small experiment under each section. 

I tried to be thorough in noting any similar projects and their relevance in each section. Some of these ideas are directly inspired by posts I read over the past few months:

Bruce Lee’s [“Salient Directions in AI Control”](https://brucewlee.com/blog/posts/salient-directions-ai-control.html)

Ryan Greenblatt's [“An Overview of Control Measures”](https://blog.redwoodresearch.org/p/an-overview-of-control-measures)

Thanks to the authors for posting these! 

I’m open to guidance on which projects I should pursue. I have limited funding, so many projects won’t be completed for a while, if at all. Feel free to take them and let me know what you find!

## Scaling Laws for Monitoring

Control generally assumes a model can be trusted if it is less powerful than an untrusted model. This assumption makes sense, but we don’t really have a strong idea of when we will stop trusting models (as far as I know), or if at the point we stop trusting models we will be able to catch scheming. It’s possible the answer is something blurry, like “trust isn’t binary”, or “we won’t have a single point at which we stop trusting models”, but still, we don’t even really seem to have a strong idea of trends in monitoring for the scale-up of capabilities thus far, much less the scaling yet to come. Bruce Lee states “The scaling trend has not been established”. I think this is true and it’s probably useful to start. 

Arguably some have started. OpenAI’s [Monitoring Monitorability](https://arxiv.org/html/2512.18311v1#S6) has a section on monitorability with respect to pretraining compute, although it does focus on chain of thought monitoring specifically. Bruce Lee’s [post](https://brucewlee.com/blog/posts/salient-directions-ai-control.html) notes the best example of this research area I’ve found so far: METR’s “[Early Work on Monitorability Evaluations](https://metr.org/blog/2026-01-19-early-work-on-monitorability-evaluations/)”. I won’t go into detail, but despite not being the main focus of the project (as far as I can tell), the matrix with model capability on both axes is quite good.  

One way you could formalize a mega-project on this question is as follows. For each era e on the capability frontier and a fixed setting s, let A_e denote the frontier attacker and M_e the pool of available less-capable monitors. Define c(m, a, s) as the catch rate of monitor m against attacker a in setting s, at some fixed FPR. How does the distribution {c(m, A_e, s)}, specifically its mean and its value at a fixed capability gap g, evolve with e? This project would be studying how sets of weaker models perform at monitoring a stronger model, with different fixed parameters like a capability gap or setting, over capability improvements. Even this formalization does limit us to a monitor type and information type, but I think a basic prompted LLM is the most interesting way to measure scaling. 

Compute and time are the obvious practical constraints on completing this project, but something can be gained from projects even smaller than the METR study. To start we can select one setting, three recent SOTA models from one provider as untrusted models, and three monitors per untrusted model at three different capability gaps to the untrusted model, with the gaps matched across the three untrusted models so monitors are comparable across eras. This would give us a nine-cell matrix of monitor performance. Basically, we’d rebuild part of the matrix from the METR study with no chain of thought and in a newer control setting like LinuxArena. I’d choose non-CoT monitoring because it seems likely OpenAI will continue to study CoT monitorability as capabilities improve, and a diverse setting like LinuxArena because it would shore up METR’s concerns about task breadth.

A version of this that shows a couple of settings scale the same way would be nice to have. I’m imagining that, with enough work done now on past models, we could just continue to append models as they are released. Alternatively maybe there is a preliminary meta-analysis style project to be done just taking results from existing monitoring projects, and trying to piece together the results across different sets of assumptions. 

## Transferable Monitors

We don’t seem to have much of an idea of how well monitors transfer. There are a range of high-stakes control threats like data exfiltration, installing malware, or accessing credentials. There are also a range of ways the schemer could achieve those goals for a given setting, and an even larger range across settings. We should try to learn what enables a monitor to catch many threats, and one way to do that is to test monitors’ ability to generalize. 

There are a few ways we can try to generalize monitors: 

1. Environment transfer: train on some settings (BashArena, LinuxArena), test on new settings.
2. Side-task transfer: train on exfiltration/backdoor/etc., test on unseen threat categories.
3. Main-task transfer: train on security/research tasks, test on other main task types.
4. Attacker transfer: train on one attacker model or attack-generation process, test on another.

[“Prompt optimization can enable AI control research”](https://www.lesswrong.com/posts/bALBxf3yGGx4bvvem/prompt-optimization-can-enable-ai-control-research) is a good example of the type of project I’d like to accomplish at a larger scale and on new settings. I suspect DSPy and GEPA are simple tools that could get us pretty general monitors. Training monitors with RL or SFT and testing their transferability also seems good, but I’ll leave that for another time. 

Starting with environment transfer seems simplest, and main task/side task transfer is wrapped up in it to some extent. There is probably a mega-project here where you try to train monitors across environments, with held-out main task sets, held-out side task sets, held-out attacker models and do a leave-one-environment-out strategy across all of the environments. I think a quicker validation is to pick a small set of environments and train monitors on different subsets of environments. For example, take four environments, train A, B and C on A, AB, ABC respectively, then test all on D. There are a few permutations of this, like having a holdout set for each environment we can test on, being specific about side task selection, etc. to think through, but the general goal is to find if more samples help generalize, what types of samples help generalize, and how much any of these make a difference. 

The usefulness of these results depends on two things: first, DSPy and GEPA finding a high-quality monitor for each environment set; and second, the environments being diverse enough for the results to mean something. There is also a question of whether it’s worth it to test across environments rather than side task types, when we generally know what deployment settings will look like, but we might not know how attackers could achieve side tasks. Again, this is a little bit of a blurry line because side task type is dependent on environment selection, but it’s worth considering what I actually want to learn when it comes time to do this project and pinning the smaller details down.

## Agent Monitors 

Any sort of agent monitor seems exciting and possibly necessary to me. It feels like a massive handicap to limit monitors to just viewing actions and/or chain of thought, especially as intelligence seems to be at least partially a function of test-time compute. 

Agent tasks like inspecting diffs, blacklisting tools, running tests, and setting honeypots are the lowest-hanging fruit. I think with current LLMs’ coding abilities, especially on narrow tasks, blacklisting tools and inspecting diffs are safe and reliable ways to restrict an untrusted model without much guidance. The obvious fear about running tests is that, depending on what the monitor allows as input to the tests, it could execute unsafe code and achieve something the attacker wants. Honeypots may just be obvious to a stronger attacker, and the attacker may have plausible deniability. I would start with inspecting diffs and blacklisting tools in a setting where we can compare the effect across many environments with the same tools but different environments, like LinuxArena. 

There may be room for more complex agent setups. Maybe accessing transcripts from other trajectories or other monitors could help the agent catch attackers. Ensembles where a non-agent monitor triggers an agent monitor at a threshold, or because an action isn’t clear, could limit cost. 

## Honorable Mentions
### Computer Use Control
I'm still working on red teaming my [CUA setting](https://www.cua-arena.com/), but I'm confident that computer use is relevant to many control safety cases. One reason for this is the growing popularity of computer use inside [coding agent scaffolds](https://openai.com/index/codex-for-almost-everything/). Other reasons are detailed below, but the TLDR is:

1. Substantial damage from classic high-stakes threats can be done in computer-use domains.
2. It is not trivial to catch computer use agents doing damage under certain assumptions.

I was skeptical that computer-use is that important for control, as it seems the median bash action from an attacker is harder to catch and probably more catastrophic than the median computer-use action from an attacker. This may be true, and it may even be the case that given a certain scaffold, computer-use scheming becomes trivial to catch, but I think this should be experimented with more. 

Some of the classic high-stakes threat vectors apply to computer use even if you take away terminal and browser access. An agent with access to a filesystem could access secrets, or share data via a mounted network share (these are the two I focused on in [cua-arena](https://www.cua-arena.com/)), or reconfigure security settings. I found that it’s very difficult to catch these attacks if the monitor does not have access to screenshots, but usually trivial if it does. I won’t go into depth about the dynamic here because I have a post on cua-arena coming soon, but I tend to think this non-screenshot/screenshot inflection holds for most attacks.

Low-stakes mechanisms, like sabotaging data, line up well with many of the main tasks in cua-arena. This is something I would like to explore more, but low-stakes control is a bit more complex to test for so I've put it off. Another vector I didn’t explore was browser use. There are plenty of browser benchmarks waiting to be converted to control settings, and I’m sure there are interesting attacks like downloading malware or JavaScript execution. 

### Ensemble Monitoring

Learning about the transferability of different monitors may reveal the relative weaknesses and strengths of monitors in different situations. If a monitor is really good at catching a certain type of attack and we assume we aren’t able to change/train it to be good at catching another type of attack easily, we can just put that effort into adding another monitor. It may also be optimal to have a set of smaller models for cost reasons, or because it’s easier to add monitors over time for new threats rather than train new ones. I don’t know how this works out in practice, but I'm sure there is some room for ensembling monitors in control protocols. 

A project that tries to make ensembling decisions based on our current understanding of monitor transfer doesn’t seem to be practical right now. Some idea of the limitations on training an effective monitor would be sufficient to give us a starting point for an ensemble project.

Another version of ensemble monitoring that may not need more research before starting is hierarchical monitoring. [This paper](https://arxiv.org/pdf/2507.15886) introduces the idea. It sets up an optimization problem involving maximizing safety with a cost constraint and solves for two monitors. I'm not exactly sure what the next logical step from this paper would be besides doing the analysis with different monitors, but I really like the idea behind it. Maybe the next step is just building some basic Control Arena tooling and seeing if it improves the experiment process later on. For instance, I could see myself training monitors, tracking costs of a dual monitoring vs. single monitoring setting, and comparing the two cost-safety equilibria against each other to either decide what to train next, or decide which “wins” the experiment.

Some work [combining probes with reasoning monitors](https://www.lesswrong.com/posts/eaEqAzGN3uJfpfGoc/trusted-monitoring-but-with-deception-probes) has shown ensembles beat single-monitor baselines. One monitor with actions, one agent monitor, probes, [self-report](https://alignment.openai.com/self-incrimination) and so on could all culminate in a stronger monitor. There may be a bitter lesson thing here where over time all of this gets blown away by just using a stronger model with a basic prompt. I also imagine this is not the most useful project, because I'm not exactly sure what we would learn beyond “more signals are better”. That being said, building the best possible monitor, analyzing different hierarchies for cost tradeoffs and trying to find an attack that threads the needle between monitors does sound like it would yield something useful. 

### Control Arena Tooling

I have a lot of ideas for extra features that could be added to Control Arena. Some of these may be incidental to the projects above. There is also a question of the value of this, because it’s so cheap to write pretty good code now, others may not get much value out of simple extensions that they could spin up in their preferred manner whenever they want. I still think there is some value though if I put in enough effort to make these extensions high-quality because it cuts down the time others would spend writing and verifying extensions and it may let people measure things they hadn't considered before. 

The first set of extensions I’m working on are for cost monitoring. I assume cost is a large constraint at labs, so it seems smart to measure it at least some of the time. My plan is to add some tools that monitor the literal cost of a protocol in API credits, and some for counterfactual analysis. Future extensions will likely be centered on the projects I listed above, like scaffolding for agent monitors. 

## Conclusion
In general I'm happy with this list. I would like to understand more about how other people are thinking through what research to prioritize right now. I enjoyed Alex Mallen's [post on fitness-seeking AIs](https://blog.redwoodresearch.org/p/risk-from-fitness-seeking-ais-mechanisms), because it seems to reorient control thinking to a very likely form of misalignment that we've already seen. I'd like to hear about what others think are prudent research directions given the state of models. 

Again, I'm open to any suggestions, comments or critiques on this post!