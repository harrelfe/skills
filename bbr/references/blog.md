# BBR-Relevant Blog Posts — Statistical Thinking

Source: https://fharrell.com/post

Curated posts relevant to BBR topics: inference, design, diagnosis, measurement,
change scores, dichotomization, and related errors.

---

## Statistical Inference and p-values

### A Litany of Problems With p-values (2017)
**URL**: https://fharrell.com/post/pval-litany
**Key points:**
- Comprehensive catalog of p-value failures organized by category
- A: Conditioning problems — conditions on H0 (unknown), not on data (known)
- B: Indirectness — p-values are indirect evidence metrics; not calibrated for decision-making
- C: Definitional problems — in continuous data, Pr(exactly this result) = 0
- D: Computing problems — require knowing exact stopping rule, intention to test, look schedule
- E: Multiplicity mess — requires knowing all tests performed, even unplanned
- F: Non-trivial hypotheses — cannot test composite assertions cleanly
- G: Cannot incorporate prior context
- H/I: Misinterpretation of both "positive" and "negative" findings
- Recommends: Bayesian and likelihood paradigms should replace frequentist inference

### Statistical Errors in the Medical Literature (2017, updated)
**URL**: https://fharrell.com/post/errmed
**Key points (BBR-relevant sections):**
- **P-value misinterpretation**: large p ≠ no effect; "not significantly different" is meaningless
  - Correct wording: "rules out, with 95% confidence, reduction > X"
  - Bayesian solution: report Pr(any benefit), Pr(benefit > threshold), Pr(similarity)
- **Dichotomania**: continuous variables forced binary lose power; CAST trial example
- **Change from baseline**: requires slope ≈ 1, linear relationship, no floor/ceiling — rarely holds;
  ANCOVA is always better; KCCQ ceiling effect example
- **Improper subgrouping**: separate subgroup p-values are wrong;
  must test interaction formally in whole-group model
- **Serial data**: response trajectory analysis preferred over single change score or final value
- **Cluster analysis**: validity must be demonstrated statistically; clusters often spurious

---

## Dichotomization and Information Loss

### Dichotomania (referenced in errmed)
**URL**: fharrell.com/post/errmed#dichotomania (section within errmed)
**Key points:**
- Creating binary variables from continuous ones loses information equivalent to discarding
  many subjects
- Odds ratios from dichotomized predictors understate true associations
- Artificial thresholds vary across studies, preventing comparability
- Solution: restricted cubic splines for continuous predictors; ordinal/continuous outcomes

---

## Covariate Adjustment and ANCOVA

### Unadjusted Odds Ratios are Conditional (2020)
**URL**: https://fharrell.com/post/marg
**Key points:**
- OR and HR are non-collapsible: unadjusted ≠ adjusted even without confounding
- In heterogeneous patient populations, unadjusted OR is attenuated toward 1.0
- The attenuated OR applies to a "typical" mixture patient — not to any specific patient
- Adjusted OR: estimates the effect for a patient with specified covariate values — clinically useful
- Failure to adjust in logistic/Cox models loses power (unaccounted heterogeneity has nowhere to go)
- In linear models: unadjusted mean difference is unbiased; adjusting only improves precision
- Senn quote: "We usually treat individuals not populations"

---

## Subgroup Analysis

### Statistical Errors — Improper Subgrouping section
**URL**: https://fharrell.com/post/errmed#improper-subgrouping
**Key points:**
- Conducting separate hypothesis tests within subgroups and comparing results is wrong
- The correct approach: include treatment × subgroup interaction in the overall model
- Interaction effects require much larger sample sizes than main effects
- Most published subgroup effects are noise; Simpson's paradox can make them misleading
- Bayesian shrinkage of subgroup effects toward overall effect is more reliable
- Pre-specify any subgroup analyses; report interaction estimate and CI, not within-subgroup p-values

---

## Diagnosis and Biomarkers

### Sensitivity and Specificity: The Backward Problem (Ch. 19 blog connections)
**Key principle** (from BBR Ch. 3 and 19):
- Sensitivity = Pr(test+ | disease+): conditions on what you don't know at test time
- What you need: Pr(disease+ | test+, patient characteristics)
- This is forward conditioning, obtained directly from logistic regression model
- "The use of sensitivity and specificity in prospective cohort studies is the mathematical
  equivalent of making three left turns in order to turn right"

### Resources for Statistical Methods in Biomarker Research
**URL**: https://fharrell.com/post/addvalue (related)
**Key points:**
- Biomarker "adds value" only if it improves predictions beyond existing clinical information
- Must compare model with vs. without new biomarker (likelihood ratio test, AUC improvement)
- Continuous biomarker should never be categorized as high/low
- Bootstrap validation of entire model selection process

---

## Change Scores

### Change from Baseline
**URL**: https://fharrell.com/post/errmed#change-from-baseline
**Key points:**
- Change scores require: linear pre-post relationship, slope ≈ 1, same transformation,
  no floor/ceiling, variable not used as inclusion criterion
- ANCOVA (model current status with baseline as covariate) uniformly superior
- Percent change: assumes multiplicative relationship; meaningless when baseline varies widely
- Ordinal models for ANCOVA: require no transformation assumption at all
- KCCQ ceiling effect: patients with high baseline scores cannot improve; change score
  misleads about treatment benefit in this group

---

## Machine Learning vs. Statistical Modeling (BBR-relevant angle)

### Classification vs. Prediction (classif)
**URL**: https://fharrell.com/post/classification
**Key points (connects to BBR Ch. 18 and 19):**
- Classification = forced binary decision; appropriate only at the very end of analysis
- Probability estimation is the correct intermediate goal
- A probability model can always produce a classifier; a classifier cannot produce probabilities
- Medical diagnosis should provide Pr(disease | data), not a binary diagnosis label
- Threshold for action should come from utility function (cost of FP vs. FN), not from AUC

### In ML Predictions for Health Care, the Confusion Matrix Is a Matrix of Confusion (2018)
**URL**: https://fharrell.com/post/mlconfusion
**Key points:**
- AUROC measures discrimination; sensitivity/specificity condition on the unknown
- Calibration (accuracy of probability estimates) is ignored in most ML health applications
- For medical decision-making, calibrated probabilities are essential
- The confusion matrix optimizes an implicit threshold that is rarely the correct clinical threshold

---

## Observational Studies

### Causal by Design (2026)
**URL**: https://fharrell.com/post/causal
**Key points (connects to BBR Ch. 17):**
- RCTs analyzed respecting randomization are causal by design; no causal calculus needed
- Retrospective observational studies with target trial emulation cannot achieve causal inference
- Prospective design is required for observational studies to provide credible causal evidence
- "Confounding by indication" is only solved by randomization, not by statistical adjustment alone
  (though adjustment reduces bias from measured confounders)

---

## Reproducible Research

### Decline Effect and Reproducibility (BBR Ch. 21 related)
**Key connections:**
- Blog discussions of publication bias, HARKing, decline effect
- See also Andrew Gelman's blog for related discussion of researcher degrees of freedom
- Pre-registration resources: OSF (osf.io), ClinicalTrials.gov, AsPredicted

---

## Quick Reference: Which Blog Post for Which BBR Topic

| BBR topic | Blog post |
|---|---|
| p-value problems | pval-litany |
| Statistical errors in papers | errmed |
| Unadjusted ORs in RCTs | marg |
| Subgroup analysis | errmed (subgroup section) |
| Change from baseline | errmed (change section) |
| Sens/spec vs. probability models | bbr/dx + mlconfusion |
| Classification vs. prediction | classification |
| Observational studies / causation | causal |
| Bootstrap validation | bootcal, split-val (→ rms skill) |
| Ordinal models for diagnosis | rpo (→ rms/ordinal.md) |
