---
name: bayes
description: >
  Use this skill for ANY question about Bayesian analysis or clinical trial design:
  posterior probability calculations, prior specification, skeptical priors, sequential
  and adaptive trial designs, Bayesian stopping rules, posterior exceedance probabilities,
  ROPE analyses, Bayesian operating characteristics, multiplicity in the Bayesian
  framework, multiple endpoints and totality of evidence, Bayesian sample size and power
  for ordinal outcomes, borrowing historical data, Bayesian RCT analysis, blrm, rmsb,
  Stan, brms, MCMC, and frequentist vs. Bayesian comparisons. Also trigger for: why
  posterior probabilities beat p-values, why alpha-spending is unnecessary under Bayes,
  prior justification, simulating Bayesian operating characteristics, reporting Bayesian
  results to regulators, or the BET course (hbiostat.org/bayes/bet). Use this skill
  alongside the rms skill for Bayesian regression mechanics, and the principles skill
  for broader philosophical arguments. Always use this skill for Bayesian methodology.
---

# Bayesian Analysis and Clinical Trial Design

Primary sources:
- [Introduction to Bayes for Evaluating Treatments (BET)](https://hbiostat.org/bayes/bet)
- [Bayesian Clinical Trial Design Course](https://hbiostat.org/bayes)

For deep dives see `references/` subfolder:
- `references/design.md`  — trial design workflow, operating characteristics, simulation
- `references/priors.md`  — prior specification, skeptical priors, historical data
- `references/blog.md`    — curated blog posts on Bayesian topics

**Skill boundary**: This skill covers Bayesian methodology and computation. The **principles
skill** covers the philosophical case for Bayesian over frequentist thinking. The **rms
skill** covers regression model mechanics; use rms for `blrm`/`rmsb` fitting details
and this skill for the Bayesian design and inference framework around them.

---

## The Core Contrast: Frequentist vs. Bayesian

### What Each Paradigm Computes

| | Frequentist | Bayesian |
|---|---|---|
| **Fundamental quantity** | P(data this extreme \| H0 true) | P(effect > threshold \| data, prior) |
| **Conditions on** | The unknown (H0) | The known (data) |
| **Time direction** | Backward | Forward |
| **What it models** | Noise (data extremeness) | Signal (effect probability) |
| **Sequential testing** | Requires α-spending adjustments | No adjustment — posterior updates |
| **Multiple endpoints** | Requires multiplicity correction | Compound assertions, no penalty |
| **Stopping rule** | Must be pre-specified; affects p-value | Irrelevant to posterior |
| **Error probability** | α = P(assert effect \| null true) | P(wrong \| assert effect) = 1 − PP |

### The Fundamental Distinction
Frequentist statistics asks: "How surprised would we be by data this extreme if the null were true?"
Bayesian statistics asks: "Given the data we observed, how probable is the effect?"

The former is indirect evidence; the latter is what decision makers actually need.

**Key quote** (Harrell): "In some ways frequentist hypothesis testing involves modeling noise while Bayesian inference involves modeling signal."

**α is not a decision error probability:**
- α = P(assert efficacy | drug has exactly zero effect) — does not use any data
- The probability that a positive result turns out to be wrong = P(drug ineffective | data, prior) — a Bayesian quantity
- Fisher's error: "not more than 1% of such decisions will be wrong" — wrong, conflates P(H0 | p < α) with P(p < α | H0)

**The poker analogy** (Harrell): Asking for α on a Bayesian design is like asking a poker player winning >$10M/year to justify their ranking by how often they placed bets in games they didn't win.

---

## Bayesian Mechanics

### Bayes' Theorem
**Posterior ∝ Prior × Likelihood**

posterior(θ | data) ∝ prior(θ) × likelihood(data | θ)

The posterior distribution is a complete probability distribution over all possible values of the unknown treatment effect θ. It captures current beliefs after seeing the data.

### Key Posterior Summaries
From the posterior distribution of θ (e.g., treatment log-OR):

| Summary | Meaning |
|---|---|
| P(θ > 0 \| data) | Probability of any efficacy |
| P(θ > ε \| data) | Probability of clinically meaningful efficacy |
| P(θ < γ \| data) | Probability of futility / similarity |
| P(θ₁ > ε₁ AND θ₂ > ε₂ \| data) | Compound assertion (multiple endpoints) |
| Credible interval [L, U] | P(θ ∈ [L, U] \| data) = 0.95 — direct probability statement |
| Posterior mean/median | Point estimate of effect |

**If P(efficacy) = 0.96, then P(wrong if act on efficacy) = 0.04.** This is the actual decision error probability that regulators and decision makers need. It is not available in the frequentist paradigm.

### Sequential Analysis — No Penalty for Multiple Looks
This is the most practically important advantage of Bayesian methods:

- Posterior probability is calibrated regardless of how often data are examined
- Previously computed posteriors are merely **obsolete** — not relevant to current evidence
- "It is entirely appropriate to collect data until a point has been proven or disproven, or until the data collector runs out of time, money, or patience." — Edwards, Lindman, Savage (1963)
- The stopping rule does not affect the interpretation of the final posterior
- Multiplicity comes from giving data multiple chances to be extreme (frequentist concern), not from computing evidence multiple times (Bayesian — no concern)

Hybrid Bayesian-frequentist designs create problems precisely because they import frequentist multiplicity thinking into a framework that does not need it. See `references/blog.md`.

---

## Prior Specification

### Philosophy
- "Data does not create beliefs; rather it modifies existing beliefs." — Harrell
- A prior must be specified to anchor posterior probability calculations
- Prior should be agreed upon collaboratively before data are examined
- For regulatory submissions: iterative process between sponsor and reviewers, pre-study

### Skeptical Priors (Recommended Default)
When no strong prior data exist, use a skeptical prior:
- Centered at zero (equal probability of benefit and harm)
- Low probability of large effects (treatment is incremental, not curative)
- Gaussian with mean 0; smaller σ → more skeptical
- Choose σ by specifying a "large" effect and setting P(effect > large) = 0.1 or 0.2

**Constructing a skeptical prior:**
```r
# If "large benefit" means log-OR > 1.5 and you want P(log-OR > 1.5) = 0.1:
sigma_skeptical <- 1.5 / qnorm(0.9)   # ≈ 1.17

# More skeptical (lower P of large benefit):
sigma_very_skeptical <- 1.5 / qnorm(0.95)   # ≈ 0.91

# Mixture prior (flexible, allows different tail behavior):
sd1 <- 1    / qnorm(1 - 0.1)   # P(μ > 1) = 0.1
sd2 <- 0.25 / qnorm(1 - 0.05)  # P(μ > 0.25) = 0.05
# 1:1 mixture of two zero-mean normals with sd1 and sd2
```

A skeptical prior with mean 0 and σ ≈ 0.5–1 (on log-OR scale) is equivalent to giving weight to a small number of prior "null" subjects — a modest, defensible discount.

**In exchange for using a skeptical prior:** the sponsor can take unlimited data looks with no α-spending penalty.

### Analysis Prior vs. Sampling Prior
- **Analysis prior** (a): used to compute posteriors; agreed upon pre-study
- **Sampling prior / simulation prior** (s): used to generate data in operating characteristic simulations; represents what nature might throw at you; often the same as analysis prior but can differ

If critic uses a different prior than the analyst, the true error probability of a positive finding is P(drug ineffective | data, critic's prior) — still Bayesian, still direct.

### When Prior Data Are Available
- Strong Phase 2 data similar to Phase 3: may use Phase 2 posterior as Phase 3 prior (possibly discounted)
- Discount using: mix of skeptical prior and Phase 2 posterior, or power prior (downweighting by factor δ ∈ [0,1])
- Historical control data: use with caution; explicitly model potential bias
- Expert opinion alone: seldom sufficient without anchoring to data
- See: `references/priors.md` for borrowing/discounting methods

---

## Multiplicity — A Non-Issue for Sequential Testing, Manageable for Others

### Sequential Testing
Sequential testing is a **non-issue** in Bayesian analysis:
- Each data look simply updates the posterior; earlier values are obsolete
- Frequentist α-spending "involves results that may have been obtained and completely ignores results that were obtained since the computation of α does not involve any data" — Harrell
- α-spending is not consistent with rules of evidence

### Multiple Endpoints
The Bayesian solution is **compound assertions**, not corrections:
- Instead of testing endpoints separately and correcting: compute one posterior probability of a compound condition
- P(θ₁ > ε₁ OR θ₂ > ε₂) — probability at least one endpoint shows meaningful benefit
- P(θ₁ > ε₁ AND θ₂ > ε₂) — probability both show meaningful benefit
- P(≥ 3 of 5 endpoints improved) — probability majority show benefit

This is impossible to compute with frequentist methods; trivial with Bayesian posterior simulation.

**Migraine example** (FDA co-primary endpoints: pain, nausea, photophobia, phonophobia):
```r
# From posterior draws:
# P(hit all 4): mean(theta1 > 0 & theta2 > 0 & theta3 > 0 & theta4 > 0)
# P(hit A,B and at least one of C,D): mean(theta1>0 & theta2>0 & (theta3>0|theta4>0))
# P(any 3 of 4): mean((theta1>0)+(theta2>0)+(theta3>0)+(theta4>0) >= 3)
```

### Multiple Treatments and Subgroups
- Multiple treatments: compound assertions comparing specific treatment combinations
- Subgroups: Bayesian hierarchical model with random effects for subgroup-specific effects; automatically pools information across subgroups (partial pooling)
- Evidence for A vs. B comes strictly from A vs. B data and the prior for that effect — not discounted because C vs. D was also assessed

### Raising the Bar Correctly
"It is more clinical, less arbitrary, and easier to interpret when multiplicities are addressed by the assertions being made" — Harrell.
- Wrong approach: lower α threshold (e.g., Bonferroni)
- Better approach: raise the bar on what counts as clinically meaningful (ε > 0 instead of ε = 0)
- Best: compound assertions requiring benefit on multiple or majority of endpoints

---

## Bayesian Clinical Trial Design Workflow

### 1. Specify Goals and Assertions
Define what "success" means as probability statements:
- P(any efficacy) = P(θ > 0 | data) > 0.95
- P(non-trivial efficacy) = P(θ > MCID | data) > 0.80
- P(futility) = P(θ < γ | data) > 0.90 → stop early

### 2. Specify Prior(s)
- Analysis prior: collaboratively agreed, skeptical if no prior data
- Simulation prior: may differ; represents distribution of possible true effects

### 3. Choose Maximum N and Interim Schedule
- Based on Bayesian operating characteristics (see next section)
- More frequent looks → lower expected sample size (no penalty for looks)
- Add precision requirement: don't stop until credible interval width < target

### 4. Simulate Bayesian Operating Characteristics
See `references/design.md` for full simulation workflow.

Key OCs to estimate:
- P(early stop for efficacy | true effect = MCID) — "Bayesian power"
- P(early stop for efficacy | true effect = 0) — "Bayesian type I analog"
- Expected sample size as function of true effect
- P(correct decision) across a range of true effects

Tools: `Hmisc::gbayesMixPost()`, `Hmisc::gbayesSeqSim()` — see https://hbiostat.org/r/hmisc/gbayesseqsim

### 5. Execute Trial
- Compute posterior at each interim look
- Stop for efficacy if P(θ > ε | data) > upper threshold
- Stop for futility if P(θ < γ | data) > futility threshold
- Continue to maximum N if no stopping criterion met

### 6. Report Results
Standard Bayesian reporting (from BET Ch. 10):
- Prior details: how developed, who agreed, when
- P(any efficacy): P(θ > 0 | data, prior)
- P(non-trivial efficacy): P(θ > ε | data, prior) for pre-specified ε
- 0.95 credible interval for treatment effect (emphasize directional PPs over CI)
- Compound posterior probability if multiple endpoints
- For non-superior treatments: P(similarity), P(non-inferiority)
- No hard "win/loss" cutoff; let the posterior speak

---

## Bayesian Regression Modeling (blrm / rmsb)

For deep modeling mechanics → **rms skill** (`references/ordinal.md`). Here: the Bayesian framing.

`blrm` in the `rmsb` package fits Bayesian proportional odds / partial PO models using Stan. Key advantages over frequentist `lrm`/`orm`:
- Exact uncertainty quantification even under penalization/shrinkage
- Posterior distribution for every parameter and derived quantity
- Easy computation of P(OR > 1), P(mean difference > δ), etc. from posterior draws
- Handles partial PO (non-proportional odds for selected covariates) cleanly

```r
library(rmsb)
dd <- datadist(dt); options(datadist = "dd")

# Basic Bayesian ordinal regression
f <- blrm(y ~ rcs(age, 4) + treatment + sex, data = dt)

# Extract posterior probabilities
draws <- as.data.frame(f$draws)          # posterior draws of all parameters
# P(treatment log-OR > 0):
mean(draws$treatment > 0)
# P(treatment log-OR > log(1.3)) — 30% OR improvement:
mean(draws$treatment > log(1.3))

# Partial PO: allow treatment effect to vary by Y level
f_ppo <- blrm(y ~ rcs(age, 4) + sex + treatment,
              ppo = ~ treatment, data = dt)

# Posterior predictive summaries via Predict()
p <- Predict(f, age, treatment, fun = Mean(f))
ggplot(as.data.frame(p), aes(x = age, y = yhat, color = treatment)) +
  geom_line() + geom_ribbon(aes(ymin = lower, ymax = upper), alpha = 0.2)
```

### brms (alternative, more flexible)
```r
library(brms)
# Ordinal logistic regression
f_brms <- brm(y ~ age + treatment + sex,
              family = cumulative(link = "logit"),
              prior  = c(prior(normal(0, 2.5), class = b),
                         prior(normal(0, 10),  class = Intercept)),
              data   = dt, chains = 4, iter = 2000)
posterior_summary(f_brms)
# P(treatment effect > 0):
mean(as.matrix(f_brms, pars = "b_treatment") > 0)
```

---

## Bayesian Power and Sample Size

Bayesian power = P(posterior probability of efficacy exceeds threshold | true effect = MCID, given prior and data model). Unlike frequentist power, it uses no "unobservables" — it is computed by simulation over observable quantities.

```r
library(Hmisc)
# Power for PO model comparison (proportional odds)
popower(p1 = c(0.1, 0.2, 0.4, 0.3),  # Y distribution under control
        odds.ratio = 1.5,             # target OR
        n = 300)

# Bayesian sample size for ordinal outcome: see BET Ch. 9.7
# Key approach: simulate trials, compute P(posterior PP > threshold)
# as function of n and true effect size; find n achieving desired Bayesian power
```

For sample size simulation examples see `references/design.md` and https://hbiostat.org/bayes/bet/design#sec-design-power

---

## MCMC and Computation

Bayesian inference via MCMC (Stan backend):

```r
# Stan interfaces in R:
library(rmsb)    # blrm: Bayesian ordinal/logistic via cmdstanr or rstan
library(brms)    # flexible formula interface to Stan
library(rstanarm) # drop-in replacement for lm/glm with Bayes

# Diagnostics (always check before using results)
library(bayesplot)
mcmc_trace(f$draws, pars = c("treatment", "age"))   # check mixing
mcmc_acf(f$draws, pars = "treatment")               # check autocorrelation
rhat(f)                                              # Rhat < 1.01 required
neff_ratio(f)                                        # effective sample size ratio
```

**MCMC essentials:**
- Run ≥ 4 chains; check R̂ < 1.01 for all parameters
- Check effective sample size (n_eff); want > 400 for reliable posterior summaries
- Warm-up iterations: discard first half (Stan default: 1000 warm-up + 1000 draws per chain)
- For posterior probabilities: use `mean(draws > threshold)` — exact to within MC error

---

## Reporting Bayesian Results to Regulators

Common regulatory objection: "What is your α?"

Response strategy:
1. The Bayesian equivalent of α (probability of asserting efficacy when drug is ineffective) is computed directly from the operating characteristic simulations
2. With an agreed-upon skeptical prior and pre-specified stopping rules, this probability is lower than the frequentist α for the same evidence threshold
3. The actual decision error probability (P(wrong | assert efficacy)) = 1 − PP = 0.04 when PP = 0.96 — this is more relevant than α
4. Reference: FDA guidance documents on Bayesian adaptive designs (2019, 2023)

**Side-by-side comparison**: https://hbiostat.org/proj/covid19/statdesign#bftable

Bayesian statement examples (from `fharrell.com/post/bayes-freq-stmts`):
- "The probability that treatment B lowers SBP is 0.96"
- "The probability that B lowers SBP by ≥ 3mmHg is 0.82"
- "The probability of similarity (within 3mmHg) is 0.31 — the study is uninformative about similarity"
- "The 0.95 credible interval for the mean SBP difference is [2.1, 8.4]; there is 0.95 probability the true difference is in this interval"

---

## Key Distinctions and Pitfalls

| Pitfall | Correct Bayesian approach |
|---|---|
| α-spending for sequential looks | No adjustment needed; posterior simply updates |
| Bonferroni for multiple endpoints | Use compound posterior assertion |
| Stopping for efficacy = "early stopping bias" | Skeptical prior pulls estimate back; posterior is calibrated |
| "Bayesian trial needs frequentist α to be valid" | Bayesian OCs directly provide decision error probabilities |
| Flat/non-informative prior when skepticism appropriate | Use skeptical prior; in exchange get unlimited looks |
| Changing prior after seeing data | Cheating; prior must be locked before unblinding |
| Reporting only posterior mean, not PP | Always report directional posterior probabilities |
| Hybrid Bayesian-frequentist design | Usually creates an analytical mess; commit to one paradigm |

---

## Chapter Map — BET Course Notes

| Chapter | Topic | URL |
|---|---|---|
| 1 | High-level view, essence of Bayes, drug approval | hbiostat.org/bayes/bet/highview |
| 2 | Background, Bayes' theorem, prior to posterior | hbiostat.org/bayes/bet/background |
| 3 | Measures of evidence, frequentist vs. Bayes | hbiostat.org/bayes/bet/evidence |
| 4 | Multiplicity, compound assertions | hbiostat.org/bayes/bet/multiplicity |
| 5 | Sequential analysis, skeptical prior simulation | hbiostat.org/bayes/bet/sequential |
| 6 | Posterior probabilities in decision making | hbiostat.org/bayes/bet/ppdecision |
| 7 | Multiple outcomes, totality of evidence | hbiostat.org/bayes/bet/multytotality |
| 8 | Simulated RCT with two endpoints | hbiostat.org/bayes/bet/simrct |
| 9 | Bayesian clinical trial design, OCs, sample size | hbiostat.org/bayes/bet/design |
| 10 | General recommendations | hbiostat.org/bayes/bet/genrec |

For design simulation detail → `references/design.md`
For prior specification detail → `references/priors.md`
For blog post summaries → `references/blog.md`
