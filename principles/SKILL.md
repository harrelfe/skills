---
name: statistical-principles
description: >
  Apply Frank Harrell's statistical principles and philosophy when the user is
  reasoning about study design, inference, p-values, Bayesian vs. frequentist
  methods, model building, variable selection, validation, measurement, or
  reporting results. Use this skill whenever the user asks how to analyze data,
  interpret statistical output, choose between methods, design an experiment,
  or critique a published analysis — even if they don't use the words "Bayesian"
  or "frequentist". The skill encodes a coherent philosophical stance that
  should inform all statistical advice given in this project.
---

# Statistical Principles and Philosophy

This skill encodes the statistical philosophy of Frank Harrell
(Vanderbilt University, Department of Biostatistics), synthesized from
his blog *Statistical Thinking* (fharrell.com). It is the reference
framework for all statistical reasoning in this project.

Primary sources:
- https://www.fharrell.com/post/principles — core principles list
- https://www.fharrell.com/post/journey — Bayesian vs. frequentist philosophy
- https://www.fharrell.com/post/improve-research — research methodology
- https://www.fharrell.com/post/bayes-freq-stmts — how to report results
- https://www.fharrell.com/post/pval-litany — comprehensive p-value critique
- https://www.fharrell.com/post/split-val — model validation

---

## I. Core Principles

These 18 principles, directly from Harrell's *Fundamental Principles of
Statistics*, are the foundation. Everything else in this skill elaborates on them.

1. Use methods grounded in theory or extensive simulation.
2. Understand uncertainty; the most honest inference is a Bayesian model
   that takes into account what you don't know (equal variances? normality?
   interaction terms?).
3. Sampling distributions make frequentist probabilities hard to approximate;
   posterior distributions do not depend on things that shouldn't matter
   (earlier interim analyses, analyses yet to be run).
4. Dealing with multiplicity on the randomness scale is less important than
   setting a higher bar in the first place — formulate more stringent assertions.
5. Base analyses on statistical models: models yield likelihoods that bridge
   Bayesian and frequentist inference and extend naturally to missing data,
   measurement error, censoring, serial correlation, and external information.
6. Unify analyses; avoid one-off tests by using semiparametric models.
7. Design experiments to maximize information.
8. Understand your measurements; question how the underlying information was
   captured.
9. Be more interested in questions than null hypotheses; be more interested in
   estimation than in answering narrow yes/no questions.
10. Verify that sample size supports the intended analyses; live within the
    information content of the data.
11. Avoid dichotomous decisions about model complexity (interactions,
    nonlinearities, unequal variances). Put parameters in the model for what you
    don't know; Bayesian priors on complexity provide continuous solutions that
    relax assumptions as n grows. Resulting uncertainty intervals are honest.
12. Use all information in the data during analysis.
13. Use discovery and estimation procedures unlikely to claim noise is signal.
14. Strive for optimal quantification of evidence about effects.
15. Give decision makers the inputs — other than the utility function — that
    optimize decisions.
16. Present information in ways that are intuitive, maximize information
    content, and are correctly perceived.
17. Give the client what she needs, not what she wants.
18. Teach the client to want what she needs.

---

## II. Bayesian vs. Frequentist Philosophy

### The core asymmetry

Frequentism answers: *How surprising are my data if H₀ is true?*
Bayes answers: *Given my data, what is the probability the effect is real?*

Harrell's formulation: "Far better an approximate answer to the right
question, which is often vague, than the exact answer to the wrong
question, which can always be made precise." (Tukey, quoted approvingly.)

### Why Harrell prefers Bayes

**Directness.** Bayesian posterior probabilities answer the question
directly: P(effect > 0 | data, prior). A frequentist p-value answers an
indirect and often irrelevant question about data extremeness.

**Multiplicity dissolves.** Frequentist multiplicity problems arise from
the chances you give *data* to be extreme, not from the chances you give
*assertions* to be true. Posterior probabilities from independent analyses
are coherent; no ad hoc alpha-spending is needed.

**Stopping rules don't matter.** Posterior probabilities have a simple
interpretation independent of the stopping rule, the number of interim
analyses, or planned future analyses. Frequentist results require complex
adjustments for every deviation from the pre-specified plan.

**Uncertainty is propagated honestly.** Rather than a binary in/out
decision for model terms, Bayesian priors can hold interactions or
nonlinearities "half in the model" — skeptical priors favor simpler
structure, but data can override as n grows. Uncertainty intervals are
wider and more honest.

**Posterior probabilities define their own error rates.** If
P(efficacy) = 0.97, then P(no effect or harm) = 0.03. That 0.03 is the
relevant decision risk, counting both inefficacy and harm — not the
type I error, which conditions on the very thing you're trying to learn.

**Penalized/shrinkage estimation is natural.** Penalized likelihood
(empirical Bayes) has no frequentist inferential framework. Confidence
intervals for shrunken estimates are uninterpretable. Bayesian credible
intervals after shrinkage via skeptical priors are just as interpretable
as with flat priors.

**Complex compound assertions are tractable.** Posterior probabilities
can address: P(efficacy on endpoint A *and* B), P(benefit > trivial),
P(non-inferiority), P(benefit on ≥ 2 of 5 endpoints). Frequentist
closed testing procedures for such goals can become "train wrecks."

**Adaptive and sequential trials.** A Bayesian solution for a simple
two-group comparison embeds unchanged into an adaptive trial design.
Frequentist solutions require highly complex modifications.

### The subjectivity charge

Both paradigms are subjective. The difference is transparency:

- Frequentist subjectivity: choice of data model + choice of sampling
  scheme + stopping rule + 1-tailed vs. 2-tailed + multiplicity
  adjustments + conversion of p-value to evidence (all hidden).
- Bayesian subjectivity: choice of data model + prior distribution
  (explicit, stated, debatable).

"The subjectivist states his judgements, whereas the objectivist sweeps
them under the carpet by calling assumptions knowledge." (IJ Good)

### When Bayes ≠ p-value (even with flat priors)

- Outside Gaussian models, p-values use normal approximations of unknown
  accuracy; Bayesian calculations are exact.
- Equivalence requires fixed sample size and a single data look. Any
  sequential analysis or multiplicity breaks it.
- Two-sided p-values have no clean Bayesian analog.
- Flat priors are usually scientifically silly (e.g., allowing OR = 10,000
  in a drug comparison).

---

## III. Problems with P-values

When advising on the use of p-values, be aware of these failure modes:

**Conditioning.** P-values condition on what is unknown (H₀) and do not
condition on what is known (the data). They are backward probabilities.

**Indirectness.** A p-value cannot provide evidence *for* an assertion,
only against its complement. Proof by contradiction is a poor foundation
when working with probabilities.

**Definition problems.** The event being computed is data "more extreme"
than observed — a construct requiring justification. What counts as
"more extreme" in sequential designs or with multiple endpoints?

**Computation problems.** Outside a few canonical cases, p-values are
approximate (unknown accuracy). In a 2×2 table from an ECMO trial,
13 different p-values were calculated from the same data, ranging from
0.001 to 1.0.

**Multiplicity is intractable.** There is no principled, unique solution.
Bonferroni is consistent with a prior that all nulls are simultaneously
true — often absurd. The choice of 1-tailed vs. 2-tailed is arbitrary
and unresolved.

**Large n.** Trivially small effects produce impressively small p-values;
statistical significance ≠ clinical significance.

**"Negative" trial misinterpretation.** p > 0.05 is not evidence of
equivalence. "Absence of evidence is not evidence of absence." Yet
investigators routinely make this error.

**Gaming.** Any threshold creates a motive to p-hack: swap hypotheses,
exclude subjects, modify the model until p < 0.05.

**Expert misinterpretation.** McShane & Gal (JASA) showed that even
trained statisticians confuse population and sample when a p-value is
present, and that their treatment recommendations changed based on
crossing the 0.05 threshold rather than on the observed effect size.

---

## IV. How to Report Results

### Frequentist reporting (when it must be used)

Avoid these common errors:

| Wrong | Right |
|---|---|
| "Treatment B did not improve SBP (p=0.4)" | "We were unable to find evidence against A=B (p=0.4). More data needed." |
| "Significantly different (p=0.02)" | "Evidence against the null of no difference (p=0.02); the observed effect favors B." |
| "The CI is [−5, 13], so A≈B" | "If indefinitely replicated, 95% of such intervals contain the true value. For the current study, the true value is either inside or outside — we cannot say which." |

Never declare equivalence from a non-significant result without a formal
equivalence test and a clinically specified margin.

### Bayesian reporting (preferred)

Use probability language directly:
- "The probability that B lowers SBP more than 3 mmHg is 0.81."
- "SBP is probably (probability 0.67) reduced with treatment B."
- "Assuming a skeptical prior, the probability of any benefit is 0.93."

The posterior credible interval has the interpretation users actually
want: "The probability is 0.95 that the true treatment effect lies in
[−4.5, 10.5]."

Use the word *probably* liberally. State probabilities. Defer
dichotomous action decisions to decision makers — provide them with
P(effect) and P(effect > clinically meaningful threshold), not a
binary verdict.

If multiple looks were made during the trial, Bayesian results require
no modification; just report the latest cumulative posterior.

---

## V. Study Design

Good design is what permits causal interpretation. Key distinctions:

**Associational question:** "Did treatment B work better in patients who
received it?" — answerable with statistical models in observational data.

**Causal question:** "Would *this* patient fare better under B vs. A?" —
requires randomization (or unverifiable assumptions) for observational
data. Only a randomized (ideally crossover) study can directly answer the
causal question.

### Design checklist
- Maximize information: prefer continuous measurements over categorized
  ones; minimize measurement error.
- Identify and block on known sources of variability (batch effects, day-
  of-week effects, regional variation).
- Pre-specify the analysis completely before seeing the data (see Section VI).
- Choose sample size so the pre-specified analysis is well-powered; do not
  plan complex analyses the sample cannot support.
- Prefer semiparametric models (e.g., proportional odds) that unify what
  would otherwise be a collection of special-case tests.

---

## VI. Pre-specification and Investigator Degrees of Freedom

Pre-specification is essential for confirmatory inference. It prevents
the researcher from entering the "garden of forking paths."

Uncontrolled investigator degrees of freedom — choice of response
variable, subjects included, normalization, model — are the primary
driver of non-reproducibility.

**Bayesian advantage.** You can pre-specify uncertainty: include a
parameter for departure from linearity with a skeptical prior that favors
linearity; include an interaction term with a prior favoring its absence.
As n grows, the data override the prior. This is honest model
pre-specification, and provides wider, more calibrated uncertainty
intervals. Frequentist two-stage approaches (test for linearity, then
decide) do not control type I error.

---

## VII. Model Building Philosophy

**Do not make dichotomous decisions about model complexity.** Including
or excluding interactions and nonlinearities based on a p-value leads to:
- uncertainty intervals that are too narrow (model uncertainty ignored),
- low power for the tests,
- low precision for covariate-specific effect estimates.

Instead: place parameters in the model for every aspect you're uncertain
about, with priors favoring the simpler structure. The posterior reflects
combined uncertainty honestly.

**Use all data during analysis.** Do not discard subjects, truncate
continuous variables, or categorize continuous predictors without a
compelling reason. Dichotomizing a continuous predictor throws away
information and reduces power.

**Use semiparametric models** (proportional odds, Cox) rather than a
collection of one-off tests. They unify the analysis, avoid distributional
assumptions about the outcome, and handle covariates naturally.

**Model specification is subjective — in both paradigms.** The model
choice has more impact than the choice of prior distribution, and unlike
the prior it does not "wear off" as n increases. Choose carefully and
pre-specify.

---

## VIII. Validation

**Split-sample validation is not recommended** except when n > ~20,000.
Problems:
- It validates a single example model, not the modeling process.
- Results vary substantially between splits.
- Researchers sometimes re-split after disappointing results without
  disclosing this.
- Recombining after validation leaves the final model unvalidated.

**Bootstrap (or 100 repeats of 10-fold CV) is preferred** for internal
validation. It validates the entire modeling process including variable
selection, depicts uncertainty in feature selection, and uses all data
for development.

**"External" validation by geography or time** is often done to satisfy
reviewers but is usually misguided: it turns time and location effects
into surprises rather than estimable covariates. Include time and location
in the model, then validate internally.

True independent replication (new investigators, new data) validates:
investigator bias, analysis plan specificity, measurement technology,
survey techniques, inclusion criteria, and systemic biases — things
internal validation cannot address.

---

## IX. Measurement

Treat measurement quality as seriously as analytical method. Ask:

- What does this measurement actually capture?
- Is there systematic bias? Toward whose viewpoint?
- What is the measurement error, and is it ignorable?
- Is resolution being lost by binning a continuous measurement?
- Are there batch effects, edge effects, operator effects, day effects?

Optimal statistical information comes from continuous measurements with
small measurement error. Categorizing a continuous variable to create
"groups" discards information and reduces power — avoid it.

---

## X. Practical Guidance for Common Situations

### "Should I use a t-test or non-parametric test?"
Prefer the proportional odds model; it is a generalization of the
Wilcoxon test, handles covariates, and gives an interpretable effect
measure. Avoid the binary choice.

### "Should I include this interaction term?"
In a Bayesian framework: include it with a skeptical prior rather than
testing and deciding. In a frequentist framework: if including it, keep
it in and accept wider CIs; do not pre-test.

### "The p-value is 0.06 — is it significant?"
Reframe: what does the posterior probability of benefit look like under a
reasonable prior? What does the confidence/credible interval cover? Is
the study adequately powered to distinguish no effect from a clinically
meaningful effect? The 0.05 threshold is arbitrary.

### "The trial was negative — can we conclude equivalence?"
No. A non-significant result means only that you have failed to find
evidence against H₀. Equivalence requires a pre-specified margin and a
formal equivalence test. With Bayes: compute P(|effect| < δ) where δ
is the clinical margin.

### "We had three interim analyses — how do we report the final p-value?"
With frequentist methods: complex alpha-spending corrections are required.
With Bayes: report the posterior as-is; it is independent of the
stopping rule and interim look schedule.

### "Should I validate my model on a held-out test set?"
Use bootstrap internal validation unless n > ~20,000 or you have genuine
external data from independent investigators and a different time/place.
Do not split-sample just to satisfy a reviewer.

---

## XI. Key References from fharrell.com

| Topic | URL |
|---|---|
| Core principles list | https://www.fharrell.com/post/principles |
| Bayesian journey | https://www.fharrell.com/post/journey |
| Research methodology | https://www.fharrell.com/post/improve-research |
| Reporting Bayes vs. freq | https://www.fharrell.com/post/bayes-freq-stmts |
| P-value problems (comprehensive) | https://www.fharrell.com/post/pval-litany |
| Split-sample validation critique | https://www.fharrell.com/post/split-val |
| General Bayesian resources | https://hbiostat.org/bayes |
| RMS course notes | https://hbiostat.org/rmsc |
