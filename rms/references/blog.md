# Frank Harrell Blog Posts — Statistical Modeling Reference

Source: https://fharrell.com/post (Statistical Thinking)

Curated posts relevant to regression modeling, validation, ordinal models, and ML vs. statistics.

---

## Model Validation

### Split-Sample Model Validation (2017)
**URL**: https://fharrell.com/post/split-val/
**Key points:**
- Data splitting is unstable: re-splitting gives substantially different results unless n > ~20,000
- Splitting validates one example model; resampling validates the *process* (including variable selection)
- Bootstrap requires ~5–10× less data than splitting to achieve the same precision
- "Re-do" temptation after poor test-set performance is common and leads to unreported attempts
- Users who recombine after splitting have no validation of the combined-data model
- **Recommendation**: always use bootstrap `validate()` in rms. Repeated 10-fold CV (100 repeats) is roughly equivalent to the bootstrap but slower.

### Bootstrap Confidence Limits for Overfitting-Corrected Performance (2025)
**URL**: https://fharrell.com/post/bootcal/
**Key points:**
- The Efron-Gong optimism bootstrap estimates bias from overfitting and subtracts it from apparent performance
- No reliable CI for overfitting-corrected measures existed until this work
- Quadratic calibration is a good compromise between linear and loess for CI computation
- Applies to binary logistic regression; expected to generalize
- Implemented incrementally in `validate()` and `calibrate()`

### Biostatistical Modeling Plan (modplan)
**URL**: https://fharrell.com/post/modplan/
**Key points:**
- Template statistical analysis plan for prediction model development
- Pre-specifies: outcome, predictors, spline knots (guided by spearman2), imputation approach
- Uses `aregImpute` + `fit.mult.impute` + bootstrap `validate()`
- For continuous non-normal Y: uses `orm` (PO model) rather than OLS
- Explicitly rejects split-sample validation; uses bootstrap
- Summarizes EPV rationale: logistic regression needs ~20 EPV; ML needs ~200 EPV

---

## Ordinal Models

### Violation of Proportional Odds is Not Fatal (2020)
**URL**: https://fharrell.com/post/po/
**Key points:**
- The Wilcoxon test involves U statistics (concordance probability c)
- Somers' Dyx = 2*(c - 0.5): the rank correlation corresponding to the Wilcoxon test
- Even under non-proportional odds, the PO log-OR is a well-estimated weighted average
- Simulation shows PO model harm from assumption violation is small
- Stephen Senn: anyone accepting either dichotomy of a 3-point scale should accept the PO model's compromise

### If You Like the Wilcoxon Test You Must Like the Proportional Odds Model (wpo)
**URL**: https://fharrell.com/post/wpo/
**Key points:**
- Wilcoxon statistic and PO log-OR have a 1-1 relationship for two-group comparisons
- The PO model strictly generalizes the Wilcoxon test (handles covariates, interactions, arbitrary Y ties)
- Accepting Wilcoxon = accepting proportional odds as the underlying model

### Equivalence of Wilcoxon Statistic and Proportional Odds Model (powilcoxon)
**URL**: https://fharrell.com/post/powilcoxon/
**Key points:**
- Detailed simulations with derivation of the Wilcoxon statistic from the PO assumption
- Quantifies amount of non-PO and shows it relates weakly to approximation error

### Assessing the Proportional Odds Assumption and Its Impact (impactpo)
**URL**: https://fharrell.com/post/impactpo/
**Key points:**
- Graphical and formal methods for checking PO
- Simulation of consequences when PO is violated
- PDF and R script available

### Resources for Ordinal Regression Models (rpo)
**URL**: https://fharrell.com/post/rpo/ (hub page)
**Key points:**
- Comprehensive index of all ordinal regression resources
- Covers `lrm`, `orm`, `blrm` (rmsb), VGAM, ordinal, brms packages
- Links to: PPO/CPPO models, Bayesian approaches, clinical trial design, software comparison
- Semiparametric models encode ECDF in intercepts; β shifts the distribution
- PO assumption can be relaxed via partial PO (VGAM `vglm`, rmsb `blrm` with `ppo =`)

### Borrowing Information Across Outcomes (yborrow, 2024)
**URL**: https://fharrell.com/post/yborrow/
**Key points:**
- Ordinal outcomes in RCTs borrow treatment effect information across all Y levels → single OR
- Partial PO model bridges between standalone mortality evidence and ordinal composite
- When death is included as worst ordinal category, model can provide mortality-specific inference
- Relevant for COVID-19 ordinal scale studies and similar

### Information Gain from Using Ordinal Instead of Binary Outcomes
**URL**: https://fharrell.com/post/ordinal-info/
**Key points:**
- Ordinal outcomes always provide at least as much information as dichotomized binary outcomes
- Power gains from using full ordinal scale vs. any single binary cutpoint

### Ordinal Models for Paired Data (pair, 2023)
**URL**: https://fharrell.com/post/pair/
**Key points:**
- Rank-difference test > Wilcoxon signed-rank for paired data
- PO model with robust cluster sandwich SE generalizes rank-difference test
- Preserves α = 0.05 under null; power ≥ paired t-test (exceeds for non-normal data)
- Generalizes to longitudinal (>2 periods) and covariate adjustment

---

## Machine Learning vs. Statistical Modeling

### In ML Predictions for Health Care the Confusion Matrix Is a Matrix of Confusion (mlconfusion, 2018)
**URL**: https://fharrell.com/post/mlconfusion/
**Key points:**
- Sensitivity/specificity condition on the unknown truth — wrong conditioning for prediction
- AUROC measures discrimination but not calibration
- Calibration is crucial for medical decision-making and largely overlooked in ML
- Classification (forced-choice) vs. probability modeling: the latter addresses the real problem
- ML claims of improved discrimination are often based on uncalibrated models

### Classification vs. Prediction
**URL**: https://fharrell.com/post/classification/
**Key points:**
- Classification = premature forced binary decision; discards probabilistic information
- Threshold for action should come from decision analysis, not from the modeling step
- Probability models are strictly more useful than classifiers: a classifier is derived from a probability model but not vice versa

---

## Statistical Errors and Best Practices

### Statistical Errors in the Medical Literature (errmed)
**URL**: https://fharrell.com/post/errmed/
**Key points:**
- **Change from baseline**: wrong. Use ANCOVA (baseline as covariate). Requires baseline linearly related to follow-up, same transformation, no floor/ceiling effects, slope ≈ 1.
- **Regression to the mean**: strong when baseline used as inclusion criterion
- **Proportional odds model** preferred over OLS for ordinal/continuous non-normal Y: no transformation assumption needed
- Detailed discussion of conditions under which change scores are invalid

### Biomarker Research Bad Practices
**URL**: https://fharrell.com/post/addvalue/ (related)
**Key points:**
- Bar for "new information" from biomarker should account for existing predictors
- Continuous biomarker should not be categorized
- Bootstrap must validate entire selection process

---

## Bayesian Methods

### Traditional Frequentist Inference Uses Unrealistic Priors (uprior, 2024)
**URL**: https://fharrell.com/post/uprior/
**Key points:**
- Standard frequentist inference corresponds to a flat prior: implies θ is equally likely to be enormous as small
- Non-informative priors are often unrealistic; moderate regularizing priors are more defensible
- Bayesian framework provides full exact inference even under penalization (unlike frequentist penalized MLE)

---

## Computational

### Statistical Computing Approaches to Maximum Likelihood Estimation (mle, 2024)
**URL**: https://fharrell.com/post/mle/
**Key points:**
- MLE requires iterative optimization; multiple algorithms compared
- Pre-processing (centering, orthogonalization) affects convergence
- R vs. Fortran comparisons for `lrm` rewrite
- `orm` can fit n=300,000 with 299,999 intercepts in ~2.5 seconds

### Bootstrap Confidence Limits (bootcal, 2025)
**URL**: https://fharrell.com/post/bootcal/
- See validation section above

### Measures of Central Tendency for Asymmetric Distributions (aci, 2025)
**URL**: https://fharrell.com/post/aci/
**Key points:**
- Three measures: mean, median, pseudomedian (Hodges-Lehmann estimator)
- For very skewed distributions, CLT fails for the mean even at large n
- BCa bootstrap best for mean CIs but still unreliable at n=200 for lognormal
- **Pseudomedian wins**: robust + efficient + accurate CI from three methods
- Recommends pseudomedian as primary location measure for asymmetric distributions

---

## Causal Inference

### Causal by Design (2026)
**URL**: https://fharrell.com/post/causal/
**Key points:**
- Well-controlled RCTs analyzed in a randomization-respecting way are causal by design
- No causal calculus needed for properly randomized experiments
- Retrospective observational studies cannot achieve causal inference through target trial emulation alone
- Prospective design is required for observational studies to provide reliable causal evidence

---

## Quick Reference: When to Use Which Blog Post

| Question | Blog post |
|---|---|
| Should I split my data for validation? | split-val |
| Is the PO assumption required? | po, impactpo |
| Why not use Wilcoxon instead of lrm? | wpo, powilcoxon |
| How to handle ordinal outcome in RCT? | rpo, yborrow |
| ML vs. regression: which to use? | mlconfusion, genreg (Ch. 2.5) |
| Change from baseline analysis? | errmed |
| Bayesian vs. frequentist? | uprior |
| orm performance and MLE internals? | mle |
| Location measure for skewed Y? | aci |
| Template analysis plan? | modplan |
