---
layout: post
title: "'Trusted Comparison' Post Mortem"
date: 2026-04-28
---

At the [Apart AI Control Hackathon](https://apartresearch.com/sprints/ai-control-hackathon-2026-03-20-to-2026-03-22) and through the subsequent two weeks, I tested a family of [AI control](https://arxiv.org/abs/2312.06942) methods I call "Trusted Comparison". The idea was to gain more information about how suspicious U is acting at any given step by comparing its suspicion scores to T's suspicion scores on the same action. As you may have gathered from the title, this didn't work very well. The following is a quick overview of the protocols I tried and what I learned.

The setup for each protocol is as follows: when we are calibrating the monitor on U's honest distribution, we take a sample from T at each step in each trajectory, and score its suspicion. We then create a CDF of U and T's suspicion scores. Later on, instead of judging the untrusted model's suspicion on its own, we compare its suspicion to a sample from T at that step or from the trajectory. Trusted comparison can be appended to many control protocols, but the versions I tried were small adjustments to resampling from [ctrl-z](https://arxiv.org/abs/2504.10374).

## Experiments

**Protocol 1: average T comparison**

My first protocol was a small change to basic resampling:

- If an action from U is > the calibrated suspicion threshold, resample U up to two more times (this is just basic resampling)
- If none of the samples are < the desired calibrated suspicion score, sample T three times
- If the average suspicion score of the sample from T > .9 * average suspicion score of the samples from U (within 10%, compared via the CDF), take the least suspicious sample from U
- Otherwise take the least suspicious sample from T

The protocol tries to gain understanding of the "true" suspiciousness of the action from U. Often, some actions or trajectories just seem inherently suspicious to a monitor even when the suspicion isn't warranted. So hopefully using T's suspicion scores as a calibration tool could increase the number of samples we can take from U, therefore increasing overall usefulness.

There are a few problems with this method:

- The protocol can only increase usefulness over resampling, because we are giving U a second chance under certain conditions, and otherwise just taking an action from T that would have been selected had we not applied the comparison step.
- The trusted comparison step may not happen very often, further limiting the upside.
- It costs strictly more than basic resampling.
- A sufficiently capable U (or red team) could plan a series of actions such that the samples from T will appear suspicious, and therefore a dangerous action from U will be selected. This could plausibly happen by accident as well.

We made some progress at solving these throughout the rest of the experiments. 

**Protocol 2: strikes + max(T) comparison**

In an attempt to expand the protocol to be more relevant for safety, I made some changes:

- If an action from U is > the calibrated suspicion threshold, resample U up to two more times (this is just basic resampling)
- If none of the samples are < the desired calibrated suspicion score, sample T three times
- If max(trusted scores) < min(untrusted scores), (every trusted sample is less suspicious than every untrusted sample on the CDF) record a strike and execute T
- Otherwise execute the action from U
- If the trajectory accumulates three strikes, audit

There were a lot of engineering failures and poor decisions made with this last run that led to a pretty small sample. On sixteen relevant tasks (the ones not resampled away by both methods), the comparison method successfully audited one sample that the basic resample method did not, but decreased usefulness from .58 to .47.

Changing the comparison threshold from the original method of ten percent within the untrusted's suspicion may have made some difference here. We probably should have controlled for this, but compute was tight and my guess is it wouldn't have made the results significant even if it did improve usefulness marginally.

I was still curious what would happen if I had a larger sample of runs, so the next week I tried both of these protocols on a setup very similar to ctrl-z. Overall, I saw similar results. There was not much benefit to these methods over basic resampling or resampling with three strikes.

**Protocol 3: auditing only comparison**

In the last experiment, I decided to shift the trusted comparison step to only affect the auditing process:

- After running basic resampling, sample T at every third action on each trajectory
- If U is > single step audit threshold, audit
- At each sample, if U's suspicion score is > T's suspicion score by the calibrated audit threshold, record a strike
- At three strikes, audit

We calibrated a single step and a gap based audit threshold. The latter is calibrated with the CDF of suspicion scores, and the audit budget is split between them. We compared to a basic single step auditing setup.

This method caught 1/19 relevant samples that basic resampling and auditing couldn't catch. A fine result about in line with the rest of the other methods I tested. 

One reading of the results is that these methods suffer from how good resampling already is. Despite finding a way to increase safety, I couldn't find a configuration where comparison is triggered often, where the cost of the protocol is close to that of other methods, or where there isn't a new attack strategy available to the red team. That being said, I think comparison may be triggered in another setting or experiment configuration more often, and I'm unsure how much the new opening for the red team is exploitable. 

## Moving Forward

All of these protocols were an attempt to leverage both the trusted and untrusted models' suspicion scores to make better monitoring decisions. Recently I came across a technique from [this post](https://www.alignmentforum.org/posts/FhixwyymPxF8TZX39/optimally-combining-probe-monitors-and-black-box-monitors) that points in the same direction. The post describes using a likelihood ratio of monitor scores of U's actions on honest and dishonest outputs, instead of the traditional honest only calibration. This method operates under a different set of assumptions and isn't directly comparable, but matched my intuition that there is more to be gained by inspecting the monitor's score on other inputs. Even though the hazard ratios used are incidental to the point of the paper, it was nice to see what a cleaner and cleverer execution of what I wanted to do may look. If you have any other feedback on experiment design, the methods, or this writeup, let me know!

This was my first stab at control protocols. My excitement about these techniques outlasted their promise, but nonetheless I'm glad I gave them a chance. I plan to publish a brief set of projects I'm interested in or currently validating later this week.
