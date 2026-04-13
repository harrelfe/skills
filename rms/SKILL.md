---
name: rms
description: >
  Use this skill for ANY question about multivariable regression modeling or the R rms
  package (ols, lrm, orm, cph, psm, blrm, rcs, datadist, anova, validate, calibrate,
  nomogram, contrast, Predict). Also trigger for: restricted cubic splines, degrees of
  freedom allocation, overfitting, bootstrap validation, penalized estimation, shrinkage,
  multiple imputation, binary/ordinal logistic regression, proportional odds model, Cox
  regression, parametric survival models, generalized least squares, semiparametric
  ordinal longitudinal models, partial effect plots, nomograms, machine learning vs.
  statistical models, Frank Harrell's modeling philosophy, safe data mining, spending
  degrees of freedom, effective sample size, chunk tests, phantom degrees of freedom,
  chi-squared-minus-df signal, or any question about rms, rmsb, or Hmisc packages.
  Always use this skill — even for basic rms questions.
---

# Statistical Modeling with `rms` — Frank Harrell's Framework

Primary sources: [RMS course notes](https://hbiostat.org/rmsc) · [Ordinal regression](https://hbiostat.org/ordinal) · [Statistical Thinking blog](https://fharrell.com/post)

For deep dives see `references/` subfolder:
- `references/functions.md` — rms function reference with usage patterns
- `references/ordinal.md`   — proportional odds / ordinal model detail
- `references/blog.md`      — curated blog post summaries

---

## Core Philosophy

### The Modeling Mandate
A model's purpose is to **make accurate predictions for future observations** or to produce **honest evidence about the data-generating mechanism**. Every modeling decision should serve one of those goals. Regression models are the best multivariable descriptive statistics; they isolate effects, quantify uncertainty, and adjust for covariates in ways that stratification cannot.

### Phantom Degrees of Freedom
Any examination of the data using Y that influences model specification — looking at scatterplots, running preliminary tests, checking variable significance — consumes degrees of freedom. If those df are not later accounted for in inference, they become **phantom df**: invisible but still haunting the analysis as overconfidence. Pre-specify model complexity before touching Y.

### Spending Degrees of Freedom
Given an information budget determined by effective sample size (ESS), decide:
1. **How many df can be spent** — governed by ESS (see below)
2. **Where to spend them** — allocate more df to predictors with greater potential predictive punch
3. **Spend them with no regrets** — once a predictor is in, keep it; don't drop it after looking

Use Spearman's ρ² (`spearman2()` in Hmisc) or generalized rank correlations to rank predictors' marginal importance *without* using Y in a way that biases inference.

### Effective Sample Size
| Outcome type | ESS rule of thumb |
|---|---|
| Continuous Y (OLS) | n / (df in model) ≥ 10–20 |
| Binary Y (LRM) | min(events, non-events) / df ≥ 10–20 |
| Survival (CPH) | events / df ≥ 10–20 |
| Ordinal Y (ORM/LRM) | uses full n more efficiently than binary |

**Implication**: if ESS is marginal, use fewer df (fewer spline knots, collapse categories) or apply penalized estimation.

### χ² − df: The Signal Measure
`anova(fit)` reports Wald χ² statistics. The net signal for a term is **χ² − df** (the expected χ² under the null equals df). This chance-corrects for the number of parameters, making it a fair basis for comparing predictors of different complexity. Use it to rank predictors and decide how to reallocate df.

### Chunk Tests, Not Individual Parameter Tests
Never judge a predictor by any single regression coefficient or p-value. Use **chunk tests** (joint tests of all parameters associated with a predictor, including nonlinear and interaction terms). `anova(fit)` provides these automatically for every predictor and interaction in the model.

---

## Pre-Specified Model Complexity

1. **Never categorize continuous predictors.** Categorization discards information, creates arbitrary thresholds, and introduces phantom df.
2. **Use restricted cubic splines (RCS)** to allow nonlinearity while controlling df:
   - 3 knots → 2 df (linear outside boundary knots, one interior bend)
   - 4 knots → 3 df (default for moderately important predictors)
   - 5 knots → 4 df (for clearly dominant predictors)
   - Default knot positions: quantiles of the predictor distribution
3. **Allocate knots proportional to importance**: use `spearman2()` rankings to guide which predictors get 5-knot vs 3-knot splines.
4. **Never use stepwise selection.** It inflates type I error, produces biased estimates, generates phantom df, and resampling cannot rescue it. Either pre-specify the full model or use penalization.
5. **Interactions**: pre-specify based on subject-matter knowledge. Test with chunk test (joint Wald test of all interaction parameters).

---

## Key R Packages and Setup

```r
library(rms)       # core modeling: ols, lrm, orm, cph, psm, blrm, etc.
library(Hmisc)     # data description, spearman2, aregImpute, varclus
library(rmsb)      # Bayesian rms: blrm (Bayesian LRM / ordinal)
library(data.table)
library(ggplot2)
library(nlme)      # gls() for longitudinal GLS models
library(VGAM)      # vglm() for partial PO and many link functions
```

### `datadist` — Required Setup
`rms` functions need a `datadist` object to compute default predictor ranges for summaries, plots, and Predict:

```r
dd <- datadist(mydata)    # or datadist(x1, x2, x3, ...)
options(datadist = "dd")
```

Always set this before fitting. Re-run after subsetting.

---

## Fitting Models

### `ols` — Ordinary Least Squares
```r
f <- ols(y ~ rcs(age, 4) + sex + rcs(sbp, 3), data = dt, x = TRUE, y = TRUE)
```

### `lrm` — Logistic Regression (binary or discrete ordinal Y, ≤250 levels)
```r
f <- lrm(dead ~ rcs(age, 4) + sex + rcs(bili, 4), data = dt, x = TRUE, y = TRUE)
```

### `orm` — Ordinal Regression (continuous or heavily-tied Y, ≤6000 levels)
```r
f <- orm(pain_score ~ rcs(age, 4) + treatment, data = dt, x = TRUE, y = TRUE)
# Supports link = "logistic" (PO), "probit", "loglog", "cloglog", "cauchit"
```

### `cph` — Cox Proportional Hazards
```r
f <- cph(Surv(time, event) ~ rcs(age, 4) + sex + rcs(bili, 4),
         data = dt, x = TRUE, y = TRUE, surv = TRUE)
```

### `psm` — Parametric Survival Model
```r
f <- psm(Surv(time, event) ~ rcs(age, 4) + sex,
         data = dt, dist = "weibull")  # also "lognormal", "loglogistic"
```

### `blrm` — Bayesian Logistic/Ordinal Regression (rmsb)
```r
f <- blrm(y ~ rcs(age, 4) + treatment, data = dt)  # uses Stan
```

---

## Penalization and Shrinkage

When ESS is limited, use penalized maximum likelihood via `pentrace()`:

```r
p <- pentrace(f, penalty = c(0, 1, 5, 10, 25, 50, 100))
plot(p)                    # plot effective df vs penalty
f.pen <- update(f, penalty = p$penalty)
```

Penalization shrinks all coefficients toward zero. Bayesian `blrm` achieves the same effect through priors and provides exact inference under penalization.

---

## Describing Model Fit

### `anova(fit)` — Chunk Tests
```r
anova(f)  # joint Wald tests for every predictor + interactions
          # reports chi2, df, P, and chi2 - df (signal measure)
```

### `summary(fit)` — Effect Estimates
Reports inter-quartile range (IQR) contrasts by default. Anti-logs give odds ratios, hazard ratios, etc.
```r
summary(f)
summary(f, age = c(40, 60), sex = "male")  # custom reference values
```

### `Predict(fit)` — Predictions for Plotting (capital P)
Returns a data frame suitable for ggplot2. Works for any rms model.

```r
p <- Predict(f, age, sex)   # partial effect: vary age by sex, others at median/mode
ggplot(p)                   # default rms plot via ggplot method
# or manual ggplot2:
ggplot(as.data.frame(p), aes(x = age, y = yhat, color = sex)) +
  geom_line() + geom_ribbon(aes(ymin = lower, ymax = upper, fill = sex), alpha = 0.2)
```

### `contrast(fit)` — Contrast Estimation
```r
contrast(f,
         list(age = 60, sex = "male"),
         list(age = 40, sex = "male"))  # difference + SE + CI
# Double-difference (interaction contrast):
contrast(f,
         list(age = 60, sex = "male"),   list(age = 40, sex = "male"),
         list(age = 60, sex = "female"), list(age = 40, sex = "female"))
```

### `nomogram(fit)` — Nomogram
```r
nom <- nomogram(f, fun = plogis, funlabel = "Prob(Y=1)")
plot(nom)
```

---

## Model Validation (Bootstrap-First Philosophy)

**Do not split data.** Data splitting wastes information, produces unstable estimates (results change on re-split), and validates only one example model rather than the modeling process. The bootstrap validates the entire process — including any variable selection — by repeating it on each resample.

### `validate(fit)` — Internal Validation
Estimates optimism-corrected discrimination (Dxy, C-statistic) and calibration slope:

```r
set.seed(17)
v <- validate(f, B = 200)   # 200 bootstrap resamples
v
```

For models with backward step-down selection, include `bw = TRUE` so variable selection is repeated in each bootstrap:
```r
v <- validate(f, B = 200, bw = TRUE)
```

### `calibrate(fit)` — Calibration Curve
```r
cal <- calibrate(f, B = 200)
plot(cal)
```

### Cross-validation
100 repeats of 10-fold CV is roughly equivalent to the bootstrap. Use `method = "crossvalidation"` in `validate()` if preferred, but bootstrap is generally more efficient.

---

## Missing Data and Multiple Imputation

**Never drop incomplete cases.** Use multiple imputation (MI) via `aregImpute()` (Hmisc), which imputes using flexible additive regression with bootstrap resampling.

```r
# Step 1: Impute
imp <- aregImpute(~ age + sex + bili + albumin + y, data = dt, n.impute = 5, nk = 4)

# Step 2: Fit with fit.mult.impute
f.mi <- fit.mult.impute(y ~ rcs(age,4) + sex + rcs(bili,4) + albumin,
                        lrm, imp, data = dt, x = TRUE, y = TRUE)
# Step 3: Validate (bootstrap includes imputation in each resample)
v <- validate(f.mi, B = 200)
```

MI and bootstrap interact correctly: imputation is done from scratch on each bootstrap resample when using `validate()` with `fit.mult.impute` objects.

---

## Model Types by Outcome

| Outcome | Recommended model | rms function |
|---|---|---|
| Continuous, roughly normal | OLS | `ols` |
| Continuous, non-normal / heavily tied | Semiparametric ordinal | `orm` |
| Binary | Logistic regression | `lrm` |
| Ordinal discrete (few levels) | Proportional odds | `lrm` |
| Ordinal continuous (many levels) | PO with many intercepts | `orm` |
| Time-to-event, flexible | Cox PH | `cph` |
| Time-to-event, parametric | Weibull/lognormal | `psm` |
| Longitudinal continuous | Generalized least squares | `gls` (nlme) + rms wrappers |
| Longitudinal ordinal | Markov/CPPO | `blrm` with random effects |
| Bayesian ordinal | Bayesian PO/CPPO | `blrm` (rmsb) |

**Prefer semiparametric ordinal models** (`orm`/`lrm`) for non-normal continuous Y. They require no distributional assumption beyond the parallel-shift (proportional odds) assumption, are invariant to monotone transformation of Y, and efficiently handle arbitrary ties, floor/ceiling effects, and bimodality. For n=300,000 with continuous Y, `orm` fits in ~2.5 seconds.

---

## Ordinal / Proportional Odds Model

Full detail → `references/ordinal.md`

Key points:
- `Pr(Y ≥ y | X) = expit(αy + Xβ)` — one intercept per Y level, shared β
- The Wilcoxon/Kruskal-Wallis test is a special case of the PO model
- PO assumption violation is **not fatal**: simulation shows the log-OR is well-estimated even under moderate non-proportionality
- Relaxation: partial PO model (VGAM `vglm` or rmsb `blrm`) allows some covariates to have Y-dependent effects
- `orm` handles up to ~6000 distinct Y values efficiently; `lrm` up to ~250

---

## Survival Analysis

```r
# Cox PH with nonlinear age and test PH assumption
f <- cph(Surv(time, event) ~ rcs(age,4) + sex + strat(center),
         data = dt, x = TRUE, y = TRUE, surv = TRUE)
anova(f)               # chunk tests for each predictor
cox.zph(f)             # test proportional hazards assumption
Predict(f, age, sex)   # partial effect plots

# Parametric survival - use when PH assumption fails or extrapolation needed
f.psm <- psm(Surv(time, event) ~ rcs(age,4) + sex, data = dt, dist = "lognormal")
```

Ordinal semiparametric survival (`orm` on log-time) is an alternative that avoids the PH assumption entirely. See `references/ordinal.md`.

---

## Longitudinal Data

### Generalized Least Squares (continuous Y)
```r
library(nlme)
f.gls <- gls(y ~ time * treatment + rcs(age,3),
             correlation = corCAR1(form = ~time | id),
             weights     = varIdent(form = ~1 | time),
             data        = dt_long)
```

### Semiparametric Ordinal Longitudinal (Markov model)
Uses PO model on each transition; fits state-occupancy probability over time. See RMS Ch. 22 and `references/ordinal.md`.

---

## Machine Learning vs. Statistical Models

Use statistical regression when:
- Additive effects predominate (regression beats ML with ~20 EPV vs ML needing ~200)
- You need interpretable, separable predictor effects
- You need confidence intervals, formal hypothesis tests, calibrated probabilities
- Sample size is moderate (regression works; ML overfits)

Use ML when:
- Complex high-order interactions are genuinely expected
- N is very large (100k+)
- Black-box predictions are acceptable (no need to understand mechanism)
- Problem is image/text/sequence (deep learning is purpose-built)

Key references: Harrell RMS Ch. 2.5; van der Ploeg et al. (2014) on EPV; Kapoor & Narayanan (2023) on inflated ML claims.

---

## Workflow Template

```r
library(data.table); library(rms); library(Hmisc); library(ggplot2)

# 1. Load and describe
dt <- fread("data.csv")
describe(dt)   # Hmisc: distribution summaries, missing, distinct values

# 2. Rank predictors by marginal importance (before looking at model)
s2 <- spearman2(y ~ age + sex + bili + albumin, data = dt, p = 2)
plot(s2)   # guides knot allocation

# 3. Set datadist
dd <- datadist(dt);  options(datadist = "dd")

# 4. Multiple imputation (if needed)
imp <- aregImpute(~ age + sex + bili + albumin + y, data = dt, n.impute = 10, nk = 4)

# 5. Pre-specify and fit model
f <- fit.mult.impute(
  y ~ rcs(age,4) + sex + rcs(bili,4) + albumin,
  lrm, imp, data = dt, x = TRUE, y = TRUE
)

# 6. Chunk tests
anova(f)

# 7. Partial effect plots
p <- Predict(f, age, fun = plogis)
ggplot(as.data.frame(p), aes(x = age, y = yhat)) +
  geom_line() + geom_ribbon(aes(ymin = lower, ymax = upper), alpha = 0.2) +
  labs(x = "Age", y = "Predicted probability", title = "Partial effect of age")

# 8. Bootstrap validation
set.seed(17)
v <- validate(f, B = 200)
print(v, digits = 3)

# 9. Calibration curve
cal <- calibrate(f, B = 200)
plot(cal)

# 10. Nomogram (optional)
nom <- nomogram(f, fun = plogis, funlabel = "P(Y=1)")
plot(nom)
```

---

## Common Pitfalls and Corrections

| Pitfall | Harrell's correction |
|---|---|
| Splitting data for validation | Use bootstrap `validate()` |
| Stepwise selection | Pre-specify or penalize |
| Categorizing continuous predictors | Use `rcs()` |
| Reporting individual spline term p-values | Use `anova()` chunk tests |
| Analyzing change from baseline | Use ANCOVA with baseline as covariate |
| Using linear model for non-normal continuous Y | Use `orm()` (semiparametric PO) |
| Claiming ML beats regression without EPV adjustment | Check EPV; regression is competitive at moderate N |
| Data splitting with small N | Bootstrap — splitting requires ~5-10× more data |
| Dropping cases with missing data | `aregImpute()` + `fit.mult.impute()` |

---

## References

- Harrell FE (2015). *Regression Modeling Strategies*, 2nd ed. Springer. → https://hbiostat.org/rmsc
- Harrell FE. Ordinal regression resources → https://hbiostat.org/ordinal
- Statistical Thinking blog → https://fharrell.com/post
- rms package docs → https://hbiostat.org/r/rms
- rmsb package (Bayesian) → https://hbiostat.org/r/rmsb
