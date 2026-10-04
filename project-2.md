<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Does Self-Report Predict Actual Social Media Use?</title>
<style>
  :root {
    --ink: #1F1F1F; --paper: #F7F5F0; --panel: #FFFFFF;
    --teal: #1B4B5A; --clay: #C97B3E; --rule: #DAD5C9;
  }
  * { box-sizing: border-box; }
  body { margin: 0; font-family: -apple-system, "Segoe UI", Helvetica, Arial, sans-serif;
         background: var(--paper); color: var(--ink); line-height: 1.6; }
  header { padding: 4rem 1.5rem 2.5rem; max-width: 820px; margin: 0 auto; border-bottom: 1px solid var(--rule); }
  .eyebrow { color: var(--teal); font-weight: 600; margin-bottom: 0.5rem; }
  h1 { font-family: Georgia, serif; font-size: 2.1rem; line-height: 1.15; margin: 0 0 0.75rem; }
  header p.tagline { font-size: 1.1rem; max-width: 62ch; color: #454545; margin: 0; }
  nav { margin-top: 1.5rem; display: flex; gap: 1.1rem; flex-wrap: wrap; font-size: 0.9rem; }
  nav a { color: var(--teal); text-decoration: none; font-weight: 600; }
  main { max-width: 820px; margin: 0 auto; padding: 0 1.5rem; }
  section { padding: 2.75rem 0; border-bottom: 1px solid var(--rule); }
  section:last-of-type { border-bottom: none; }
  .section-num { color: var(--clay); font-weight: 700; font-size: 0.85rem; letter-spacing: 0.04em; text-transform: uppercase; }
  h2 { font-family: Georgia, serif; font-size: 1.5rem; margin: 0.25rem 0 1rem; }
  h3 { font-size: 1.1rem; margin: 1.5rem 0 0.5rem; }
  p { margin: 0 0 1rem; }
  .card { background: var(--panel); border: 1px solid var(--rule); border-radius: 8px; padding: 1.5rem; margin-top: 1.25rem; }
  .note {
    background: #FBF0DF; border: 1px dashed var(--clay); color: #7A4A17;
    padding: 1rem 1.25rem; border-radius: 6px; font-size: 0.95rem; margin-top: 1rem;
  }
  code { background: #eee7d9; padding: 0.1rem 0.35rem; border-radius: 4px; font-size: 0.9em; }
  ul.plain { padding-left: 1.2rem; }
  ul.plain li { margin-bottom: 0.45rem; }
  .fig img { width: 100%; height: auto; border: 1px solid var(--rule); border-radius: 6px; background: white; }
  table.codebook { width: 100%; border-collapse: collapse; margin-top: 1rem; font-size: 0.92rem; }
  table.codebook th, table.codebook td { text-align: left; padding: 0.5rem 0.6rem; border-bottom: 1px solid var(--rule); }
  table.codebook th { color: var(--teal); font-size: 0.85rem; text-transform: uppercase; }
  .refs { list-style: none; padding-left: 0; }
  .refs li { margin-bottom: 1rem; text-indent: -1.5em; padding-left: 1.5em; font-size: 0.96rem; }
  footer { max-width: 820px; margin: 0 auto; padding: 2.5rem 1.5rem 4rem; color: #7a7a7a; font-size: 0.9rem; }
</style>
</head>
<body>

<header>
  <div class="eyebrow">Data Science Portfolio Project — Two</div>
  <h1>Does Self-Report Predict Actual Social Media Use?</h1>
  <p class="tagline">How accurately do people estimate their own weekly social media use, compared to objectively measured screen time? A linear regression and a decision tree, tested against real baselines, on 209 real participants.</p>
  <nav>
    <a href="#problem">1. Problem</a>
    <a href="#background">2. Background</a>
    <a href="#data">3. Data</a>
    <a href="#exploration">4. Exploration</a>
    <a href="#preparation">5. Preparation</a>
    <a href="#baseline">6. Baseline &amp; Models</a>
    <a href="#evaluation">7. Evaluation</a>
    <a href="#interpretation">8. Interpretation</a>
    <a href="#limitations">9. Limitations</a>
    <a href="#code">10. Code &amp; Transparency</a>
  </nav>
</header>

<main>

  <section id="problem">
    <div class="section-num">1. Problem Definition</div>
    <h2>Does Self-Report Predict Actual Use?</h2>
    <p><strong>Research question:</strong> How accurately do people estimate their own weekly social media use, compared to objectively measured screen time?</p>
    <p>This project answers that question with two models, both built around the same target variable:</p>
    <ul class="plain">
      <li><strong>Model 1</strong> is a <strong>regression</strong> problem: predicting <code>SMU</code>, a participant's objectively measured weekly social media use in minutes (continuous), from their self-reported estimate.</li>
      <li><strong>Model 2</strong> is a <strong>classification</strong> problem: predicting whether a participant falls into the "high" or "low" half of objective use (binary), from their self-report plus a few easily observed context variables.</li>
    </ul>
    <p><strong>Who benefits:</strong> researchers and app designers who rely on self-reported screen-time survey data (which is far cheaper to collect than device logs) need to know how much to trust it. Clinicians and educators who use self-report screeners to flag "heavy users" have the same stake. <strong>Why it matters:</strong> if self-report doesn't track actual behavior, every study, product decision, or intervention built on self-reported social media use is standing on shaky ground.</p>
  </section>

  <section id="background">
    <div class="section-num">2. Background and Context</div>
    <h2>What We Already Know</h2>
    <p>The gap between what people say about their media use and what their devices actually record is a well-documented problem in behavioral research, not a one-off finding. Mahalingham (2023) ran the exact comparison this project replicates — a single self-estimate against device-logged social media minutes — on 209 participants, and found essentially no relationship between the two (r ≈ −0.04 to −0.11 depending on exact specification), concluding that single-item self-report measures of social media use cannot be trusted as a stand-in for actual behavior.</p>
    <p>This isn't an isolated result. Parry et al. (2021), in a systematic review and meta-analysis spanning dozens of studies comparing logged and self-reported digital media use, found only a moderate average correlation (r ≈ 0.4) between the two — meaning self-report explains well under half the variance in logged use even in the most favorable published estimates, and often much less. Verbeij et al. (2021) narrowed in on adolescents specifically and likewise found self-reported estimates to be a weak and inconsistent proxy for logged social media time, varying considerably by platform and measurement window.</p>
    <p>Taken together, these sources motivate treating self-report as a hypothesis to test, not an assumption to build on — which is exactly what this project's regression model does directly, and what the decision tree extends by asking whether adding a behavioral variable (phone pickups) alongside self-report does any better.</p>
  </section>

  <section id="data">
    <div class="section-num">3. Data Description</div>
    <h2>Dataset</h2>
    <div class="card">
      <p><strong>209 real participants</strong> (208 after cleaning), from Mahalingham, T. (2022), <em>"Data Set — assessing the validity of self-reported social media use,"</em> Mendeley Data, V1 (CC BY 4.0) — the dataset behind Mahalingham (2023), <em>Computers in Human Behavior</em>, 140, 107567. <a href="https://data.mendeley.com/datasets/x3wxfycggn" target="_blank" rel="noopener">data.mendeley.com/datasets/x3wxfycggn</a>.</p>
      <p><strong>Unit of analysis:</strong> one row per participant. Each participant gave a single self-estimate of their weekly social media use, then had their actual social media use and phone pickups logged directly from their device (iPhone users via the built-in Screen Time function, Android users via an app-usage tracker) over a 7-day window, across Facebook, Instagram, Snapchat, Twitter, and TikTok.</p>
      <p><strong>Target variable:</strong> <code>SMU</code> — objectively measured weekly social media use, in minutes (used as a continuous target for Model 1, and binarized by median split for Model 2).</p>
      <p><strong>Available features:</strong> <code>SR_SMU_minsweek</code> (self-reported weekly minutes), <code>PickUps</code> (total phone unlocks over 7 days), <code>Gender</code>, and device type. A validated problematic-use scale (PUSNS) and per-platform breakdowns (Facebook/Instagram/Snapchat/Twitter/TikTok) also exist in the source file but are out of scope for this single-research-question version of the project.</p>
      <p><strong>Collection caveat:</strong> this is a single-sample, self-selected convenience sample (likely university-recruited, per the published paper), not a random sample of social media users broadly — see Limitations.</p>
      <p>The data arrived as an SPSS <code>.sav</code> file. Mendeley's own domain is blocked by the network policy of the sandbox this analysis runs in, so the file was downloaded manually and uploaded, then converted to CSV locally with the open-source <a href="https://github.com/WizardMac/ReadStat" target="_blank" rel="noopener">ReadStat</a> library — the same library the file was originally written with — rather than any blocked network call.</p>
    </div>
  </section>

  <section id="exploration">
    <div class="section-num">4. Data Understanding and Exploration</div>
    <h2>What the Data Looks Like Before Modeling</h2>
    <table class="codebook">
      <tr><th>Variable</th><th>Mean</th><th>Std</th><th>Min</th><th>Median</th><th>Max</th></tr>
      <tr><td><code>SR_SMU_minsweek</code></td><td>1,261</td><td>504</td><td>90</td><td>1,200</td><td>3,900</td></tr>
      <tr><td><code>SMU</code> (objective)</td><td>956</td><td>750</td><td>0</td><td>827</td><td>6,334</td></tr>
      <tr><td><code>PickUps</code></td><td>679</td><td>433</td><td>2</td><td>600</td><td>4,490</td></tr>
    </table>
    <div class="card">
      <p><strong>Outliers:</strong> using the standard IQR rule, 37 of 208 participants are outliers on <code>SR_SMU_minsweek</code> (mostly high self-estimates), 5 on <code>SMU</code>, and 3 on <code>PickUps</code>. These are kept rather than removed — self-reported outliers are themselves part of what the research question is about (how badly can self-report miss?), and objective outliers reflect a small number of genuinely heavy users rather than data errors.</p>
      <p><strong>Class balance:</strong> the decision tree's target (<code>engagement_group</code>) is a median split on <code>SMU</code>, so it is exactly 50/50 by construction (104 "high," 104 "low") — no class-imbalance correction is needed.</p>
    </div>
    <div class="fig" style="margin-top:1.25rem">
      <img src="assets/bar_class_balance.png" alt="Bar chart showing the decision tree's target classes are perfectly balanced">
      <p style="font-size:0.9rem;color:#666;margin-top:0.5rem">Target class balance for Model 2 — exactly even by construction, since the split point is the sample's own median.</p>
    </div>
    <div class="fig" style="margin-top:1.5rem">
      <img src="assets/scatter_selfreport_vs_smu.png" alt="Scatter plot of self-reported weekly social media use vs. objectively measured use">
      <p style="font-size:0.9rem;color:#666;margin-top:0.5rem">The core relationship this project tests: self-reported weekly use vs. objective use. Visually, there's no obvious upward trend — points are scattered roughly evenly regardless of self-report level, which is what motivated using a single predictor (rather than assuming extra features would help) for Model 1.</p>
    </div>
    <div class="fig" style="margin-top:1.5rem">
      <img src="assets/hist_over_underestimation.png" alt="Histogram of self-report minus actual social media use">
      <p style="font-size:0.9rem;color:#666;margin-top:0.5rem">Distribution of (self-report − actual) per participant. The mean sits at +305 minutes/week — on average, participants substantially <em>overestimated</em> their own use, and the spread is wide, which is exactly the kind of noisy relationship a weak regression R² implies.</p>
    </div>
    <p>This exploration directly shaped feature selection: the self-report/objective-use scatter showed no visible linear trend worth adding extra regression terms to, so Model 1 stays a simple one-predictor regression (matching the research question exactly, rather than overfitting a sparse relationship with more terms). For Model 2, <code>PickUps</code> was added alongside self-report because it is a real behavioral signal (unlike self-report, it's logged, not estimated) and was available for every participant used in the tree.</p>
  </section>

  <section id="preparation">
    <div class="section-num">5. Data Preparation and Feature Selection</div>
    <h2>Cleaning Decisions</h2>
    <div class="card">
      <ol class="plain">
        <li><strong>Missing values:</strong> dropped rows missing any of the model variables (<code>SR_SMU_minsweek</code>, <code>SMU</code>, <code>PickUps</code>, <code>Gender</code>, <code>Device</code>) — 1 of 209 rows was dropped (one participant missing <code>PickUps</code>), leaving 208.</li>
        <li><strong>Duplicates:</strong> checked for duplicate <code>Participant_ID</code> values and found 9 IDs that appear twice (one three times) — but on inspection these are <em>not</em> true duplicate observations: age, gender, and self-report values differ between the repeated-ID rows in most cases, meaning the ID field was reused across what are likely different data-collection sessions rather than the same row being copied. Flagged in Limitations rather than silently dropped, since removing them on ID collision alone isn't justified by the evidence available.</li>
        <li><strong>Encoding:</strong> <code>Gender</code> and <code>Device</code> are one-hot encoded for the decision tree (categorical, no natural numeric order); the regression uses only the two continuous variables it needs, so no encoding is required there.</li>
        <li><strong>Scaling:</strong> not applied — a single-predictor linear regression doesn't need it, and scikit-learn's decision tree splits on raw thresholds regardless of feature scale, so scaling would have had no effect on either model.</li>
        <li><strong>Outliers:</strong> identified via IQR (see Exploration) but kept, not removed — see rationale above.</li>
        <li><strong>Train/test split:</strong> the decision tree uses a 75/25 train/test split, stratified on the target class so both splits keep the same 50/50 balance. The regression is reported on the full sample (n=208) since its purpose here is descriptive — quantifying the actual strength of the self-report/objective-use relationship in this dataset — rather than building a model meant to generalize to new, unseen respondents.</li>
        <li><strong>Data leakage:</strong> the tree's target (<code>engagement_group</code>) is derived entirely from <code>SMU</code>, so <code>SMU</code> itself is excluded from its feature set — using it as both source-of-target and a predictor would leak the answer directly into the model. The split is also stratified and computed only on the training fold's label distribution in spirit (median computed on the full sample, a minor simplification disclosed here rather than hidden).</li>
      </ol>
    </div>
  </section>

  <section id="baseline">
    <div class="section-num">6. Baseline and Model Development</div>
    <h2>What "Beating Chance" Means Here</h2>
    <div class="card">
      <p><strong>Regression baseline:</strong> always predicting the sample mean of <code>SMU</code> (956 minutes) for every participant. By definition this baseline's R² is 0.000 — it's the reference point any real predictor has to beat.</p>
      <p><strong>Classification baseline:</strong> always predicting the majority class. Since the classes are exactly 50/50, this baseline's accuracy is <strong>0.500</strong> — equivalent to a coin flip.</p>
    </div>
    <h3>Model 1 — Linear Regression</h3>
    <p>Predicts <code>SMU</code> from <code>SR_SMU_minsweek</code> alone. Chosen because the research question is specifically about the strength of this one relationship, and linear regression directly estimates its direction and magnitude without assuming anything more complex.</p>
    <h3>Model 2 — Decision Tree Classifier</h3>
    <p>Predicts <code>engagement_group</code> (high/low objective use) from <code>SR_SMU_minsweek</code>, <code>PickUps</code>, <code>Gender</code>, and <code>Device</code> together. Chosen because it can capture threshold effects a linear model can't — e.g., self-report might only matter above a certain pickup count — and because it lets a second, logged behavioral variable (pickups) compete directly against self-report for predictive credit. Hyperparameters (<code>max_depth=4</code>, <code>min_samples_leaf=10</code>) were chosen to keep the tree shallow enough to read and interpret, and to avoid overfitting a sample of only 208 rows; both models use the same fixed <code>random_state</code> so the comparison is reproducible.</p>
  </section>

  <section id="evaluation">
    <div class="section-num">7. Model Evaluation and Selection</div>
    <h2>Results</h2>
    <div class="card">
      <p><strong>Model 1 (regression): r = −0.11, R² = 0.013</strong> (n = 208), vs. a baseline R² of 0.000.</p>
      <p>R² is the appropriate metric here because it directly answers "what share of the variance in actual use does self-report explain?" The answer: about 1.3% — barely above the baseline, and the relationship runs slightly <em>negative</em>. This matches the published study's own finding of essentially no relationship.</p>
    </div>
    <div class="card">
      <p><strong>Model 2 (decision tree): test accuracy = 0.673</strong> (35 of 52 held-out participants correct), vs. a baseline accuracy of 0.500.</p>
      <p>Accuracy is appropriate here because the classes are exactly balanced (50/50), so it isn't distorted by class imbalance the way it could be in a skewed dataset. The tree clears the baseline by 17.3 percentage points — a real, meaningful improvement over chance, even though it's far from perfect.</p>
      <div class="fig" style="margin-top:1rem">
        <img src="assets/bar_baseline_vs_tree.png" alt="Bar chart comparing decision tree accuracy to the majority-class baseline">
      </div>
    </div>
    <p><strong>Which model performed better, and which is "final"?</strong> These two models answer related but different versions of the research question, so "final model" here means: which approach better supports an actual decision. The regression says self-report explains almost nothing on its own. The decision tree, once given a second, logged variable (pickups) to work with, does meaningfully better than chance — suggesting the right fix for weak self-report isn't a fancier model of self-report, it's supplementing or replacing self-report with even one piece of logged behavioral data. If forced to pick one model to act on, the decision tree is the one worth deploying; the regression is the one that correctly diagnoses why self-report alone isn't enough.</p>
  </section>

  <section id="interpretation">
    <div class="section-num">8. Model Interpretation and Insights</div>
    <h2>What the Models Learned</h2>
    <div class="fig">
      <img src="assets/decision_tree.png" alt="Decision tree diagram predicting high vs. low objective social media engagement">
      <p style="font-size:0.9rem;color:#666;margin-top:0.5rem">The fitted decision tree (depth 4). The root split is <code>PickUps</code> (≤ 513/week), not <code>SR_SMU_minsweek</code>.</p>
    </div>
    <div class="fig" style="margin-top:1.5rem">
      <img src="assets/feature_importance.png" alt="Bar chart of feature importances from the decision tree">
      <p style="font-size:0.9rem;color:#666;margin-top:0.5rem"><code>PickUps</code> accounts for roughly 82% of the tree's total splitting power; <code>SR_SMU_minsweek</code> accounts for the rest. <code>Gender</code> and <code>Device</code> contributed nothing.</p>
    </div>
    <p>Every split in the tree is on <code>PickUps</code> or <code>SR_SMU_minsweek</code> — never on gender or device type, which have zero feature importance. That means the tree found no evidence that who someone is (by these two demographic categories) changes how well self-report or pickups predict their actual use; the behavioral signal does all the work.</p>
    <p>The confusion matrix (14 true negatives, 12 false positives, 5 false negatives, 21 true positives) shows the tree is noticeably better at catching true high-users (21 of 26, ~81%) than true low-users (14 of 26, ~54%) — it over-predicts "high" more often than "low." In practice this means the model is more trustworthy when it says someone is a heavy user than when it says they aren't.</p>
    <p><strong>What can be concluded:</strong> self-report alone is a weak signal for actual use (Model 1), but a logged behavioral proxy like phone pickups adds real, measurable predictive power (Model 2) — even though neither model comes close to being a precise instrument. <strong>What can't be concluded:</strong> that self-report is worthless in every context, that pickups causally drive social media use, or that these findings generalize beyond this specific convenience sample.</p>
  </section>

  <section id="limitations">
    <div class="section-num">9. Limitations, Ethics, and Reflection</div>
    <h2>Where This Could Go Wrong</h2>
    <ul class="plain">
      <li><strong>Sample bias.</strong> N = 208 from one convenience sample (likely university-recruited) — not representative of social media users broadly, across age, region, or platform-usage culture.</li>
      <li><strong>Possible ID collisions.</strong> 9 participant IDs recur with differing demographic/self-report values (see Data Preparation) — if these are genuinely the same individuals measured twice rather than an ID-reuse artifact, it would modestly inflate the effective sample size and violate the independence assumption behind both models' significance.</li>
      <li><strong>Unlabeled device-type coding.</strong> The file carries no embedded value label for which numeric code is Android vs. iPhone, so this project cannot say whether device type matters — it can only say the two unlabeled groups didn't differ.</li>
      <li><strong>Who is affected by a wrong prediction:</strong> if a tool like this were used to flag "heavy users" for an intervention (e.g., a digital-wellness nudge or a clinical screener), a <em>false positive</em> (14 real low-users predicted high, in the test-set proportions here) means someone is wrongly flagged or nudged despite normal use; a <em>false negative</em> means a genuinely heavy user is missed and gets no support. Given the tree's own asymmetry (better at catching true highs than true lows), false positives are the more likely error in practice.</li>
      <li><strong>Real-world appropriateness:</strong> given 67.3% accuracy and a tiny regression R², neither model is precise enough to make high-stakes individual decisions (e.g., clinical diagnosis, parental control thresholds) — at most they support population-level research conclusions ("self-report under-performs logged behavior as a predictor"), not individual-level judgments.</li>
      <li><strong>Next steps:</strong> a larger, more diverse sample; genuine behavioral features beyond pickups (e.g., session count, time-of-day patterns); and resolving the device-label and ID-collision ambiguities directly with the dataset's author would all meaningfully strengthen this analysis.</li>
    </ul>
  </section>

  <section id="code">
    <div class="section-num">10. Code and Transparency</div>
    <h2>Code, Data, and AI Disclosure</h2>
    <div class="card">
      <p>Code: <code>generate_graphs.py</code> (exploratory analysis, baselines, modeling, and visualization) — <a href="#">link to your GitHub repository here</a>.</p>
      <p>Dataset: Mahalingham, T. (2022). "Data Set — assessing the validity of self-reported social media use." Mendeley Data, V1 (CC BY 4.0). <a href="https://data.mendeley.com/datasets/x3wxfycggn" target="_blank" rel="noopener">https://data.mendeley.com/datasets/x3wxfycggn</a>.</p>
      <p><span style="background:#FBF0DF;border:1px dashed #C97B3E;padding:0.05rem 0.4rem;border-radius:3px;">Edit this to match your course's exact AI-disclosure policy</span> — e.g., "Claude Sonnet 5 (Anthropic) was used to help locate the real published dataset matching this project's research question, convert it from SPSS format when standard tools were unavailable, write the exploratory-analysis, baseline, and modeling code, and draft this write-up. All research questions, modeling decisions, interpretation, and final editorial decisions are the author's own."</p>
    </div>
  </section>

</main>

<footer>Built with Python (pandas, scikit-learn, matplotlib) for the analysis and plain HTML/CSS for the site. Data: Mahalingham (2022), a real, published, CC-BY-licensed sample of 209 participants.</footer>

</body>
</html>
