# Ordinal and Proportional Odds Models

Sources: https://hbiostat.org/ordinal · https://hbiostat.org/rmsc/ordinal · https://fharrell.com/post/rpo

## Table of Contents
1. [Core Idea](#core)
2. [Proportional Odds Model](#po)
3. [Fitting with lrm and orm](#fitting)
4. [Derived Quantities](#derived)
5. [Testing and Relaxing PO Assumption](#testing)
6. [Partial PO Model](#ppo)
7. [Continuous Y with orm](#continuous)
8. [Ordinal for Survival Analysis](#ordsurv)
9. [Longitudinal Ordinal Models](#longitudinal)
10. [Software Summary](#software)

---

## Core Idea {#core}

Semiparametric ordinal models extend the empirical CDF to incorporate covariates X:

- The **intercepts** encode the marginal Y distribution (ECDF on the link scale)
- The **β coefficients** describe how covariates shift the Y distribution
- **No distributional assumption** for Y: handles ties, floor/ceiling effects, bimodality
- **Invariant** to monotone transformations of Y
- Estimates effect ratios, Pr(Y ≥ y | X), quantiles of Y|X, and (if Y is interval-scaled) E(Y|X)

The ECDF is a complete sample summary for homogeneous Y. Ordinal regression generalizes it to adjust for X.

---

## Proportional Odds Model {#po}

The proportional odds (PO) model:

**Pr(Y ≥ y | X) = expit(αy + Xβ)**

- One intercept αy per distinct level of Y (minus one)
- Single β vector applies across all Y levels (parallelism on logit scale)
- Odds ratio = exp(β): the odds of Y ≥ y for a one-unit increase in X, **the same for every y**

**Example**: Y ∈ {0, 1, 2, 3}, X = sex and treatment:
```
Pr(Y ≥ y | X) = expit(αy + β1·male + β2·active)
α1=1, α2=0, α3=-1, β1=-0.5, β2=-0.4

Male on active treatment: β part = -0.5 + -0.4 = -0.9
Pr(Y≥1) = expit(1 - 0.9) = expit(0.1) = 0.52
Pr(Y≥2) = expit(0 - 0.9) = expit(-0.9) = 0.29
Pr(Y≥3) = expit(-1 - 0.9) = expit(-1.9) = 0.13
→ Pr(Y=2) = 0.29 - 0.13 = 0.16
```

**Connection to nonparametric tests:**
- The Wilcoxon/Mann-Whitney test is a special case of the PO model (two-group, no covariates)
- The Kruskal-Wallis test is a special case for k > 2 groups
- The log-rank test is a special case of the Cox PH model
- These rank tests actually assume *more* than their model counterparts

---

## Fitting with `lrm` and `orm` {#fitting}

```r
library(rms)
dd <- datadist(dt); options(datadist = "dd")

# Binary or discrete ordinal Y (≤250 distinct values)
f.lrm <- lrm(y ~ rcs(age,4) + treatment + sex,
             data = dt, x = TRUE, y = TRUE)

# Continuous or many-level ordinal Y (≤6000 distinct values, rms ≥7.0)
f.orm <- orm(y ~ rcs(age,4) + treatment + sex,
             data = dt, x = TRUE, y = TRUE)

# Available link functions for orm:
# "logistic" (proportional odds, default)
# "probit"   (proportional hazards on probit scale)
# "loglog"   (proportional hazards: complementary log-log)
# "cloglog"  (proportional hazards: log-log)
# "cauchit"
f.orm.ph <- orm(y ~ rcs(age,4) + treatment,
                family = cloglog, data = dt, x = TRUE, y = TRUE)
```

**Note on orm performance**: with n=300,000 and continuous Y (299,999 intercepts, 20 covariates), fits in ~2.5 seconds. Pieces of the covariance matrix computed in ~0.3 seconds. The order-restriction on intercepts (α1 ≤ α2 ≤ ...) allows estimating more intercept parameters than n.

---

## Derived Quantities from orm {#derived}

`orm` produces helper functions for deriving estimands:

```r
# Mean E(Y|X)
mfun <- Mean(f.orm)
p <- Predict(f.orm, age, treatment, fun = mfun)
ggplot(as.data.frame(p), aes(x = age, y = yhat, color = treatment)) +
  geom_line() + geom_ribbon(aes(ymin = lower, ymax = upper), alpha = 0.2)

# Median
medfun <- Quantile(f.orm, 0.5)
p <- Predict(f.orm, age, fun = medfun)

# 75th percentile
q75fun <- Quantile(f.orm, 0.75)

# Exceedance probability Pr(Y >= y | X) for specific y
exfun <- ExProb(f.orm)
p <- Predict(f.orm, age, fun = exfun(y = 3))  # Pr(Y >= 3 | age, others at median)
```

---

## Testing and Relaxing the PO Assumption {#testing}

The PO assumption states that the log-OR for any predictor is the same across all Y cutpoints. This can be tested but **violation is not fatal**:

- Simulation shows the PO log-OR is a well-estimated weighted average of the true cutpoint-specific log-ORs even under moderate non-proportionality
- The harm from assuming PO when it's slightly violated is small compared to the harm from dichotomizing Y or using a less efficient model
- Anyone who accepts a Wilcoxon test must accept the PO model (they assume the same thing)

**Graphical check** (pre-fitting):
```r
# Plot logit(F(y)) for each Y level vs. a continuous X — should be parallel
# Use orm() with a saturated categorical version of X
```

**Formal test** — score test in VGAM:
```r
library(VGAM)
f.vglm <- vglm(y ~ treatment + age, family = cumulative(parallel = FALSE), data = dt)
f.po   <- vglm(y ~ treatment + age, family = cumulative(parallel = TRUE),  data = dt)
lrtest(f.po, f.vglm)   # LR test for proportionality
```

---

## Partial PO Model {#ppo}

The partial PO (PPO) model allows selected covariates to have Y-level-dependent effects while keeping others proportional:

**VGAM approach** (frequentist, discrete Y, ≤50 levels):
```r
library(VGAM)
# Treatment is non-proportional; age is proportional
f.ppo <- vglm(y ~ age + treatment, data = dt,
              family = cumulative(parallel = FALSE ~ treatment, TRUE ~ age,
                                  reverse = TRUE))
```

**rmsb approach** (Bayesian, discrete Y, ≤200 levels):
```r
library(rmsb)
# Partial PO: allow treatment to have y-dependent OR
f.blrm <- blrm(y ~ rcs(age,4) + sex + treatment,
               ppo = ~ treatment,   # treatment gets non-PO parameterization
               data = dt)

# Constrained partial PO (CPPO): treatment OR varies monotonically with y
f.cblrm <- blrm(y ~ rcs(age,4) + sex + treatment,
                cppo = ~ treatment,
                data = dt)
```

---

## Continuous Y with `orm` {#continuous}

For truly continuous Y (no ties or many ties), `orm` with the logistic link is the **preferred model** over OLS when:
- Y is skewed (e.g., lab values, time measurements)
- Y has floor/ceiling effects
- You don't want to assume a distributional form for Y

Key advantages over OLS:
1. No normality assumption needed
2. Invariant to Y transformation — no need to find the "right" transformation
3. Automatically handles ties
4. Can produce mean, median, quantiles of Y|X as derived quantities
5. More efficient than OLS for non-normal Y

```r
# Continuous Y example (e.g., creatinine)
f <- orm(creatinine ~ rcs(age,4) + sex + diabetes,
         data = dt, x = TRUE, y = TRUE)
anova(f)

# Get mean creatinine by age and sex
mfun <- Mean(f)
p <- Predict(f, age, sex, fun = mfun)
ggplot(as.data.frame(p), aes(x = age, y = yhat, color = sex)) +
  geom_line() + geom_ribbon(aes(ymin = lower, ymax = upper, fill = sex), alpha = 0.2) +
  labs(x = "Age", y = "Mean creatinine (mg/dL)")
```

See RMS Ch. 15 for a detailed case study.

---

## Ordinal Semiparametric Survival Analysis {#ordsurv}

The Cox PH model is itself a semiparametric ordinal model on log-survival time using the complementary log-log link. The direct ordinal approach via `orm` on log(time) avoids the PH assumption entirely and handles ties without approximation.

See RMS Ch. 25:
```r
# Convert survival time to ordinal outcome (ignoring censoring is wrong;
# use orm on observed times in a more sophisticated censored-data setting,
# or use the approach in RMS Ch. 25)
```

For the full approach, see: https://hbiostat.org/rmsc/ordsurv

---

## Semiparametric Ordinal Longitudinal Models {#longitudinal}

A Markov-process approach for longitudinal ordinal data (RMS Ch. 22):
- Model transition probabilities Pr(Y(t) ≥ y | Y(t-1), X) using PO model
- Allows computation of state-occupancy probabilities over time
- Implemented via `blrm` with random effects (rmsb)

```r
library(rmsb)
# State occupancy model (each row = one time transition)
# y_next ~ y_current + treatment + rcs(age, 3) with random subject intercept
f.markov <- blrm(
  y_next ~ y_current + treatment + rcs(age, 3),
  data = dt_long,
  cluster = ~ subject_id
)
```

For detailed state-occupancy probability estimation, see https://hbiostat.org/rmsc/markov

---

## Software Summary {#software}

| Package | Function | Y type | k limit | Features |
|---|---|---|---|---|
| rms | `lrm` | discrete | 250 | logit link, efficient |
| rms | `orm` | continuous | 6000 | multiple links, Mean/Quantile/ExProb |
| rmsb | `blrm` | discrete/continuous | 300 | Bayesian PO/PPO/CPPO, random effects |
| VGAM | `vglm` | discrete | 50 | PPO, CPPO, many link functions |
| ordinal | `clm` | discrete | 50 | PPO, CPPO, random effects |
| brms | `brm` | discrete | varies | Bayesian, flexible |

**Key blog posts:**
- "Violation of proportional odds is not fatal" — https://fharrell.com/post/po
- "If you like the Wilcoxon test you must like the proportional odds model" — https://fharrell.com/post/wpo
- "Equivalence of Wilcoxon statistic and proportional odds model" — https://fharrell.com/post/powilcoxon
- "Assessing the proportional odds assumption and its impact" — https://fharrell.com/post/impactpo
- "Resources for ordinal regression models" — https://fharrell.com/post/rpo
- "Borrowing information across outcomes" (partial PO) — https://fharrell.com/post/yborrow
- "Information gain from ordinal vs binary outcomes" — https://fharrell.com/post/ordinal-info
- "Ordinal models for paired data" — https://fharrell.com/post/pair
