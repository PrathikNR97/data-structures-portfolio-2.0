# Impulse Buying and Impulsive Spending: What Actually Predicts It?

A secondary analysis of a published behavioral-economics experiment, examining whether trait impulsivity and a simple checkout delay affect how much people spend on impulse.

---

## Table of Contents

1. [Research Question](https://www.google.com/search?q=%2523research-question&utm_source=gemini)
2. [Dataset](https://www.google.com/search?q=%2523dataset&utm_source=gemini)
3. [Variables](https://www.google.com/search?q=%2523variables&utm_source=gemini)
4. [Data Cleaning (pandas)](https://www.google.com/search?q=%2523data-cleaning-pandas&utm_source=gemini)
5. [Visualizations](https://www.google.com/search?q=%2523visualizations&utm_source=gemini)
6. [Limitations](https://www.google.com/search?q=%2523limitations&utm_source=gemini)
7. [Code & AI Usage Disclosure](https://www.google.com/search?q=%2523code--ai-usage-disclosure&utm_source=gemini)
8. [References](https://www.google.com/search?q=%2523references&utm_source=gemini)

---

## Research Question

Do people with a higher trait tendency toward impulse buying actually spend more money impulsively when given the chance to shop freely? And separately: does forcing a short delay before checkout reduce how much people spend on impulse purchases?

This is answerable and specific because both variables — trait impulsivity (a validated 20-item scale) and actual dollars spent impulsively — were measured directly in a real shopping session, rather than relying on retrospective self-report of "how much do you overspend."

---

## Dataset

* **Source:** Moser, C., Schoenebeck, S., & Resnick, P. (2020). *Impulse Buying: Designing for Self-Control with E-commerce* [Data set]. University of Michigan – Deep Blue Data. `deepblue.lib.umich.edu/data/concern/data_sets/m039k5047`. Licensed CC BY 4.0. Specifically, this analysis uses the Study 4 file (`Study4_Data_Public_Mar10.csv`), an in-lab experiment.
* **Unit of analysis:** One row = one study participant who completed a real, unscripted 15-minute Amazon shopping session in a lab setting.
* **Size:** 131 participants (rows), 82 raw columns — narrowed to 10 relevant columns for this analysis (see Variables, below).
* **Missing values:** The core numeric variables used here (impulse dollars spent, impulse-buying scale score, treatment group, age, income, education) have zero missing values for all 131 participants. One variable, `Average_Regret`, is missing for 105 of 131 participants (80%) — this is not a data quality problem: regret about an impulse purchase is undefined for the 105 participants who never made one, so it's a structural (not random) missing pattern.

---

## Variables

The experiment placed participants in a real online shopping environment and either let them check out immediately or forced a 10-minute wait before checkout, to test whether a brief "cooling-off" period curbs impulsive spending (Moser et al., 2019).

| Variable | What it measures |
| --- | --- |
| **IB_SCALE** | Trait impulse-buying tendency, a validated 20-item personality scale (Verplanken & Herabadi, 2001) completed before shopping. Range ~5–35; higher = more impulsive shopping personality in general. |
| **TotalImpulseDollars** | Actual dollars spent on items the participant classified as unplanned/impulsive during the session — the primary "overspending" measure in this study. |
| **TotalImpulseProducts** | Count of impulse items purchased in the session. |
| **TreatmentGroup** | Random assignment: 0 = checked out immediately, 1 = forced 10-minute delay before checkout. |
| **Age, Income, Education** | Demographic controls, included to check whether spending differences are confounded by these rather than by impulsivity or the delay. |
| **Average_Regret** | Post-purchase regret (1–7), only defined for participants who made an impulse purchase. |

---

## Data Cleaning (`pandas`)

All cleaning steps and the reasoning behind each are documented inline in `generate_graphs.py`. Summary:

* **Column selection:** narrowed the raw 82-column export down to the 10 columns this analysis uses (demographic sub-flags, open-ended coding fields, and SPSS filter variables were dropped as out of scope).
* **Missing-value check:** verified with `df.isna().sum()` that all core numeric variables were fully populated — no imputation was needed or applied to them.
* **String-to-numeric conversion:** `Average_Regret` was exported as text because 105 participants have a blank space (`" "`) instead of a number. Rather than treating that blank as 0 (which would falsely imply "no regret"), it was converted to NaN via `pd.to_numeric(..., errors="coerce")`, preserving the fact that regret is undefined — not zero — for someone who bought nothing impulsively.
* **Recoding for readability:** `TreatmentGroup` (0/1) was mapped to plain-language labels (`"No delay (control)"` / `"10-minute delay"`) purely for chart labeling.
* **No transformation of the spending variable:** `TotalImpulseDollars` is heavily right-skewed with a large spike at $0. It was left untransformed (not log-scaled) because $0 is a real, meaningful value here — most participants genuinely bought nothing on impulse — and log-transforming would obscure that floor.

---

## Visualizations

### 1. Trait impulsivity vs. actual impulsive spending

* **Scatter plot** of impulse-buying tendency score versus dollars spent impulsively, showing a weak positive relationship ($n = 131$). Each point is one participant.
* **What it shows:** The relationship is positive but weak (Pearson $r = 0.155$). Most participants — regardless of their trait impulsivity score — spent exactly $0 impulsively (the dense row of points along the bottom). Among those who did spend impulsively, there's no clean pattern where higher trait impulsivity predicts higher dollars spent; some of the highest spenders have only moderate `IB_SCALE` scores. This suggests a validated personality trait measured in the abstract doesn't translate cleanly into predicting a specific spending amount in a specific 15-minute session — situational factors likely matter more than the trait alone.

### 2. Impulsive spending by checkout-delay condition

* **Box and whisker plot** comparing impulsive spending between the no-delay control group and the 10-minute delay group ($n = 131$; 66 control, 65 delay). Circles are individual outlier values above the whisker.
* **What it shows:** Both groups have a median of $0 — most people in either condition didn't spend impulsively at all. But the control group's middle 50% of spenders (the box) reaches up to about $10, while the delay group's box is compressed to $0, with its impulsive spenders showing up only as individual outlier points (up to ~$19). This visually matches the original study's finding: a 10-minute delay did not produce a statistically significant drop in impulsive dollars spent, largely because most participants kept shopping (rather than stepping away) during the delay itself.

---

## Limitations

* **Small, non-representative sample:** 131 participants from a single in-lab study (likely recruited from a university population) cannot be assumed to represent the general U.S. adult population's shopping behavior.
* **Missing category labels:** The public CSV does not ship with a full codebook alongside it in this repository, so the exact category labels for Income and Education bands (e.g., what "3" means) are not confirmed here — they're used only as ordinal controls, not interpreted by exact category.
* **Single shopping session:** $0 impulsive spending in a single 15-minute Amazon session does not mean someone never overspends — it may just mean nothing caught their eye that day. A longitudinal or diary-based design would better capture "impulse buying frequency" as the original research questions envisioned it.
* **Lab setting, not naturalistic:** Participants knew they were being observed shopping for a study, which plausibly suppressed impulsive behavior relative to shopping alone at home.
* **Unanswered questions:** Whether the delay's lack of effect would hold with a longer delay (the original dissertation's Study 3 tested ~25 hours and found a real effect), and whether income or education moderate the trait-impulsivity/spending relationship, are both open questions this dataset can partially but not fully answer.

---

## Code & AI Usage Disclosure

### Code

Full analysis code (data loading, cleaning, and chart generation): `[github.com/your-github-handle/your-repo-name](https://github.com/your-github-handle/your-repo-name)` — see `generate_graphs.py`.

### AI usage disclosure

> *Edit this to reflect your actual process — for example:* "Claude (Anthropic) was used to help locate a publicly available dataset matching the research question, write and comment the pandas data-cleaning script, generate the two visualizations, and draft this write-up. All research questions, interpretation of results, and final editorial decisions are the author's own."

---

## References

* Moser, C., Schoenebeck, S. Y., & Resnick, P. (2019). Impulse buying: Design practices and consumer needs. In *Proceedings of the 2019 CHI Conference on Human Factors in Computing Systems* (pp. 1–15). Association for Computing Machinery.
* Rook, D. W. (1987). The buying impulse. *Journal of Consumer Research*, 14(2), 189–199.
* Verplanken, B., & Herabadi, A. (2001). Individual differences in impulse buying tendency: Feeling and no thinking. *European Journal of Personality*, 15(S1), S71–S83.
