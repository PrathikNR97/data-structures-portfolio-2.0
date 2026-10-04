Data Science Portfolio Project — Two
Does Self-Report Predict Actual Social Media Use?
How accurately do people estimate their own weekly social media use, compared to objectively measured screen time? A linear regression and a decision tree, tested against real baselines, on 209 real participants.

1. Problem
2. Background
3. Data
4. Exploration
5. Preparation
6. Baseline & Models
7. Evaluation
8. Interpretation
9. Limitations
10. Code & Transparency
1. Problem Definition
Does Self-Report Predict Actual Use?
Research question: How accurately do people estimate their own weekly social media use, compared to objectively measured screen time?

This project answers that question with two models, both built around the same target variable:

Model 1 is a regression problem: predicting SMU, a participant's objectively measured weekly social media use in minutes (continuous), from their self-reported estimate.
Model 2 is a classification problem: predicting whether a participant falls into the "high" or "low" half of objective use (binary), from their self-report plus a few easily observed context variables.
Who benefits: researchers and app designers who rely on self-reported screen-time survey data (which is far cheaper to collect than device logs) need to know how much to trust it. Clinicians and educators who use self-report screeners to flag "heavy users" have the same stake. Why it matters: if self-report doesn't track actual behavior, every study, product decision, or intervention built on self-reported social media use is standing on shaky ground.

2. Background and Context
What We Already Know
The gap between what people say about their media use and what their devices actually record is a well-documented problem in behavioral research, not a one-off finding. Mahalingham (2023) ran the exact comparison this project replicates — a single self-estimate against device-logged social media minutes — on 209 participants, and found essentially no relationship between the two (r ≈ −0.04 to −0.11 depending on exact specification), concluding that single-item self-report measures of social media use cannot be trusted as a stand-in for actual behavior.

This isn't an isolated result. Parry et al. (2021), in a systematic review and meta-analysis spanning dozens of studies comparing logged and self-reported digital media use, found only a moderate average correlation (r ≈ 0.4) between the two — meaning self-report explains well under half the variance in logged use even in the most favorable published estimates, and often much less. Verbeij et al. (2021) narrowed in on adolescents specifically and likewise found self-reported estimates to be a weak and inconsistent proxy for logged social media time, varying considerably by platform and measurement window.

Taken together, these sources motivate treating self-report as a hypothesis to test, not an assumption to build on — which is exactly what this project's regression model does directly, and what the decision tree extends by asking whether adding a behavioral variable (phone pickups) alongside self-report does any better.

3. Data Description
Dataset
209 real participants (208 after cleaning), from Mahalingham, T. (2022), "Data Set — assessing the validity of self-reported social media use," Mendeley Data, V1 (CC BY 4.0) — the dataset behind Mahalingham (2023), Computers in Human Behavior, 140, 107567. data.mendeley.com/datasets/x3wxfycggn.

Unit of analysis: one row per participant. Each participant gave a single self-estimate of their weekly social media use, then had their actual social media use and phone pickups logged directly from their device (iPhone users via the built-in Screen Time function, Android users via an app-usage tracker) over a 7-day window, across Facebook, Instagram, Snapchat, Twitter, and TikTok.

Target variable: SMU — objectively measured weekly social media use, in minutes (used as a continuous target for Model 1, and binarized by median split for Model 2).

Available features: SR_SMU_minsweek (self-reported weekly minutes), PickUps (total phone unlocks over 7 days), Gender, and device type. A validated problematic-use scale (PUSNS) and per-platform breakdowns (Facebook/Instagram/Snapchat/Twitter/TikTok) also exist in the source file but are out of scope for this single-research-question version of the project.

Collection caveat: this is a single-sample, self-selected convenience sample (likely university-recruited, per the published paper), not a random sample of social media users broadly — see Limitations.

The data arrived as an SPSS .sav file. Mendeley's own domain is blocked by the network policy of the sandbox this analysis runs in, so the file was downloaded manually and uploaded, then converted to CSV locally with the open-source ReadStat library — the same library the file was originally written with — rather than any blocked network call.

4. Data Understanding and Exploration
What the Data Looks Like Before Modeling
Variable	Mean	Std	Min	Median	Max
SR_SMU_minsweek	1,261	504	90	1,200	3,900
SMU (objective)	956	750	0	827	6,334
PickUps	679	433	2	600	4,490
Outliers: using the standard IQR rule, 37 of 208 participants are outliers on SR_SMU_minsweek (mostly high self-estimates), 5 on SMU, and 3 on PickUps. These are kept rather than removed — self-reported outliers are themselves part of what the research question is about (how badly can self-report miss?), and objective outliers reflect a small number of genuinely heavy users rather than data errors.

Class balance: the decision tree's target (engagement_group) is a median split on SMU, so it is exactly 50/50 by construction (104 "high," 104 "low") — no class-imbalance correction is needed.

