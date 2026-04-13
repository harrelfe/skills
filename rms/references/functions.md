# rms Function Reference

## Table of Contents
1. [Setup Functions](#setup)
2. [Fitting Functions](#fitting)
3. [Post-fit Description](#description)
4. [Prediction and Effects](#prediction)
5. [Validation Functions](#validation)
6. [Imputation (Hmisc)](#imputation)
7. [Penalization](#penalization)
8. [Utility / Helpers](#utility)

---

## Setup {#setup}

### `datadist()`
Computes predictor distribution summaries (range, quantiles, mode) needed by `summary()`, `Predict()`, `nomogram()`. **Must be set before fitting.**

```r
dd <- datadist(mydata)        # from data frame
dd <- datadist(x1, x2, x3)   # from individual vectors
options(datadist = "dd")      # register globally
```

Extend for new variables:
```r
dd <- datadist(dd, newvar)
```

### `rcs(x, k)` — Restricted Cubic Spline
Creates spline basis. `k` = number of knots (3–5 typical).

```r
rcs(age, 4)          # 4 knots at default quantile positions (5th,35th,65th,95th pctiles)
rcs(age, c(30,50,70)) # knots at specified values
rcs(age, 3)          # 2 df: linear outside boundary knots
rcs(age, 5)          # 4 df: for important predictors
```

### `spearman2()` (Hmisc) — Predictor Importance Ranking
Computes generalized Spearman ρ² between each predictor and Y *without* conditioning on other predictors. Use to allocate spline knots before fitting.

```r
s2 <- spearman2(y ~ age + sex + bili + albumin + creatinine, data = dt, p = 2)
plot(s2)   # horizontal bar chart of ρ² values
```

`p = 2` accounts for nonlinearity. Ranks predictors without using Y in a way that biases inference.

---

## Fitting Functions {#fitting}

All fitting functions share this interface pattern:
- Formula uses `rcs()`, `pol()`, `strat()`, `cluster()`, `offset()`, `interaction()`
- `x = TRUE, y = TRUE` stores design matrix and response — **required for `validate()`, `calibrate()`, `bootcov()`**
- `penalty` argument for penalized estimation

### `ols()` — OLS Linear Regression
```r
f <- ols(y ~ rcs(age,4) + sex + rcs(sbp,3), data = dt, x = TRUE, y = TRUE)
```

### `lrm()` — Binary or Discrete Ordinal Logistic Regression
```r
# Binary Y
f <- lrm(dead ~ rcs(age,4) + sex + rcs(bili,4), data = dt, x = TRUE, y = TRUE)
# Discrete ordinal Y (up to ~250 distinct values)
f <- lrm(severity ~ treatment + rcs(age,3), data = dt, x = TRUE, y = TRUE)
```
Key output: `Dxy` (Somers' D, = 2*(C - 0.5)), `C` (c-statistic/AUC), `R2` (Nagelkerke), `g` (Gini's mean difference of Xβ), `gp` (Gini's mean difference of Ŷ probabilities).

### `orm()` — Ordinal Regression for Continuous/Many-Level Y
Efficient for up to ~6000 distinct Y values. Multiple link functions.

```r
f <- orm(pain ~ rcs(age,4) + treatment, data = dt, x = TRUE, y = TRUE)
# Link functions: "logistic" (PO, default), "probit", "loglog", "cloglog", "cauchit"
f <- orm(pain ~ rcs(age,4) + treatment, family = probit, data = dt, x = TRUE, y = TRUE)
```

Derived quantities from `orm` fit:
```r
Mean(f)      # function to compute E(Y|X) from PO model
Quantile(f, 0.5)   # function to compute median Y|X
ExProb(f)    # function to compute P(Y >= y | X) for any y
```

### `cph()` — Cox Proportional Hazards
```r
f <- cph(Surv(time, event) ~ rcs(age,4) + sex + strat(clinic),
         data = dt, x = TRUE, y = TRUE, surv = TRUE)
# surv = TRUE stores survival curves; needed for Predict() with type = "survival"
```

`strat()` stratifies the baseline hazard without estimating a coefficient.
`cluster()` specifies robust sandwich SE for clustered data.

### `psm()` — Parametric Survival
```r
f <- psm(Surv(time, event) ~ rcs(age,4) + sex,
         data = dt, dist = "weibull")
# dist options: "weibull", "lognormal", "loglogistic", "exponential", "gaussian"
```

### `blrm()` (rmsb) — Bayesian Logistic/Ordinal Regression
```r
library(rmsb)
f <- blrm(y ~ rcs(age,4) + sex + treatment, data = dt)
# Partial PO (allow treatment to have y-dependent OR):
f <- blrm(y ~ rcs(age,4) + sex + treatment, ppo = ~ treatment, data = dt)
```

Uses Stan (RStan/cmdstanr). Provides exact credible intervals under penalization.

---

## Post-fit Description {#description}

### `print(fit)` / `fit`
Default print: coefficients, SEs, overall likelihood ratio test, C-statistic, R², g-index, calibration slope. For `orm`/`lrm` also prints score statistics and number of intercepts.

### `anova(fit)` — Chunk Tests ← **Always use this**
```r
a <- anova(f)
print(a)          # table: chi2, df, P-value, chi2-df for each predictor + interactions
plot(a)           # forest-style importance plot (chi2 - df per predictor)
```

For a predictor modeled with `rcs(x, 4)`, `anova()` gives:
- "Nonlinear" row testing the 2 extra df beyond linearity
- "x" row testing the overall 3-df chunk (linear + nonlinear)

### `summary(fit)` — Effect Summaries
IQR contrasts by default; customizable. Anti-log gives OR, HR, ratio:
```r
summary(f)
summary(f, age = c(40, 65), sex = "female")
plot(summary(f))   # forest plot of effects
```

---

## Prediction and Effects {#prediction}

### `Predict(fit, ...)` — Partial Effects (capital P)
Computes predicted Xβ (or transformed: `fun = plogis` for probabilities) while holding unspecified predictors at medians/modes.

```r
# Univariate
p <- Predict(f, age)
ggplot(p)

# By group
p <- Predict(f, age, sex)
ggplot(p)

# With probability scale for lrm
p <- Predict(f, age, fun = plogis)
ggplot(as.data.frame(p), aes(x = age, y = yhat)) +
  geom_line() + geom_ribbon(aes(ymin = lower, ymax = upper), alpha = 0.2) +
  labs(x = "Age", y = "Pr(Y=1)")

# Survival curves from cph
p <- Predict(f, age, sex, time = c(1, 3, 5))

# Mean Y from orm (uses Mean(f) internally)
p <- Predict(f, age, fun = Mean(f))
```

### `contrast(fit)` — Contrasts
```r
# Single difference: age 60 vs 40, male
contrast(f, list(age = 60, sex = "male"), list(age = 40, sex = "male"))

# Double difference (interaction contrast): does age effect differ by sex?
contrast(f,
         list(age = 60, sex = "male"),   list(age = 40, sex = "male"),
         list(age = 60, sex = "female"), list(age = 40, sex = "female"))

# Grid of contrasts for confidence band
contrast(f,
         list(age = 40:70, sex = "male"),
         list(age = 40,    sex = "male"))
```

### `nomogram(fit)` — Nomogram
```r
nom <- nomogram(f,
                fun      = list(plogis),
                funlabel = list("Prob(outcome)"),
                lp       = FALSE)     # omit linear predictor axis
plot(nom)
```

For survival models:
```r
nom <- nomogram(f, fun = list(function(lp) 1 - exp(-exp(lp))),
                funlabel = "1-yr survival")
```

---

## Validation Functions {#validation}

### `validate(fit, B = 200)` — Bootstrap Internal Validation
```r
set.seed(17)
v <- validate(f, B = 200)
# Output: Dxy, R2, Intercept (calibration), Slope (calibration), g, gp, Brier
# Columns: training, test, optimism, corrected, n

# If stepdown selection was used in fitting, include bw = TRUE
v <- validate(f, B = 200, bw = TRUE)
```

For survival: adds Dxy and R² with censoring adjustments.

### `calibrate(fit, B = 200)` — Calibration Curve
```r
cal <- calibrate(f, B = 200)
plot(cal)
# Plots: apparent calibration, bias-corrected, ideal diagonal
# Use loess = TRUE for smooth curve (default for lrm/orm)
```

### `bootcov(fit, B = 200)` — Bootstrap Covariance Matrix
Bootstraps the covariance matrix of coefficients (useful for derived quantities):
```r
f.bc <- bootcov(f, B = 200)
summary(f.bc)   # bootstrap CIs for all effects
```

---

## Imputation (Hmisc) {#imputation}

### `aregImpute()` — Multiple Imputation
Uses adaptive regression (additive) with bootstrap for each imputation. Handles all variable types.

```r
imp <- aregImpute(
  ~ age + sex + bili + albumin + creatinine + y,
  data     = dt,
  n.impute = 10,   # 10 imputed datasets (use 20+ for publication)
  nk       = 4,    # spline knots in imputation models
  tlinear  = FALSE # allow nonlinear imputation for binary/ordinal vars
)
```

### `fit.mult.impute()` — Fit Model Across Imputed Datasets
```r
f <- fit.mult.impute(
  y ~ rcs(age,4) + sex + rcs(bili,4) + albumin,
  fitter = lrm,
  xtrans = imp,
  data   = dt,
  x      = TRUE,
  y      = TRUE
)
```

Rubin's rules for combining are applied automatically. `validate()` on the result bootstraps the entire impute-then-fit process.

---

## Penalization {#penalization}

### `pentrace()` — Choose Penalty Parameter
```r
p <- pentrace(f, penalty = c(0, 0.5, 1, 5, 10, 25, 50, 100, 200))
plot(p)                # plot effective df vs log penalty
p$penalty              # optimal penalty (minimizes AIC by default)
```

### Fitting with Penalty
```r
f.pen <- update(f, penalty = 10)
# Or at fit time:
f.pen <- lrm(y ~ rcs(age,4) + sex + rcs(bili,4), data = dt,
             penalty = list(simple = 10, nonlinear = 20),
             x = TRUE, y = TRUE)
```

`list(simple, nonlinear)` allows heavier penalization of nonlinear terms than linear terms.

---

## Utility / Helpers {#utility}

### `describe()` (Hmisc) — Data Description
```r
describe(dt)   # n, missing, distinct values, mean, quantiles, extremes for all vars
```

### `varclus()` (Hmisc) — Variable Clustering
Identifies redundant predictors via hierarchical clustering of Spearman correlations:
```r
vc <- varclus(~ age + sex + bili + albumin, data = dt)
plot(vc)
```

### `redun()` (Hmisc) — Redundancy Analysis
Identifies which variables are predictable from others (R² > threshold):
```r
r <- redun(~ age + sex + bili + albumin + creatinine, data = dt, r2 = 0.9)
r$Out   # variables that are redundant
```

### `rcorr()` (Hmisc) — Rank Correlation Matrix
```r
rcorr(as.matrix(dt[, .(age, bili, albumin)]), type = "spearman")
```

### `xYplot()` / `ggplot(Predict(...))` — Plotting
Prefer `ggplot2` with `as.data.frame(Predict(...))` for full control:
```r
p <- Predict(f, age, sex, fun = plogis)
ggplot(as.data.frame(p), aes(x = age, y = yhat, color = sex, fill = sex)) +
  geom_line() +
  geom_ribbon(aes(ymin = lower, ymax = upper), alpha = 0.2) +
  scale_color_manual(values = c("male" = "#2166ac", "female" = "#d6604d")) +
  labs(x = "Age (years)", y = "Predicted probability",
       color = "Sex", fill = "Sex") +
  theme_bw()
```

### `latex(fit)` / `html(fit)` — Formatted Output
```r
latex(anova(f))     # LaTeX table
html(summary(f))    # HTML table (for R Markdown / Quarto)
```

### `Function(fit)` — Extract Prediction Function
Returns a standalone R function that computes predictions:
```r
pfun <- Function(f)   # e.g. for logistic: returns function(age, sex, ...) -> log-odds
pfun(age = 55, sex = "male")
```

---

## Key rms Print Output Explained

For `lrm` / `orm` output:
```
Obs  Events  ...          # sample size, events
Dxy   C     R2   g  gp   Brier
0.42  0.71  0.22  1.4  0.11  0.18
```
- `Dxy` = Somers' D = 2*(C − 0.5); rank correlation of predicted vs observed
- `C` = c-statistic (AUC equivalent) = (Dxy + 1)/2
- `R2` = Nagelkerke R²
- `g` = Gini's mean difference of Xβ̂ (typical absolute difference in log-odds)
- `gp` = Gini's mean difference of predicted probabilities (typical abs risk difference)
- `Brier` = mean squared prediction error

For `cph`:
- `Dxy` and `C` are censoring-adjusted (Harrell's C-statistic)
- `R2` is a censored-data analog
