# Bayesian Trial Design and Operating Characteristics

Source: https://hbiostat.org/bayes/bet/design (BET Ch. 9)

---

## Table of Contents
1. [Goals of a Therapeutic Trial](#goals)
2. [Bayesian Operating Characteristics](#boc)
3. [Design Example: Two-Treatment Parallel-Group](#example)
4. [Sequential Monitoring and Futility](#futility)
5. [Sample Size Estimation](#samplesize)
6. [Simulating OCs with Hmisc](#simulation)
7. [Bayesian OCs for Frequentist Procedures](#bfreq)
8. [Borrowing Information / Historical Data](#borrowing)

---

## Goals of a Therapeutic Trial {#goals}

A Bayesian trial can simultaneously target multiple conclusions:
- **Efficacy**: P(θ > 0 | data) — any benefit
- **Non-trivial efficacy**: P(θ > MCID | data) — clinically meaningful benefit
- **Futility / inefficacy**: P(θ < γ | data) — treatment doesn't work
- **Harm**: P(θ < −δ | data) — treatment is harmful
- **Similarity**: P(|θ| < ε | data) — treatments are similar
- **Non-inferiority**: P(θ > −δ_NI | data) — treatment no worse by δ_NI
- **Estimation**: full posterior distribution of θ with credible interval

Bayesian methods provide direct probability statements for all of these with the same framework; only the threshold and assertion changes. Sample size requirements may differ by intent, but the analysis machinery is identical.

---

## Bayesian Operating Characteristics {#boc}

Bayesian OCs are simulated over a **continuous distribution** of true effects (sampling prior), not just at the null and a single alternative. This is more realistic and informative than the frequentist two-point (null vs. alternative) approach.

### Key OCs to Report

1. **Probability of early stop for efficacy** as function of true θ
   - At θ = MCID: this is "Bayesian power"
   - At θ = 0: this is the "Bayesian type I analog" (should be low)
   - Trace the full curve — shows discrimination ability

2. **Expected sample size** as function of true θ
   - Under null: want small (most trials futile → stop early)
   - Under MCID: want substantially less than maximum N
   - Bayesian sequential designs regularly achieve 50–70% of maximum N on average

3. **Probability of correct decision** as function of true θ
   - This is the most important OC; not available in frequentist paradigm
   - "Correctness of the conclusion is the only relevant OC for post-data decision making"

4. **Median stopping time** and distribution of sample sizes

### Prototype Design Summary (BET Ch. 9 example)
Maximum N comparison:
- Frequentist: N = 234 for 0.90 power
- Bayesian (0.9 for Δ > 0): N = 196
- Bayesian (0.9 for Δ > MCID): N = 282
- Bayesian sequential average N: ~113
- Minimum N for precision: Np = 60

Simulation results (10,000 trials, δ = 0.3, γ = 0.1):
- 5,184 stopped early for inefficacy (avg N = 62); 97% correct
- 4,010 stopped early for non-trivial efficacy (avg N = 102); 91% correct
- 634 stopped early for similarity (avg N = 423); 96% correct
- 172 reached maximum N = 750 (median true Δ = γ at boundary)

---

## Design Example: Two-Treatment Parallel-Group {#example}

```r
library(Hmisc)
library(data.table)

# Decision thresholds (pre-specified)
pp_efficacy <- 0.95   # P(θ > 0 | data) > 0.95 → stop for efficacy
pp_futility <- 0.90   # P(θ < 0.05 | data) > 0.90 → stop for futility
n_max       <- 500    # maximum sample size
n_min       <- 20     # minimum before any look (ignore early looks)

# Analysis prior: skeptical, mean 0
# P(μ > 1) = 0.1, P(μ > 0.25) = 0.05 (1:1 mixture)
sd1 <- 1    / qnorm(0.9)
sd2 <- 0.25 / qnorm(0.95)
wt  <- 0.5  # 1:1 mixture weight

# Simulate 10,000 trials
set.seed(42)
results <- gbayesMixPost(
  # See ?gbayesMixPost in Hmisc for full argument list
  # Draws mu from sampling prior (same as analysis prior here)
  # For each simulated trial, sequentially computes posterior PP
  # stops when PP threshold hit or n_max reached
)
# For working code see: https://hbiostat.org/bayes/bet/sequential
# and https://hbiostat.org/r/hmisc/gbayesseqsim

# Summary across simulation scenarios
# Plot: P(stop for efficacy) vs. true mu — OC curve
# Plot: Expected N vs. true mu
# Plot: P(correct decision) vs. true mu
```

### Computing Posteriors Analytically (Normal Model)
For a one-sample normal model with known σ = 1 and prior N(μ₀, τ²):

```r
# Posterior after n observations with sample mean ybar:
# Posterior mean:  (μ₀/τ² + n*ybar)  / (1/τ² + n)
# Posterior var:   1 / (1/τ² + n)

bayes_normal_post <- function(ybar, n, prior_mean = 0, prior_sd, sigma = 1) {
  prior_prec <- 1 / prior_sd^2
  data_prec  <- n / sigma^2
  post_prec  <- prior_prec + data_prec
  post_mean  <- (prior_mean * prior_prec + ybar * data_prec) / post_prec
  post_sd    <- sqrt(1 / post_prec)
  list(mean = post_mean, sd = post_sd,
       pp_pos = pnorm(0, mean = post_mean, sd = post_sd, lower.tail = FALSE))
}

# Example: 100 subjects, observed mean = 0.3, skeptical prior σ = 0.5
post <- bayes_normal_post(ybar = 0.3, n = 100, prior_sd = 0.5)
cat("Posterior mean:", round(post$mean, 3), "\n")
cat("P(efficacy):", round(post$pp_pos, 3), "\n")
```

### Using gbayesMixPost (Hmisc) for Mixture Priors
```r
library(Hmisc)

# Compute posterior CDF for mixture-of-normals prior
# Arguments: ybar (observed mean), v (1/n, variance of sample mean)
#            d0, d1 (prior means), v0, v1 (prior variances), mix (weight on first)
#            what: "cdf" or "postmean"

pcdf <- gbayesMixPost(ybar = 0.3, v = 1/100,
                      d0 = 0, d1 = 0,
                      v0 = sd1^2, v1 = sd2^2,
                      mix = wt, what = "cdf")

# P(efficacy) = P(μ > 0):
pp_eff <- 1 - pcdf(0)
cat("P(efficacy):", round(pp_eff, 3))

# P(non-trivial efficacy) = P(μ > 0.25):
pp_nontrivial <- 1 - pcdf(0.25)
```

---

## Sequential Monitoring and Futility {#futility}

Futility stopping is one of the most valuable features of Bayesian sequential designs — especially important in rare diseases where committing patients to a trial that ends negatively at maximum N may deprive them of other opportunities.

```r
# Stopping rules (check after each look, after n_min):
# Stop for efficacy: PP(θ > 0) > 0.95
# Stop for futility: PP(θ < 0.05) > 0.90
# At final look, N_max: report final PP and credible interval

# If no stop by N_max: report PP with statement about precision
# Never "accept the null" — report P(similarity) instead

# Monitoring at predetermined n values (e.g., N = 50, 100, 150, ..., 500):
interim_looks <- seq(50, 500, by = 50)
```

**Senn's caution on early stopping for efficacy**: Stopping early delivers less information about how well the treatment works; the effect estimate is less precise. A precision requirement (credible interval width < target) before declaring success is good practice.

---

## Sample Size Estimation {#samplesize}

### Four Relevant Sample Sizes in Bayesian Design
1. **N for Bayesian power** (P(PP > threshold | true θ = MCID) ≥ target power)
2. **Average/expected N** under sequential design (much lower than N_max)
3. **Maximum N** (cap on enrollment)
4. **N for precision** (N_p): required for credible interval width < pre-specified target

N_p can be estimated before trial:
```r
# Half-width of 95% CI for normal model ≈ 1.96 * posterior_sd
# Posterior SD ≈ sigma / sqrt(n) for large n (data overwhelms prior)
# So: n_p ≈ (1.96 * sigma / target_halfwidth)^2

# Example: want 95% CI half-width < 0.5, sigma = 1
n_precision <- ceiling((1.96 * 1 / 0.5)^2)   # = 16 (trivial example)
```

### Bayesian Power Examples (BET Ch. 9.7)

**Ordinal outcome (PO model)**:
```r
# Using Hmisc::popower() with Bayesian framing:
# Specify outcome distribution under control and target OR
library(Hmisc)
popower(p1 = c(0.20, 0.30, 0.30, 0.20),   # 4-level ordinal, control arm
        odds.ratio = 1.6,
        n = 200)
# Reports: power, ME (margin of error)

# For Bayesian power: simulate blrm fits and compute
# fraction with P(OR > 1 | data) > 0.95
```

**Sample size for prior insensitivity** (BET sec 9.8):
As n increases, posterior is increasingly dominated by data; prior becomes less influential.
Rule of thumb: n is large enough when using a skeptical prior vs. a flat prior changes the posterior mean by < 10% of its value.

---

## Bayesian OCs for Frequentist Procedures {#bfreq}

A useful exercise: compute **Bayesian operating characteristics of a frequentist design** to understand what the frequentist procedure is actually doing in terms of decision correctness.

From BET sec 9.10.3:
```r
# For a frequentist design with alpha = 0.05, one-sided:
# Simulate 10,000 trials with true effects drawn from a distribution
# For each, record: does p < 0.05? (frequentist decision)
# Compute: P(p < 0.05 AND true effect > 0) / P(p < 0.05)
# = Bayesian positive predictive value of the frequentist procedure
# This reveals how often the frequentist "positive" result corresponds to a real effect
# Depends heavily on the prior probability that the null is true
```

Key finding: When most tested drugs truly have no effect (high prior probability of null), even a procedure with α = 0.05 will have many false positives among significant results — this is the "false discovery rate" problem entirely missed by the frequentist α framework.

---

## Borrowing Information / Historical Data {#borrowing}

When historical control data or prior trial data are available:

### Power Prior
Downweight historical data by factor δ ∈ [0, 1]:
- δ = 1: full borrowing (historical data = current data)
- δ = 0: no borrowing (historical data ignored)
- δ = 0.5: half the effective sample size of historical data contributes
```r
# Likelihood contribution from historical data:
# L_h(θ)^δ × L_current(θ)
# Implemented in rmsb::blrm via `prior_counts` argument
```

### Mixture Prior Approach
Mix a skeptical prior (no historical info) with a prior derived from historical data:
```r
# Analysis prior = w × skeptical_prior + (1-w) × historical_posterior
# w chosen to reflect confidence in historical data relevance
```

### Direct Joint Modeling (Preferred when possible)
Include historical data directly in the model with an explicit bias parameter:
```r
# Model: θ_current = θ_historical + bias
# Prior on bias: N(0, σ_bias); σ_bias chosen to reflect concern about drift
# rmsb::blrm supports this via the `prior_intercept_kron` mechanism
```

See also: `fharrell.com/post/hxcontrol` for using historical controls in RCTs.

### Pediatric Extrapolation
FDA guidance (Ye & Travis): Bayesian approach to incorporating adult data into pediatric trials.
- Adult posterior becomes pediatric prior (possibly discounted)
- blrm supports this via prior specification on treatment effect

---

## Key Design References

| Resource | URL |
|---|---|
| BET Ch. 9: Trial Design | hbiostat.org/bayes/bet/design |
| Bayesian sequential simulation | hbiostat.org/r/hmisc/gbayesseqsim |
| Bayesian fully sequential design (video) | fharrell.com/talk/gdesign |
| COVID-19 design example | hbiostat.org/proj/covid19/statdesign |
| Frequentist vs. Bayes side-by-side | hbiostat.org/proj/covid19/statdesign#bftable |
| FDA Bayesian adaptive design guidance | FDA 2019, 2023 |
| Dallow et al. prior elicitation | doi:10.1002/pst.1854 |
| Weber et al. prior knowledge | doi:10.1002/pst.1862 |
