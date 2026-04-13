# BBR Chapter Reference — Detailed Section Maps

Source: https://hbiostat.org/bbr

---

## Ch. 3 — General Overview of Biostatistics
https://hbiostat.org/bbr/overview

Key sections:
- **3.4 Types of Measurements by Role**: response, predictor, confounder; proper response variables
- **3.5 Types of Measurements by Coding**: binary, nominal, ordinal, continuous
- **3.6 Choosing Y**: maximize statistical information, power, interpretability
  - 3.6.1 Information content of different measurement scales
  - 3.6.2 Dichotomization — almost always wrong
  - 3.6.3 Change from baseline — almost always wrong
- **3.7 Preprocessing**: transformations, missing data, detection limits
- **3.8–3.9 Random variables and probability**

Key philosophy:
- Fundamental principles of statistics: use all information in data, strive for optimal
  quantification of evidence, give decision makers forward probabilities not backward ones
- Sensitivity/specificity condition on the unknown: they are NOT directly actionable
- p-values condition on H0 being true: also NOT directly actionable
- Statistical inference should move forward in time, not backward

---

## Ch. 4 — Descriptive Statistics, Distributions, and Graphics
https://hbiostat.org/bbr/descript

Key sections:
- Distributions: empirical CDF, spike histograms, Q-Q plots
- Robust summaries: median, IQR, quantiles (preferred over mean/SD for skewed data)
- Graphics for comparing groups: box plots, dot plots, violin plots
- Displaying model fits graphically
- Single-axis nomograms as graphical summaries

R tools:
```r
library(Hmisc)
describe(dt)              # comprehensive variable summaries
html(describe(dt))        # HTML rendering
Ecdf(dt$x)                # empirical CDF plot
histSpike(dt$x)           # spike histogram
```

---

## Ch. 5 — Statistical Inference
https://hbiostat.org/bbr/htest

Key sections:
- **5.3 Branches**: frequentist, Bayesian, likelihoodist
- **5.4 Errors and p-values**: type I/II, common misinterpretations
  - p-value is Pr(data this extreme | H0 true) — not Pr(H0 | data)
  - Large p-value ≠ evidence of no effect
- **5.5 Interval estimation**: frequentist CI (long-run coverage) vs Bayesian credible interval (direct probability statement)
- **5.6–5.10 One- and two-sample tests**: t-test, paired t-test, crossover design
  - With Bayesian analogs for each
- **5.11 Problems with tests and p-values**: how to present results properly
  - Wrong: "not significantly different"
  - Right: "rules out, with 95% confidence, a reduction greater than X"
- **5.12 Study design**: sizing pilots, standardized effect sizes (avoid), multiple estimands
  - Multiple endpoints: pre-specify primary; ordinal composite reduces multiplicity
- **5.13–5.14 Comprehensive examples**: paired sleep data, crossover design

Sample size concepts:
- Size for given power: requires assumed effect size, σ, α, desired power (1-β)
- Size for given precision: margin of error approach — often more honest
- Standardized effect sizes (Cohen's d etc.) are problematic: depend on the SD which
  varies across populations; prefer effects in natural units

---

## Ch. 6 — Comparing Two Proportions
https://hbiostat.org/bbr/prop

Key sections:
- Fisher's exact test: exact, preferred when expected cell counts < 5
- χ² test: asymptotic approximation
- Risk difference, relative risk, odds ratio: when each is appropriate
- Sample size for proportions comparison
- Non-collapsibility of OR: unadjusted OR ≠ adjusted OR even without confounding
- Use logistic regression (lrm) for adjusted comparisons

---

## Ch. 7 — Nonparametric Statistical Tests
https://hbiostat.org/bbr/nonpar

Key sections:
- **Wilcoxon rank-sum** (Mann-Whitney): two-sample; special case of PO model
- **Kruskal-Wallis**: k-sample; special case of PO model
- **Wilcoxon signed-rank**: paired data; prefer rank-difference test (see fharrell.com/post/pair)
- **Proportional odds model as generalization** of all rank tests:
  - lrm(y ~ group): equivalent to Wilcoxon/KW
  - Advantages: covariate adjustment, estimable effects (OR, quantiles, means)
- **7 (sec-nonpar-ecdf)** Sample size for characterizing entire distributions (ECDF)
- **7 (sec-nonpar-popower)** Power/sample size for PO comparisons: `Hmisc::popower()`, `posamsize()`
- **7 (sec-nonpar-2way)** Two-way ANOVA example using ordinal regression
- **7 (sec-nonpar-pairmodel)** Regression analysis of paired data with PO model

```r
# Power for Wilcoxon/PO model comparison
library(Hmisc)
popower(p1 = c(0.1, 0.2, 0.3, 0.4),   # outcome distribution under control
        odds.ratio = 1.5,              # assumed treatment OR
        n = 200)
posamsize(p1 = c(0.1, 0.2, 0.3, 0.4), odds.ratio = 1.5, power = 0.80)
```

---

## Ch. 8 — Correlation and Nonparametric Regression
https://hbiostat.org/bbr/corr

Key sections:
- Pearson r: assumes bivariate normality, sensitive to outliers
- Spearman ρ: rank-based, robust; preferred when normality uncertain
- Kendall τ: another rank correlation; interpretable as concordance probability minus discordance
- `rcorr()` in Hmisc: matrix of Pearson or Spearman correlations with p-values
- `spearman2()`: generalized Spearman ρ² for predictor importance ranking (used in rms)
- Hoeffding's D: detects any monotone or non-monotone association
- Sample size for estimating a single correlation: `Hmisc::pcor()` or `pwr::pwr.r.test()`
- Loess and other nonparametric smoothers for visualizing relationships

```r
library(Hmisc)
rcorr(as.matrix(dt[, .(x1, x2, y)]), type = "spearman")
spearman2(y ~ x1 + x2 + x3 + x4, data = dt, p = 2)  # guides knot allocation in rms
hoeffd(as.matrix(dt[, .(x1, y)]))                     # Hoeffding's D
```

---

## Ch. 9–10 — Introduction to rms and Regression Overview
https://hbiostat.org/bbr/rmsintro
https://hbiostat.org/bbr/reg

BBR introduces rms at a conceptual level for linear models (Ch. 9) and overview of
multiple regression and model validation (Ch. 10). Deep methodology → use **rms skill**.

Key BBR-level concepts:
- datadist() setup
- ols() for continuous Y
- Interpret: anova(), summary(), Predict(), nomogram()
- Overview of overfitting and bootstrap validation (concept level)
- R² as explained variation; limitations

---

## Ch. 11 — Multiple Groups
https://hbiostat.org/bbr/multgroup

- One-way ANOVA: F-test for overall group difference
- Post-hoc comparisons: Tukey, Dunnett; always adjust for multiplicity
- Contrast coding: planned comparisons; use contrast() in rms
- Prefer ordinal model (lrm/orm) over ANOVA for non-normal Y
- Non-parametric analog: Kruskal-Wallis → PO model

---

## Ch. 12 — Statistical Inference Review
https://hbiostat.org/bbr/infreview

- Consolidation of inference concepts from Ch. 5
- Power and sample size review
- Common errors in interpretation catalogued
- Transition to more complex models

---

## Ch. 13 — Analysis of Covariance in Randomized Studies
https://hbiostat.org/bbr/ancova

**Big picture:**
ANCOVA approximates a design where each patient is their own control. It conditions on
known patient characteristics to isolate the treatment effect.

Key sections:
- **13.1 Linear models**: covariate adjustment increases power by reducing residual σ²
- **13.2 Nonlinear models**: adjustment prevents power loss from outcome heterogeneity;
  unadjusted OR/HR in heterogeneous population is attenuated toward null (non-collapsibility)
- **13.2.1 Hidden assumptions in 2×2 tables**: assumes all patients in a treatment group
  have identical outcome probability — almost never true
- **13.3 Cox/log-rank test**: adjustment equally important; unadjusted HR is a mixture HR
- **13.5 How many covariates?**: all pre-specified strong prognostic factors; not too many
  relative to ESS (follow rms skill guidance on df allocation)
- **13.6 Differential (HTE) and absolute treatment effects**:
  - HTE = interaction between treatment and covariate; requires formal interaction test
  - Absolute treatment effect = difference in predicted probability for same patient
    under A vs. B; varies with patient characteristics
- **13.9 Statistical analysis plan template**: Bayesian SAP subsection (2025)
- **13.10 Robustness**: ANCOVA is robust to mis-specification of covariate functional form;
  any covariate adjustment beats no adjustment

```r
# Correct ANCOVA for RCT with continuous Y
f <- ols(followup ~ treatment * rcs(baseline, 4) + rcs(age, 3) + sex,
         data = dt, x = TRUE, y = TRUE)
# If interaction with baseline not significant, simplify:
f <- ols(followup ~ treatment + rcs(baseline, 4) + rcs(age, 3) + sex,
         data = dt, x = TRUE, y = TRUE)
anova(f)  # chunk test for treatment effect

# Ordinal ANCOVA for non-normal/ordinal follow-up Y
f <- lrm(followup ~ treatment + rcs(baseline, 4) + rcs(age, 3) + sex,
         data = dt, x = TRUE, y = TRUE)
```

---

## Ch. 14 — Transformations, Measuring Change, and Regression to the Mean
https://hbiostat.org/bbr/change

Key sections:
- **14.1–14.2 Transformations**: log transform for skewed data; natural log only
- **14.4 What's wrong with change** (sec-change-gen):
  - requires slope(baseline→follow-up) ≈ 1
  - requires linear pre-post relationship
  - fails when variable used as inclusion criterion (regression to the mean)
  - KCCQ ceiling effect example (sec-change-kccq): ceiling effects make change scores misleading
  - Nearly optimal model: current status with baseline covariate (sec-change-opmod)
- **14.5 Percent change**: multiplicative model assumption, not interpretable when baseline varies
- **14.6 Objective method for choosing effect measure**: likelihood-based approach
- **14.8 Regression to the mean** (sec-change-rttm):
  - Extreme observations regress toward the mean on re-measurement
  - Affects any study using baseline as inclusion criterion
  - Creates apparent "improvement" in both arms even without treatment

---

## Ch. 15 — Serial Data
https://hbiostat.org/bbr/serial

Key sections:
- Serial measurements: repeated measures within subject over time
- Avoid: summarizing serial data as a single change score or AUC
- Avoid: traditional repeated-measures ANOVA when data are unbalanced or missing
- Prefer: mixed-effects models or GLS (nlme package, see rms skill for gls())
- For ordinal/non-normal Y: semiparametric ordinal longitudinal models (RMS Ch. 22)
- Response trajectories: model trajectory shape, test whether treatment alters trajectory
- Modeling interactions with time: treatment × time interaction for differential trajectories

```r
library(nlme)
# GLS with AR(1) within-subject correlation
f <- gls(y ~ time * treatment + rcs(age, 3),
         correlation = corCAR1(form = ~time | id),
         data = dt_long)
```

---

## Ch. 16 — Observer Variability and Measurement Agreement
https://hbiostat.org/bbr/obsvar

Key sections:
- Sources of variability: within-observer, between-observer, within-subject, between-subject
- **Intraclass correlation coefficient (ICC)**:
  - ICC(1,1): single rater, absolute agreement
  - ICC(2,1): random raters, consistency
  - ICC(3,1): fixed raters, consistency
  - ICC > 0.9 generally needed for clinical use
- **Bland-Altman plots**: plot (measurement1 - measurement2) vs. mean;
  limits of agreement = mean difference ± 1.96 × SD(differences)
  - Do NOT use Pearson correlation to assess agreement
- **Cohen's κ** for categorical ratings:
  - κ = (observed agreement - chance agreement) / (1 - chance agreement)
  - Problematic when marginal distributions are very unequal
  - Weighted κ for ordinal categories
- **Kappa vs. ICC**: ICC preferred for continuous ratings; κ for nominal

```r
library(Hmisc)
# ICC
library(irr)
icc(ratings_matrix, model = "twoway", type = "agreement")

# Bland-Altman
dt[, diff := method1 - method2]
dt[, mean_val := (method1 + method2) / 2]
mean_diff <- mean(dt$diff)
loa_upper <- mean_diff + 1.96 * sd(dt$diff)
loa_lower <- mean_diff - 1.96 * sd(dt$diff)
ggplot(dt, aes(x = mean_val, y = diff)) +
  geom_point() +
  geom_hline(yintercept = c(mean_diff, loa_upper, loa_lower),
             linetype = c("solid", "dashed", "dashed"))
```

---

## Ch. 17 — Modeling for Observational Treatment Comparisons
https://hbiostat.org/bbr/propensity

Key sections:
- **17.2 Challenges**: confounding by indication, unmeasured confounders, time-zero definition
- **17.3 Propensity score**: PS = Pr(treatment B | confounders); fit logistic model with splined continuous confounders
- **17.4 Misunderstandings about PS**: PS is not a causal inference tool; it is a confounder funnel
- **17.5.1 Problems with PS matching**: discards observations, ignores outcome heterogeneity, underestimates effects in nonlinear models, is dataset-order-dependent
- **17.6 Recommended SAP**:
  1. Fit PS model: treatment ~ additive model in all potential confounders (splined)
  2. Fit outcome model: Y ~ treatment + spline(logit(PS)) + pre-specified prognostic factors
  3. Use penalization if # confounders is large relative to ESS for Y
- **17.7–17.8 Failure modes and sensitivity analysis**

```r
# Step 1: fit propensity score model
ps_model <- lrm(treatment ~ rcs(age,4) + sex + rcs(severity,4) + comorbidity,
                data = dt, x = TRUE, y = TRUE)
dt[, ps := predict(ps_model, type = "fitted")]
dt[, logit_ps := qlogis(ps)]

# Step 2: outcome model with PS adjustment
outcome_model <- lrm(outcome ~ treatment + rcs(logit_ps, 4) + rcs(severity, 3),
                     data = dt, x = TRUE, y = TRUE)
anova(outcome_model)
```

---

## Ch. 18 — Information Loss
https://hbiostat.org/bbr/info

Key sections:
- **18.1 Information allergy**: refusing to use available information; epidemic in biomedical research
- **18.3 Categorization of continuous predictors**:
  - Information bits: binary = 1 bit; continuous = many more
  - Categorization ≈ discarding 1/3+ of experimental subjects
  - No true biological thresholds exist for most variables
  - sec-info-catoutcomes: categorizing outcomes is equally damaging
- **18.4 Classification vs. probability**: forced decisions discard uncertainty;
  probability estimates preserve all information for decision-making
- **18.6.1–18.6.2 CAST trial**: suppressing PVCs (binary surrogate) killed patients;
  continuous ECG measures would have been more informative
- **18.7 Crohn's disease case study**: binary responder definition at arbitrary CDAI
  threshold missed signal visible in continuous CDAI analysis

---

## Ch. 19 — Diagnosis
https://hbiostat.org/bbr/dx

Key sections:
- **19.1 Problems with sensitivity/specificity**:
  - Backward time conditioning: conditions on diagnosis, not on test result
  - Forces both test and disease to be binary
  - Confuses group performance with individual decision-making
  - Sensitivity is irrelevant once a test result is observed
- **19.2 Problems with ROC curves and cutoffs**:
  - AUC is useful for discrimination; ROC curve shape is not useful for decisions
  - Cutoff selection from ROC is incorrect: should come from utility function
  - Workup bias requires complex adjustment for sens/spec; not for probability models
- **19.3 Optimum individual decision making**: Bayesian decision rule uses Pr(disease|data)
  and utility function; does not use a fixed threshold
- **19.4 Diagnostic risk modeling**:
  - Logistic regression for Pr(disease | test results, patient characteristics)
  - Pre-test probability incorporated as a predictor (or via case-control sampling adjustment)
  - Multi-category disease: ordinal logistic model
- **19.5–19.6 Assessing diagnostic yield**:
  - Absolute yield: difference in posterior vs. prior probability
  - Relative yield: likelihood ratio, added AUC
- **19.7 Case-control designs**: sampling on outcome allows valid sens/spec estimation;
  logistic model intercept must be adjusted for sampling fraction

---

## Ch. 20 — High-Dimensional Data
https://hbiostat.org/bbr/hdata

Key sections:
- P >> N: overfitting is inevitable without heavy regularization
- **sec-hdata-oaat**: one-at-a-time bootstrap feature selection:
  - Bootstrap CI for importance rank of each feature separately
  - Reveals uncertainty in "important" features before claiming discovery
- **sec-hdata-rmatrix**: sample size to estimate a correlation matrix reliably:
  - Need N >> P; most genomic studies are woefully underpowered for correlation structure
- **sec-hdata-simor**: simulation to understand needed sample sizes for ORs in logistic models
- Regularization required: LASSO, ridge, elastic net; never plain stepwise
- Multiple testing: Bonferroni, FDR correction; but better to use Bayesian shrinkage
- Feature ranking bootstrap: wide CIs on importance ranks are the norm, not the exception

---

## Ch. 21 — Reproducible Research
https://hbiostat.org/bbr/repro

Key sections:
- **Decline effect**: effects shrink on replication; caused by:
  - Publication bias (only significant results published)
  - HARKing (hypothesizing after results known)
  - p-hacking / garden of forking paths
  - Regression to the mean on replication
- Quarto/R Markdown: embed analysis in document; never copy-paste results
- Pre-registration: OSF, ClinicalTrials.gov; separate confirmatory from exploratory
- Data and code archiving: Zenodo, GitHub, OSF
- Version control: git for all analysis code
- Reproducibility vs. replicability: same data same code (reproducibility) vs. new data (replicability)

---

## Ch. 22 — Controlling α vs. Probability of a Decision Error
https://hbiostat.org/bbr/alpha

This chapter is the philosophical capstone of BBR and connects directly to the **principles skill**.

Key points:
- α = Pr(assert effect | no effect truly exists) — NOT Pr(wrong | assert effect)
- The probability that a positive result turns out to be wrong requires:
  - Pr(H0 is true) in the population of tested hypotheses — unknown
  - Bayesian reasoning
- Fisher's error: "not more than 1% of such decisions will be wrong" — incorrect;
  he confused P(H0 | p < 0.01) with P(p < 0.01 | H0)
- Common misconception by highly experienced statisticians and trialists
- Bayesian posterior probabilities give exactly what decision-makers want:
  Pr(effect | data) — the probability a positive result will hold up
- "Spending α" / "controlling α" for sequential looks: makes sense for frequentist framework
  but is irrelevant for Bayesian analysis where each look updates the posterior
- Optimum α from decision analysis varies with disease severity and prior probability of effect
- See also: hbiostat.org/proj/covid19/statdesign; hbiostat.org/bayes/bet/evidence
