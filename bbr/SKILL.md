---
name: bbr
description: >
  Use this skill for ANY question about biostatistics for biomedical research:
  types of measurements, study design, RCTs, observational studies, ANCOVA, sample
  size, power, confidence intervals, hypothesis testing, p-values, comparing
  proportions, nonparametric tests, correlation, serial/longitudinal data, observer
  variability, measurement agreement, propensity scores, information loss from
  dichotomization, biomarker research, sensitivity, specificity, ROC curves, medical
  diagnosis, high-dimensional data, reproducible research, and alpha vs. decision
  error probability. Also trigger for: change from baseline vs. ANCOVA, regression
  to the mean, subgroup analyses, or Frank Harrell's BBR course (hbiostat.org/bbr).
  BBR takes precedence over rms for design, measurement, and inference philosophy;
  rms takes precedence for multivariable modeling mechanics. Always use this skill
  for study design, statistical inference, or biomedical data analysis questions.
---

# Biostatistics for Biomedical Research (BBR)

Primary source: [https://hbiostat.org/bbr](https://hbiostat.org/bbr) — Frank E. Harrell Jr., Vanderbilt University

For deep dives see `references/` subfolder:
- `references/chapters.md`  — chapter-by-chapter section maps and key content
- `references/blog.md`      — curated blog posts relevant to BBR topics

**Skill boundary**: BBR covers conceptual and applied biostatistics including introductory
rms usage (Ch. 9–10). The **rms skill** covers deep multivariable modeling methodology.
For design, measurement, and inference philosophy: **use this skill**. For model fitting,
splines, validation, penalization: **use the rms skill**.

---

## Research Workflow

```
Research Question → Measurements → Design → Data Acquisition
  → Description → Analysis → Interpretation / Prediction → Validation
```

Biostatistics must be integrated from the beginning — not applied after data collection.
Study design and measurement quality are 9/10 of the problem.

---

## Core Philosophy

### Measure the Right Thing First
Before any analysis, ask: is Y measuring what matters clinically? The choice of outcome
variable determines everything downstream. Prefer:
- **High-resolution (ordinal/continuous) outcomes** over binary endpoints
- **Current status at follow-up** over change from baseline
- **Clinically meaningful scales** over surrogate endpoints that may mislead

### Estimation Over Testing
Report effect estimates with confidence intervals. Hypothesis tests answer a limited
"existence" question; they do not quantify the magnitude or direction of an effect.
A p-value > 0.05 is not evidence of no effect — it is often evidence of an underpowered
study. Bayesian posterior probabilities directly answer decision-relevant questions.

### Preserve Information
Dichotomizing continuous variables is almost never justified:
- A binary variable has 1 bit of information; a continuous variable has much more
- Dichotomization discards ≥ 1/3 of the experiment's effective subjects
- Creates arbitrary thresholds, unexplained heterogeneity, and loss of power
- There are almost no true biological thresholds — nature draws blurry lines
- Use the full continuous variable in a model with restricted cubic splines (rms skill)

### Condition Forward, Not Backward
Statistics should condition on what is **known** to predict what is **unknown**:
- Correct: Pr(disease | test result, patient characteristics) — forward probability
- Wrong: Pr(test result | disease status) — sensitivity/specificity — backward probability
- p-values also condition on what is unknown (H0) to make statements about data
- Bayesian posterior probabilities get the conditioning right

---

## Types of Measurements (Ch. 3–4)

**By role in study:**
- Response (Y): what we measure as outcome
- Predictor (X): what we adjust for or study as exposure
- Confounder: related to both X and Y; must be adjusted for

**By coding:**
- Binary: 0/1; 1 bit of information
- Nominal (unordered categorical): k levels → k-1 dummy variables
- Ordinal: ordered levels (discrete or continuous); use rank-based or ordinal models
- Continuous interval-scaled: most information; never categorize

**Choose Y to maximize statistical information and power:**
- Ordinal and continuous outcomes are always at least as informative as any binary
  dichotomization of the same variable
- The Wilcoxon/PO model uses the full ordinal distribution; any binary cutpoint throws
  away information from all other cutpoints
- Composite endpoints can increase power but must be clinically coherent

---

## Statistical Inference (Ch. 5, 12)

### What a p-value Is (and Is Not)
A p-value is Pr(data this extreme or more | H0 is exactly true). It:
- Conditions on the unknown (H0), not on the known (data) — backward in time
- Is NOT the probability the null hypothesis is true
- Is NOT the probability the result occurred by chance
- Is NOT the probability of a replication failing
- Does NOT become more meaningful at p = 0.049 vs. p = 0.051

"Absence of evidence is not evidence of absence." A non-significant result typically
means the study was underpowered, not that the treatment has no effect.

### What α Is (and Is Not) — Ch. 22; see also hbiostat.org/bbr/alpha
α (type I assertion probability) is:
- The probability of asserting an effect **when** there is none
- NOT the probability that a positive result is wrong
- NOT the probability of a decision error

The probability that a **positive result turns out to be wrong** requires:
- Knowledge of the prior probability the null is true
- Bayesian reasoning: Pr(H0 | p < α) ≠ α

Fisher's error: "A man who rejects a hypothesis at 1% will be mistaken in not more
than 1% of such decisions" — wrong, because "of such decisions" is not conditioned on H0.

Optimum α from Bayesian decision analysis: ~0.24 for terminal illness with no existing
therapies; ~0.01 for less severe conditions (Isakov et al., J Econometrics 2019).

### Confidence Intervals
A 95% CI means: if the study were repeated infinitely under the same conditions, 95%
of such intervals would contain the true parameter — it is a long-run coverage statement,
not a probability statement about this particular interval.

Bayesian 95% credible intervals directly state: "there is 95% probability the parameter
lies in this range" — given the data and prior.

### Estimation vs. Testing
Prefer to report:
- Point estimate + 95% CI
- Bayesian: posterior distribution summary + posterior probability of benefit
- Avoid: "statistically significant" / "not significant" binary framing
- Report the specific alternative that is ruled out with 95% confidence

---

## Study Design

### Randomized Controlled Trials (Ch. 3, 13)
Randomization eliminates confounding by indication and balances measured and unmeasured
baseline covariates on average. RCTs are causal by design when analyzed respecting
randomization.

**Key RCT analysis principles (Ch. 13):**
- **Always adjust for pre-specified baseline covariates** (ANCOVA)
  - Linear models: reduces residual variance → increases power
  - Nonlinear models (logistic, Cox): prevents power loss from outcome heterogeneity
  - A misspecified covariate adjustment performs better than no adjustment
- **Do not analyze change from baseline** as the primary endpoint (see below)
- Pre-specify the primary endpoint, analysis model, and covariates before unblinding
- For multi-site trials, site is usually better handled as a covariate than a stratum

**Covariate adjustment in RCTs — non-collapsibility:**
Unadjusted odds ratios and hazard ratios apply only to a homogeneous patient population.
When risk factors exist (which is almost always), the unadjusted OR/HR is attenuated
toward 1.0 relative to the covariate-adjusted value. The adjusted estimate is the
clinically meaningful one — it answers "for this patient, how much benefit?" The
unadjusted estimate applies to no one in particular (fharrell.com/post/marg).

**Subgroup analyses:**
- Pre-specified interaction tests only; never post-hoc
- Test interaction formally (interaction term in model) rather than separate subgroup p-values
- Most apparent subgroup effects are noise; require very large samples to detect genuine interactions
- Report the interaction effect estimate and its CI, not just p-values within subgroups

### Observational Studies (Ch. 17)
Observational treatment comparisons face:
- Confounding by indication (sicker patients may be more or less likely to get treatment A)
- Unmeasured confounders (the fundamental limitation)
- No unique way to adjust for measured confounders

**Propensity score approach (when # confounders >> ESS for Y):**
- PS = Pr(treatment B | baseline covariates), estimated via logistic regression with splined continuous predictors
- Adjust outcome model for spline(logit(PS)) + pre-specified strong prognostic factors
- PS is a confounder funnel, NOT a causal inference tool
- **Propensity score matching:** usually inferior — discards valuable observations, ignores outcome heterogeneity, and underestimates treatment effects in nonlinear models

**Best practice for observational studies:**
- Full covariate adjustment with penalization (or Bayesian shrinkage) if # covariates is large
- Model Y ~ treatment + spline(logit(PS)) + key prognostic factors
- Never retroactively rationalize adequacy of convenience data
- Prospective observational design is essential for credible causal inference

---

## Change from Baseline (Ch. 14) — A Major Pitfall

**Do not use change from baseline as the primary outcome.** Use ANCOVA instead:
model follow-up Y as the response, with baseline Y as a covariate.

Change scores require all of the following to hold simultaneously:
1. Variable not used as inclusion/exclusion criterion (otherwise regression to the mean)
2. Follow-up linearly related to baseline with slope ≈ 1
3. Same transformation works for both pre and post
4. No floor or ceiling effects
5. No regression to the mean

These rarely all hold. ANCOVA is uniformly more efficient and makes fewer assumptions.

**Why ANCOVA wins:**
- Answers the fundamental question: "for two patients with the same baseline, one on A and one on B, how do their expected outcomes differ?"
- Does not assume slope of baseline = 1
- Works even when the relationship between baseline and follow-up is nonlinear
- The PO ordinal model for ANCOVA requires no transformation assumption at all

**Percent change** is even worse: it assumes a specific multiplicative relationship and
is not interpretable when the denominator (baseline) varies.

**Current status vs. change:**
What matters to the patient is their current status, not how much they changed.
A patient with severe symptoms who improved slightly is still severely symptomatic.
Model current status; baseline is just a covariate.

---

## Nonparametric Methods (Ch. 7–8)

Prefer nonparametric or semiparametric methods when distributional assumptions are uncertain:
- Wilcoxon rank-sum = special case of proportional odds model (two groups, no covariates)
- Wilcoxon signed-rank for paired data (but rank-difference test is preferred; see rms/ordinal.md)
- Kruskal-Wallis = special case of PO model (k groups)
- Spearman ρ for correlation when normality is doubtful

The proportional odds (PO) model strictly generalizes all three rank tests and adds:
- Covariate adjustment
- Continuous Y with arbitrary ties
- Formal effect estimates (OR, quantiles, means, exceedance probabilities)

For correlation with confounding: use partial Spearman correlation or model-based approaches.

---

## Comparing Proportions (Ch. 6)

- Fisher's exact test or χ² for 2×2 tables; prefer exact when expected cells < 5
- Relative risk (RR) vs. odds ratio (OR): RR is more interpretable but OR is needed for logistic regression and non-collapsibility arguments
- For adjusted comparisons: use logistic regression (lrm) rather than stratified tables
- Avoid the risk difference as a primary measure in heterogeneous populations (it varies with baseline risk)
- Number needed to treat (NNT) = 1/absolute risk reduction; useful clinically but varies with baseline risk

---

## Diagnosis and Biomarkers (Ch. 19) — The Backward Probability Problem

**Sensitivity and specificity are backward-time measures:**
- Sensitivity = Pr(test+ | disease+) — conditions on what you don't know at test time
- Specificity = Pr(test– | disease–) — same problem
- Useful only for proof-of-concept case-control studies where sampling is on disease status
- In cohort studies, they require complex workup-bias adjustments

**What clinicians and patients need:**
- Pr(disease | test result, patient characteristics) — forward probability
- Derived directly from a logistic regression model applied to the patient's data
- No Bayes theorem contortions, no prevalence adjustment needed

**ROC curves:**
- Area under ROC (AUC/C-statistic) measures pure discrimination — useful
- The ROC curve itself is a set of sensitivity/specificity pairs across thresholds — not directly useful for individual decision-making
- Optimum threshold selection from ROC is wrong: threshold should come from the decision-maker's utility function, not from the curve

**Probabilistic diagnosis model (preferred):**
- Fit logistic regression: Pr(disease | test results, patient covariates)
- Pre-test probability is incorporated via the model's intercept/baseline covariates
- Post-test probability is the model's predicted probability for the specific patient
- Works for continuous, multi-valued, and multiple correlated tests simultaneously
- Ordinal logistic model handles multiple disease severity stages without forcing binary

**BI-RADS example:** Category 4 spans 3% to 94% malignancy probability — a single
category that is useless for decision-making. A probability model would give each
patient their specific risk estimate.

---

## Information Loss (Ch. 18)

**Categorizing continuous predictors:**
- Is almost never justified — genuine biological thresholds are rare
- Creates arbitrary cutpoints that differ across studies
- Reduces power (equivalent to discarding ~1/3 of subjects)
- Produces unexplained heterogeneity of response
- Solution: use restricted cubic splines in rms (see rms skill)

**Categorizing outcomes:**
- Any binary cutpoint throws away all information from other cutpoints
- The CAST trial (cardiac arrhythmia): suppressing PVCs (a binary endpoint) reduced
  arrhythmia but increased mortality — a warning about surrogate binary endpoints
- Crohn's disease example: faulty binary responder definition missed treatment signal
  visible in the continuous CDAI scale

**Classification vs. probability:**
- Forced binary classification (disease yes/no) is premature decision-making
- Probability estimates are strictly more useful: the classifier is derived from them,
  not vice versa
- Optimal decisions require: probability estimate + patient-specific utility function
- The analyst should not make the clinical decision by choosing a threshold

---

## Observer Variability and Agreement (Ch. 16)

- **Intraclass correlation coefficient (ICC)**: preferred over Pearson r for reliability
- **Bland-Altman plots**: for assessing agreement between two measurement methods
  (limits of agreement, not correlation)
- **Cohen's κ**: for categorical agreement; has problems with marginal distributions
- **Weighted κ**: for ordinal categories
- Distinguish: reliability (consistency of repeated measurements) vs. validity
  (measuring the intended construct)
- Observer variability must be quantified before a measurement can be used in research

---

## Serial and Longitudinal Data (Ch. 15)

Key principles:
- Model the entire trajectory, not just a change score or final value
- Account for within-subject correlation (repeated measures are not independent)
- Methods: mixed-effects models, GLS (generalized least squares), GEE
- For non-normal Y: semiparametric ordinal longitudinal models (rms Ch. 22)
- Avoid: ANOVA repeated measures when missing data are common; use mixed models instead
- Summary statistics (e.g., AUC of serial measurements) discard timing information

---

## High-Dimensional Data (Ch. 20)

- P >> N creates serious overfitting: any data-driven variable selection is unreliable
- Bootstrap importance rankings: wide uncertainty in feature ranks even at moderate N
- One-at-a-time feature modeling (OAAT): bootstrap CI for importance of each feature
  separately to quantify reliability before claiming "important" findings
- Regularization (LASSO, ridge, elastic net) is necessary; plain stepwise selection is wrong
- Claims of improved prediction from high-dimensional biomarker panels require large N
  and rigorous external validation
- Sample size to estimate a correlation matrix reliably: much larger than most studies

---

## Reproducible Research (Ch. 21)

- **Decline effect**: initially large effects shrink or disappear on replication — caused
  by publication bias, HARKing (hypothesizing after results known), and p-hacking
- Use Quarto/R Markdown for fully reproducible analyses embedded in reports
- Pre-register hypotheses and analysis plans before data collection
- Archive data and code (OSF, Zenodo, GitHub)
- Distinguish confirmatory (pre-specified) from exploratory (hypothesis-generating) analyses

---

## R Packages and Basic Usage

```r
library(Hmisc)      # describe(), rcorr(), spearman2(), aregImpute(), varclus()
library(rms)        # ols(), lrm(), orm(), cph(); intro in Ch. 9-10
library(data.table) # data manipulation
library(ggplot2)    # graphics
```

### Describing Data (Hmisc)
```r
describe(dt)              # n, missing, distinct, mean, quantiles, extremes
html(describe(dt))        # rendered HTML table
plot(describe(dt))        # spike histograms for all variables
rcorr(as.matrix(dt[, .(x1, x2, x3)]), type = "spearman")  # rank correlation matrix
```

### Nonparametric Tests
```r
wilcox.test(y ~ group, data = dt)          # Wilcoxon rank-sum
kruskal.test(y ~ group, data = dt)         # Kruskal-Wallis
# Better: use lrm() for PO model — generalizes both with covariate adjustment
library(rms)
f <- lrm(y ~ treatment, data = dt)        # PO model = generalized Wilcoxon
```

### Comparing Proportions
```r
prop.test(c(events_a, events_b), c(n_a, n_b))
fisher.test(table(dt$group, dt$outcome))  # when expected cells < 5
# For adjusted comparison:
f <- lrm(outcome ~ treatment + rcs(age, 4) + sex, data = dt, x=TRUE, y=TRUE)
```

### ANCOVA in RCTs (preferred over change scores)
```r
# Model follow-up Y with baseline as covariate
f <- ols(followup_y ~ treatment + rcs(baseline_y, 4) + rcs(age, 3) + sex,
         data = dt, x = TRUE, y = TRUE)
# For non-normal/ordinal Y:
f <- lrm(followup_y ~ treatment + rcs(baseline_y, 4) + rcs(age, 3) + sex,
         data = dt, x = TRUE, y = TRUE)
```

### Diagnostic Probability Model (preferred over sens/spec)
```r
# Logistic regression gives Pr(disease | test, covariates) directly
f <- lrm(disease ~ test_result + rcs(age, 4) + sex + rcs(pretest_prob, 4),
         data = dt, x = TRUE, y = TRUE)
# Predicted probabilities = post-test probabilities for each patient
predict(f, newdata = newpt, type = "fitted")
```

---

## Common Errors and Corrections

| Error | Correction |
|---|---|
| Analyze change from baseline | ANCOVA: model follow-up Y with baseline as covariate |
| Dichotomize continuous predictor | Use rcs() in regression model |
| Use sensitivity/specificity in cohort study | Use logistic regression probability model |
| Interpret p > 0.05 as "no effect" | Report CI ruling out effects above a threshold |
| Report α as "probability result is wrong" | α is not a decision error probability; use Bayesian posterior |
| Subgroup analysis without interaction test | Test interaction term formally in overall model |
| Unadjusted OR/HR in heterogeneous population | Covariate-adjusted model (ANCOVA) |
| Propensity score matching | PS in outcome model with prognostic covariate adjustment |
| ROC threshold as clinical decision rule | Decision from utility function, not ROC curve |
| Percent change as outcome | Model follow-up on original or log scale; baseline as covariate |

---

## Chapter Map

| Chapter | Topic | URL |
|---|---|---|
| 3 | Overview, measurement types, Y choice | hbiostat.org/bbr/overview |
| 4 | Descriptive statistics and graphics | hbiostat.org/bbr/descript |
| 5 | Statistical inference, p-values, CIs | hbiostat.org/bbr/htest |
| 6 | Comparing two proportions | hbiostat.org/bbr/prop |
| 7 | Nonparametric tests | hbiostat.org/bbr/nonpar |
| 8 | Correlation | hbiostat.org/bbr/corr |
| 9–10 | Intro to rms, regression overview | hbiostat.org/bbr/rmsintro |
| 13 | ANCOVA in RCTs | hbiostat.org/bbr/ancova |
| 14 | Change scores, transformations, RTM | hbiostat.org/bbr/change |
| 15 | Serial/longitudinal data | hbiostat.org/bbr/serial |
| 16 | Observer variability, agreement | hbiostat.org/bbr/obsvar |
| 17 | Observational treatment comparisons | hbiostat.org/bbr/propensity |
| 18 | Information loss, dichotomization | hbiostat.org/bbr/info |
| 19 | Diagnosis, sens/spec, ROC | hbiostat.org/bbr/dx |
| 20 | High-dimensional data | hbiostat.org/bbr/hdata |
| 21 | Reproducible research | hbiostat.org/bbr/repro |
| 22 | α vs. decision error probability | hbiostat.org/bbr/alpha |

For detailed section maps → `references/chapters.md`
For blog post summaries → `references/blog.md`
