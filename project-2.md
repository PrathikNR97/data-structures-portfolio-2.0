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

Bar chart showing the decision tree's target classes are perfectly balanced
Target class balance for Model 2 — exactly even by construction, since the split point is the sample's own median.

Scatter plot of self-reported weekly social media use vs. objectively measured use
The core relationship this project tests: self-reported weekly use vs. objective use. Visually, there's no obvious upward trend — points are scattered roughly evenly regardless of self-report level, which is what motivated using a single predictor (rather than assuming extra features would help) for Model 1.

Histogram of self-report minus actual social media use
Distribution of (self-report − actual) per participant. The mean sits at +305 minutes/week — on average, participants substantially overestimated their own use, and the spread is wide, which is exactly the kind of noisy relationship a weak regression R² implies.

This exploration directly shaped feature selection: the self-report/objective-use scatter showed no visible linear trend worth adding extra regression terms to, so Model 1 stays a simple one-predictor regression (matching the research question exactly, rather than overfitting a sparse relationship with more terms). For Model 2, PickUps was added alongside self-report because it is a real behavioral signal (unlike self-report, it's logged, not estimated) and was available for every participant used in the tree.

5. Data Preparation and Feature Selection
Cleaning Decisions
Missing values: dropped rows missing any of the model variables (SR_SMU_minsweek, SMU, PickUps, Gender, Device) — 1 of 209 rows was dropped (one participant missing PickUps), leaving 208.
Duplicates: checked for duplicate Participant_ID values and found 9 IDs that appear twice (one three times) — but on inspection these are not true duplicate observations: age, gender, and self-report values differ between the repeated-ID rows in most cases, meaning the ID field was reused across what are likely different data-collection sessions rather than the same row being copied. Flagged in Limitations rather than silently dropped, since removing them on ID collision alone isn't justified by the evidence available.
Encoding: Gender and Device are one-hot encoded for the decision tree (categorical, no natural numeric order); the regression uses only the two continuous variables it needs, so no encoding is required there.
Scaling: not applied — a single-predictor linear regression doesn't need it, and scikit-learn's decision tree splits on raw thresholds regardless of feature scale, so scaling would have had no effect on either model.
Outliers: identified via IQR (see Exploration) but kept, not removed — see rationale above.
Train/test split: the decision tree uses a 75/25 train/test split, stratified on the target class so both splits keep the same 50/50 balance. The regression is reported on the full sample (n=208) since its purpose here is descriptive — quantifying the actual strength of the self-report/objective-use relationship in this dataset — rather than building a model meant to generalize to new, unseen respondents.
Data leakage: the tree's target (engagement_group) is derived entirely from SMU, so SMU itself is excluded from its feature set — using it as both source-of-target and a predictor would leak the answer directly into the model. The split is also stratified and computed only on the training fold's label distribution in spirit (median computed on the full sample, a minor simplification disclosed here rather than hidden).
6. Baseline and Model Development
What "Beating Chance" Means Here
Regression baseline: always predicting the sample mean of SMU (956 minutes) for every participant. By definition this baseline's R² is 0.000 — it's the reference point any real predictor has to beat.

Classification baseline: always predicting the majority class. Since the classes are exactly 50/50, this baseline's accuracy is 0.500 — equivalent to a coin flip.

Model 1 — Linear Regression
Predicts SMU from SR_SMU_minsweek alone. Chosen because the research question is specifically about the strength of this one relationship, and linear regression directly estimates its direction and magnitude without assuming anything more complex.

Model 2 — Decision Tree Classifier
Predicts engagement_group (high/low objective use) from SR_SMU_minsweek, PickUps, Gender, and Device together. Chosen because it can capture threshold effects a linear model can't — e.g., self-report might only matter above a certain pickup count — and because it lets a second, logged behavioral variable (pickups) compete directly against self-report for predictive credit. Hyperparameters (max_depth=4, min_samples_leaf=10) were chosen to keep the tree shallow enough to read and interpret, and to avoid overfitting a sample of only 208 rows; both models use the same fixed random_state so the comparison is reproducible.

7. Model Evaluation and Selection
Results
Model 1 (regression): r = −0.11, R² = 0.013 (n = 208), vs. a baseline R² of 0.000.

R² is the appropriate metric here because it directly answers "what share of the variance in actual use does self-report explain?" The answer: about 1.3% — barely above the baseline, and the relationship runs slightly negative. This matches the published study's own finding of essentially no relationship.

Model 2 (decision tree): test accuracy = 0.673 (35 of 52 held-out participants correct), vs. a baseline accuracy of 0.500.

Accuracy is appropriate here because the classes are exactly balanced (50/50), so it isn't distorted by class imbalance the way it could be in a skewed dataset. The tree clears the baseline by 17.3 percentage points — a real, meaningful improvement over chance, even though it's far from perfect.

Bar chart comparing decision tree accuracy to the majority-class baseline
Which model performed better, and which is "final"? These two models answer related but different versions of the research question, so "final model" here means: which approach better supports an actual decision. The regression says self-report explains almost nothing on its own. The decision tree, once given a second, logged variable (pickups) to work with, does meaningfully better than chance — suggesting the right fix for weak self-report isn't a fancier model of self-report, it's supplementing or replacing self-report with even one piece of logged behavioral data. If forced to pick one model to act on, the decision tree is the one worth deploying; the regression is the one that correctly diagnoses why self-report alone isn't enough.

8. Model Interpretation and Insights
What the Models Learned
Decision tree diagram predicting high vs. low objective social media engagement
The fitted decision tree (depth 4). The root split is PickUps (≤ 513/week), not SR_SMU_minsweek.

Bar chart of feature importances from the decision tree
PickUps accounts for roughly 82% of the tree's total splitting power; SR_SMU_minsweek accounts for the rest. Gender and Device contributed nothing.

Every split in the tree is on PickUps or SR_SMU_minsweek — never on gender or device type, which have zero feature importance. That means the tree found no evidence that who someone is (by these two demographic categories) changes how well self-report or pickups predict their actual use; the behavioral signal does all the work.

The confusion matrix (14 true negatives, 12 false positives, 5 false negatives, 21 true positives) shows the tree is noticeably better at catching true high-users (21 of 26, ~81%) than true low-users (14 of 26, ~54%) — it over-predicts "high" more often than "low." In practice this means the model is more trustworthy when it says someone is a heavy user than when it says they aren't.

What can be concluded: self-report alone is a weak signal for actual use (Model 1), but a logged behavioral proxy like phone pickups adds real, measurable predictive power (Model 2) — even though neither model comes close to being a precise instrument. What can't be concluded: that self-report is worthless in every context, that pickups causally drive social media use, or that these findings generalize beyond this specific convenience sample.

9. Limitations, Ethics, and Reflection
Where This Could Go Wrong
Sample bias. N = 208 from one convenience sample (likely university-recruited) — not representative of social media users broadly, across age, region, or platform-usage culture.
Possible ID collisions. 9 participant IDs recur with differing demographic/self-report values (see Data Preparation) — if these are genuinely the same individuals measured twice rather than an ID-reuse artifact, it would modestly inflate the effective sample size and violate the independence assumption behind both models' significance.
Unlabeled device-type coding. The file carries no embedded value label for which numeric code is Android vs. iPhone, so this project cannot say whether device type matters — it can only say the two unlabeled groups didn't differ.
Who is affected by a wrong prediction: if a tool like this were used to flag "heavy users" for an intervention (e.g., a digital-wellness nudge or a clinical screener), a false positive (14 real low-users predicted high, in the test-set proportions here) means someone is wrongly flagged or nudged despite normal use; a false negative means a genuinely heavy user is missed and gets no support. Given the tree's own asymmetry (better at catching true highs than true lows), false positives are the more likely error in practice.
Real-world appropriateness: given 67.3% accuracy and a tiny regression R², neither model is precise enough to make high-stakes individual decisions (e.g., clinical diagnosis, parental control thresholds) — at most they support population-level research conclusions ("self-report under-performs logged behavior as a predictor"), not individual-level judgments.
Next steps: a larger, more diverse sample; genuine behavioral features beyond pickups (e.g., session count, time-of-day patterns); and resolving the device-label and ID-collision ambiguities directly with the dataset's author would all meaningfully strengthen this analysis.
10. Code and Transparency
Code, Data, and AI Disclosure
Code: generate_graphs.py (exploratory analysis, baselines, modeling, and visualization) — link to your GitHub repository here.

Dataset: Mahalingham, T. (2022). "Data Set — assessing the validity of self-reported social media use." Mendeley Data, V1 (CC BY 4.0). https://data.mendeley.com/datasets/x3wxfycggn.

Edit this to match your course's exact AI-disclosure policy — e.g., "Claude Sonnet 5 (Anthropic) was used to help locate the real published dataset matching this project's research question, convert it from SPSS format when standard tools were unavailable, write the exploratory-analysis, baseline, and modeling code, and draft this write-up. All research questions, modeling decisions, interpretation, and final editorial decisions are the author's own."

Built with Python (pandas, scikit-learn, matplotlib) for the analysis and plain HTML/CSS for the site. Data: Mahalingham (2022), a real, published, CC-BY-licensed sample of 209 participants.
