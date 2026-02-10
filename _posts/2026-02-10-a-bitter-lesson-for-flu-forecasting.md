---
layout: post
title: "A Bitter Lesson for Flu Forecasting"
date: 2026-02-10
---

### Introduction
I recently read this [paper](https://www.researchgate.net/publication/389346428_Foundation_time_series_models_for_forecasting_and_policy_evaluation_in_infectious_disease_epidemics) when exploring statistical models for epidemiological forecasting. It tested deep learning models for time series data on a set of historical epidemiological data in France. Soon after, I found a few more papers lauding the zero-shot capabilities of the same models, mostly on historical data from outside the US. I wondered how they would perform on recent data from the US, preferably from after they were trained so we could avoid data leakage. I decided to benchmark Amazon's Chronos 2, Prior Lab's TabPFN-TS, Google's TimeFM 2.5, Salesforce's Moirai MOE and Moirai 2.0 against this year's competitors in the CDC's [FluSight Challenge](https://www.cdc.gov/flu-forecasting/index.html)

From late November to early March every year, the CDC solicits predictions from teams across the US for targets you can read about [here](https://github.com/cdcepi/FluSight-forecast-hub/blob/main/README.md). I decided to focus on the quantile predictions of hospitalization rate due to influenza as it seems to be both the focus of the teams competing and FluSight's home page. The competition is ongoing, so we only have ten weeks of data to test on. 

Forecasters submit 23 quantiles plus a point prediction of hospitalizations for each of four horizons: h0, h1, h2, and h3, where h0 is a "nowcast" or the predicted hospitalizations for the week that just occurred, and the h1-h3 are the proceeding three weeks. The CDC updates the data often (pulled from the [NHSN Weekly Hospital Respiratory Dataset](https://data.cdc.gov/Public-Health-Surveillance/Weekly-Hospital-Respiratory-Data-HRD-Metrics-by-Ju/ua7e-t2fy/about_data)), but counts are not finalized for weeks after predictions occur. Forecasters don't get to see this "gold standard" data when they make their prediction, but are evaluated on it. They are evaluated with a Weighted Interval Score (WIS) relative to the baseline FluSight model's WIS, where a lower score is better.

### Results
You can find my code [here]().

For each model, I fed in the last four years from the NHSN dataset, and extracted forecasts of the next four weeks. The models performed very well right away, but a few simple changes improved performance greatly. 

First, I backfilled the data. Since we have the counts that were available at any given time with a timestamp of when they were available, and the "gold standard" final counts, we can take the average difference between the two and add it to the data we give the model. There are better ways to backfill data, but this naive approach decreased WIS by 10% or more, depending on the model. 

Second, I tried every combination of ensemble model with two strategies: inverse confidence weighting and taking the median of the predictions. This method is a little more 'hacky', but I thought it was worth investigating. I would need to see a lot more data, preferably over multiple seasons, to be sure any specific ensemble is actually significantly better than another ensemble or solo model. 

Overall we found Moirai-Moe to be the most effective solo model at the state level, but ensembles far outperform both it and UMass-flusion. 

#### State:
**Top Five Solo Models:**
- **moirai_moe:** 0.534
- **timesfm:** 0.554
- **moirai2:** 0.564
- **tabpfn-ts:** 0.572
- **chronos-2:** 0.587

**Top Five Ensemble Models:**
- **median(tabpfn-ts + timesfm):** 0.481
- **median(moirai_moe + tabpfn-ts):** 0.488
- **inv_conf(moirai_moe + tabpfn-ts + timesfm):** 0.492
- **inv_conf(tabpfn-ts + timesfm):** 0.494
- **inv_conf(moirai_moe + tabpfn-ts):** 0.501

UMass-flusion had an rWIS of .513, giving the ensembles a comfortable margin of victory. I am pretty excited by these scores! It seems deep learning may have real applications in this domain.

We can see the national level data has much lower error:

#### National:
**Top Five Solo Models:**
- **tabpfn-ts:** 0.296
- **moirai_moe:** 0.357
- **moirai2:** 0.395
- **timesfm:** 0.444
- **chronos-2:** 0.457

**Top Five Ensemble Models:**
- **median(moirai_moe + tabpfn-ts):** 0.274
- **inv_conf(moirai_moe + tabpfn-ts):** 0.275
- **median(tabpfn-ts + timesfm):** 0.296
- **inv_conf(tabpfn-ts + timesfm):** 0.305
- **median(moirai2 + tabpfn-ts):** 0.306

rWIS is cut almost in half when we switch over to the national level data. While this data isn't relevant because the competition asks for state level predictions, and because a state level prediction is much more actionable than a national level prediction, it's worth noting how much models improve with smoother data.

TabPFN-TS outperforms some ensembles to become the third best model overall, probably because the ensembles are just reducing variance, which again, is lower at the national level.

Below is a chart showing the predictions of the best deep learning model, deep learning ensemble model, and competition model for the state level nowcast over time:

<iframe src="/flusight-charts/state_predictions_h0.html" width="100%" height="600" frameborder="0"></iframe>

A common issue in the FluSight challenge is predicting when the largest seasonal spike will occur. While the deep learning models don't nail it, we can see Moirai gets pretty close by the time the peak hits.

Here we can see the rWIS over different horizons: 

<iframe src="/flusight-charts/state_rwis.html" width="100%" height="600" frameborder="0"></iframe>

Note that the rWIS improves at longer horizons simply because the baseline is getting worse over time. WIS still trends down over time. That said, as demonstrated by the models' relatively strong zero-shot performance, I think they have a shot at being great at long horizon predictions if we spend more time improving what we can.

### Speculation
In Richard Sutton's infamous [The Bitter Lesson](http://www.incompleteideas.net/IncIdeas/BitterLesson.html), he details how general methods that leverage computation are ultimately the most effective. Looking at this preliminary data, I'm inclined to believe this applies to epidemiological forecasting. 

By no means is this small sample proof that large, general deep learning models will outperform others in the long run, or generalize to a wide variety of locations, illnesses and data. However, I think the relative infancy of the techniques described, the swell of recent papers on this particular subject and the success of deep learning in domains like natural language is substantial evidence that epidemiological forecasting could at least be improved, if not dominated by deep learning techniques in the long run. 


### What's Next
There are a few directions I'm excited to explore to test my hypothesis and improve the predictions.

I think curating a large dataset with just epidemiological data for models to train on and test against is prudent. There are not many good evals for time series forecasting, much less epidemiological forecasting. On the same note, training a model like Patch-TST or another open foundation model could feasibly yield better results than the zero-shot models we tested here.

However, first on the docket for improving zero-shot performance is running a better backfill algorithm. I'm sure we can beat just taking the mean. Second, I'm looking to add more signal. Some models in the competition barely use the NHSN data, opting for other sources of ILI rates like the CDC's ILINet or google trends. TabPFN-TS and Chronos accept covariates, so we can try other validated signals like google trends, vaccination rate and wastewater data. We could also include other models in the competition as covariates to build an ensemble similar to the FluSight ensemble, although I have no idea how well this will work. Third is doing a fine-tune on either the data we used in context here or a larger set of curated data. TabPFN-TS shows promising returns on fine-tuning, so this is worth exploring. 

I also think we can get useful information out of the models beside their predictions, especially if performance continues to improve from the techniques above or larger models coming out. First, we can just test a ton of different covariates and see which combinations increase performance. If a covariate like vaccination rate improves performance, we can try to find a causal relationship between it and the prediction target (not a bad assumption in the case of vaccination rate). Second, we may be able to use shapley values or other mechanistic interpretability methods to measure feature importance. It will be difficult to prove causation in many cases, but I'm sure something can be learned.

The original paper above validated the idea of using these predictions for counterfactual policy scenarios. If we pack in meaningful covariates that we have control over, we can test how moving those covariates will move the overall number of hospitalizations. This method is again reliant on those covariates being causal. 

I plan to test these ideas and write another post if we get mileage out of these models for this domain. 

