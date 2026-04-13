# Prior Specification for Bayesian Clinical Trials

Sources: BET Ch. 2, 4, 5, 10 — hbiostat.org/bayes/bet

---

## When to Use Which Prior

| Situation | Recommended Prior |
|---|---|
| No prior data; treatment is incremental | Skeptical: N(0, σ) with σ chosen to limit P(large benefit) |
| Strong prior belief treatment won't work | Very skeptical: smaller σ |
| Strong Phase 2 data, similar to Phase 3 design | Phase 2 posterior (possibly discounted) |
| Phase 2 with uncertainty about extrapolation | Mix: w × skeptical + (1-w) × Phase 2 posterior |
| Well-established drug class | Weakly optimistic prior |
| Novel mechanism, curative potential | Less skeptical; wider σ |
| No info; just exclude impossible values | Constraint prior: P(very large benefit) ≈ 0 |

---

## Constructing Skeptical Priors

### Gaussian Skeptical Prior

**Steps:**
1. Define what "large benefit" means clinically (e.g., log-OR = 1.5, or mean difference = 1 SD)
2. Specify your tolerance: P(effect > "large") = p₀ (e.g., 0.1 or 0.2)
3. Solve: σ = "large" / qnorm(1 - p₀)

```r
# Example: "large" log-OR = 1.5, P(log-OR > 1.5) = 0.10
large_effect <- 1.5
p_large      <- 0.10
sigma_skep   <- large_effect / qnorm(1 - p_large)   # ≈ 1.17

# More skeptical: P(log-OR > 1.5) = 0.05
sigma_very   <- large_effect / qnorm(1 - 0.05)       # ≈ 0.91

# Verify:
pnorm(large_effect, mean = 0, sd = sigma_skep, lower.tail = FALSE)  # should be 0.10
```

### Mixture-of-Normals Skeptical Prior (More Flexible)

A 1:1 mixture of two zero-mean normals captures both "very small effects are likely" and
"very large effects are improbable":

```r
# Component 1: P(μ > 1) = 0.1 → sd1
# Component 2: P(μ > 0.25) = 0.05 → sd2
sd1 <- 1    / qnorm(0.9)    # ≈ 0.780
sd2 <- 0.25 / qnorm(0.95)   # ≈ 0.152
wt  <- 0.5                   # equal mixture weights

# Visualize
library(ggplot2)
x   <- seq(-3, 3, length.out = 300)
dens <- 0.5 * dnorm(x, 0, sd1) + 0.5 * dnorm(x, 0, sd2)
ggplot(data.frame(x = x, d = dens), aes(x, d)) +
  geom_line() +
  labs(x = "Treatment effect (log-OR or mean diff)",
       y = "Prior density",
       title = "Skeptical mixture prior")
```

### Equivalent Prior Sample Size

A N(0, σ²) prior on log-OR is roughly equivalent to having observed a prior "null" dataset of
n_prior subjects where: n_prior ≈ 1/σ² (for logistic models on OR scale).

This means:
- σ = 1.0 ≈ 1 prior "null" subject — almost no information
- σ = 0.5 ≈ 4 prior "null" subjects — modest skepticism
- σ = 0.25 ≈ 16 prior "null" subjects — strong skepticism; probably too strong for equipoise

A skeptical prior of N(0, 0.82) (σ ≈ 0.82) means "I believe large effects are unlikely" but
not "I am sure the drug doesn't work."

---

## Priors in brms

```r
library(brms)

# Skeptical prior on treatment log-OR:
prior_skep <- c(
  prior(normal(0, 1.0), class = b, coef = treatment),    # skeptical
  prior(normal(0, 2.5), class = b),                       # weakly informative for other coefs
  prior(student_t(3, 0, 10), class = Intercept)           # diffuse for intercepts
)

f <- brm(y ~ treatment + rcs(age, 4) + sex,
         family  = cumulative(link = "logit"),
         prior   = prior_skep,
         data    = dt,
         chains  = 4, iter = 2000, warmup = 1000,
         seed    = 42)
```

### Choosing brms Priors

For logistic/ordinal regression on log-OR scale:
- `normal(0, 2.5)`: weakly informative; rules out extremely large effects
- `normal(0, 1.0)`: mildly skeptical; P(|log-OR| > 2) ≈ 0.05
- `normal(0, 0.5)`: skeptical; P(|log-OR| > 1) ≈ 0.05
- `cauchy(0, 2.5)`: heavy tails; allows large effects but penalizes them

For intercepts: `student_t(3, 0, 10)` or `normal(0, 5)` — diffuse but proper.

---

## Priors in rmsb / blrm

```r
library(rmsb)

# blrm uses Stan; specify priors via the `conc` argument and prior lists
# Default priors in blrm are regularizing (weakly informative)
# See ?blrm for full prior specification options

# Custom prior on treatment effect:
f <- blrm(y ~ treatment + rcs(age, 4) + sex,
          # Prior scale on regression coefficients (log-OR scale)
          # Passed to Stan; defaults are regularizing
          data = dt)

# Access posterior draws:
draws <- as.data.frame(f$draws)
# Posterior PP for treatment:
mean(draws$treatment > 0)             # P(any efficacy)
mean(draws$treatment > log(1.3))      # P(OR > 1.3)
quantile(draws$treatment, c(0.025, 0.975))  # 95% credible interval
```

---

## Historical Data as Prior

### Summarize Historical Data into Prior

```r
# Step 1: Fit model to historical data
f_hist <- blrm(y ~ treatment + rcs(age, 4), data = dt_hist)
# Step 2: Summarize posterior → use as prior for new trial
# (With discounting: mix historical posterior with skeptical prior)

# Discounting via mixture:
w_hist <- 0.6    # 60% weight on historical posterior
w_skep <- 0.4    # 40% weight on skeptical prior
# Prior for new trial = w_hist * historical_posterior + w_skep * skeptical_prior
```

### Power Prior (Explicit Discounting)

The power prior uses L_hist(θ)^δ as part of the likelihood:
- δ = 0: ignore history
- δ = 1: full borrowing (history = current data in weight)
- Common choice: δ ∈ [0.3, 0.7] for moderate relevance

Not natively in blrm, but achievable by including historical data with a weight variable in the data.

### Robust Mixture Prior (Schmidli et al.)

Mix informative prior (from historical data) with a vague component:
- p_mix × informative_prior + (1 - p_mix) × vague_prior
- p_mix ∈ [0.8, 0.9]: mostly informative with a small vague "escape hatch"
- Protects against prior-data conflict

---

## Prior Elicitation from Experts

Best resource: Dallow, Best & Montague (2018), doi:10.1002/pst.1854

Key principles:
1. Elicit in clinically interpretable quantities (e.g., "what is the probability the drug reduces blood pressure by ≥ 5mmHg?")
2. Translate to prior distribution parameters via moment matching or optimization
3. Use multiple experts; check consistency
4. Avoid anchoring bias by not showing initial estimates

```r
# If expert says: P(θ > 5) = 0.3 and P(θ > 10) = 0.05
# Fit normal: solve for μ, σ such that these constraints are (approximately) met
library(nleqslv)
elicit_normal <- function(q1, p1, q2, p2) {
  f <- function(params) {
    mu <- params[1]; sigma <- params[2]
    c(pnorm(q1, mu, sigma, lower.tail = FALSE) - p1,
      pnorm(q2, mu, sigma, lower.tail = FALSE) - p2)
  }
  nleqslv(c(5, 3), f)$x
}
params <- elicit_normal(q1 = 5, p1 = 0.3, q2 = 10, p2 = 0.05)
# params[1] = prior mean, params[2] = prior SD
```

---

## Checking Prior Sensitivity

Always report sensitivity of main conclusions to prior choice:

```r
# Fit with three priors:
priors <- list(
  flat     = c(prior(normal(0, 10), class = b, coef = treatment)),
  weak     = c(prior(normal(0, 2.5), class = b, coef = treatment)),
  skeptical = c(prior(normal(0, 1.0), class = b, coef = treatment))
)

results <- lapply(priors, function(p) {
  f <- brm(y ~ treatment + age + sex, family = cumulative(),
           prior = p, data = dt, chains = 4)
  # Extract: posterior mean, 95% CI, P(treatment > 0)
  post <- as.matrix(f, pars = "b_treatment")
  c(mean = mean(post), ci = quantile(post, c(0.025, 0.975)),
    pp_pos = mean(post > 0))
})

# If conclusions differ substantially across priors → need more data
# If conclusions stable → results are "prior insensitive"
```

Sample size for prior insensitivity (BET sec 9.8):
As n → ∞, posterior dominated by data. At what n does switching from skeptical to flat prior change the posterior mean by < 10%? That n is "prior insensitive."
