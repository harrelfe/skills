# Bayesian Blog Posts — Statistical Thinking

Source: https://fharrell.com/post

---

## My Journey from Frequentist to Bayesian Statistics (2017)
**URL**: https://fharrell.com/post/journey
**Key points:**
- Cannot believe in infinitely many repetitions of identical experiments (required by frequentism)
- Bayesian sequential testing: posterior probabilities have simple interpretation independent of stopping rule and frequency of data looks
- Multiple clinical endpoints handled via compound assertions — impossible with closed testing procedures
- Penalized maximum likelihood (empirical Bayes) loses its frequentist inferential framework; Bayesian posterior under skeptical prior is just as interpretable as under flat prior
- Once you do the right simulations or grasp the theory: multiplicity comes from chances you give data to be extreme (frequentist), not from chances you give effects to be real (Bayesian)

---

## Continuous Learning from Data: No Multiplicities from Bayesian Posterior Probabilities (2017)
**URL**: https://fharrell.com/post/bayes-seq
**Key points:**
- Posterior probabilities are computed the same way whether from a one-look or sequential design
- Earlier posterior values are merely obsolete — not relevant to current evidence
- The stopping rule is irrelevant to interpreting the final posterior
- Frequentist multiplicity: "What might have happened?" grows with number of looks. Bayesian: "What did happen?" — no growth
- If trial stops early for efficacy, skeptical prior pulls the estimate back — posterior mean is perfectly calibrated
- Frequentist sequential designs require complex adjustments to point estimates and CIs that are almost never applied in practice
- Quote (Edwards, Lindman, Savage 1963): "It is entirely appropriate to collect data until a point has been proven or disproven, or until the data collector runs out of time, money, or patience."

---

## Bayesian vs. Frequentist Statements About Treatment Efficacy
**URL**: https://fharrell.com/post/bayes-freq-stmts
**Key points:**
- Contrasts language for reporting: frequentist statements are indirect; Bayesian are direct
- Frequentist "positive": "p = 0.03, reject H0" — doesn't tell you how much treatment works
- Bayesian equivalent: "P(BP lowered) = 0.96; P(BP lowered ≥ 3mmHg) = 0.82"
- Frequentist "negative": "p = 0.3, fail to reject H0" — absence of evidence ≠ evidence of absence
- Bayesian equivalent: "P(similarity within 3mmHg) = 0.31 — study is uninformative about similarity"
- Multiple looks: frequentist estimates require complex adjustments; Bayesian results require no modification — report latest cumulative evidence
- Credible interval: "0.95 probability the true effect is in [-4.5, 10.5]" — direct statement
- Confidence interval: "if repeated infinitely, 95% of intervals contain the true value" — indirect

---

## Wedding Bayesian and Frequentist Designs Created a Mess (hybrid)
**URL**: https://fharrell.com/post/hybrid
**Key points:**
- Real example: Asthmatx bronchial thermoplasty RCT used Bayesian design with frequentist contamination
- Hybrid design: Bayesian posterior probabilities used with pre-specified frequentist α constraint
- When enrollment exceeded plan, hybrid created analytical conflicts: Bayesian PP > threshold but α not "controlled"
- Lesson: commit to one paradigm; mixing them inherits the worst of both
- Bayesian posterior is calibrated regardless of when analysis done — no need for frequentist α
- Flat prior used initially; should have used skeptical prior (would have naturally constrained decision)

---

## Borrowing Information Across Outcomes (yborrow)
**URL**: https://fharrell.com/post/yborrow
**Bayesian aspects:**
- blrm with partial PO model: allows treatment effect to differ for mortality vs. non-fatal outcomes
- Bayesian power: P(posterior PP of mortality benefit > 0.95)
- Simulation: vary the prior on the non-PO contrast (τ) to find the interval that gives 90% Bayesian power for mortality
- Ordinal model borrows information across outcome levels; partial PO allows mortality-specific inference
- With n = 1449 and appropriate prior on τ, can achieve reasonable Bayesian power for mortality even in a trial not sized for mortality alone

---

## Traditional Frequentist Inference Uses Unrealistic Priors (uprior, 2024)
**URL**: https://fharrell.com/post/uprior
**Bayesian connection:**
- Standard frequentist inference (two-sided test) corresponds implicitly to a flat prior: P(θ = 0) = P(θ = any other value)
- Flat prior implies the true effect is equally likely to be 0 or 1000 — wildly unrealistic
- Non-informative prior ≠ "objective" analysis — it embeds a specific (unrealistic) assumption
- Modest skeptical priors are more realistic and produce more trustworthy inference
- Bayesian inference with skeptical prior is more conservative than naive frequentist inference

---

## NFL Football Multiplicities (nfl)
**URL**: https://fharrell.com/post/nfl
**Key points:**
- Illustrates Bayesian approach to multiple comparisons in an accessible sports context
- Multiple team comparisons → skeptical hierarchical prior naturally pools estimates
- Frequentist approach would require conservative multiple comparison adjustment
- Bayesian hierarchical model: each team's effect shrunk toward overall mean (James-Stein style)
- Main point: multiplicity comes from the number of chances data have to be extreme; not from the number of assertions you make with a Bayesian posterior

---

## Using Historical Controls in RCTs (hxcontrol)
**URL**: https://fharrell.com/post/hxcontrol
**Key points:**
- Summarize historical control data as posterior → use as prior for new RCT
- Discounting: mix historical posterior with skeptical prior; or use power prior
- Direct modeling approach: include historical data in the joint model with a bias parameter
- "Trust the HD sample size but not what the HD is estimating"
- rmsb/blrm supports this via prior specification on the treatment effect

---

## Quick Reference: Which Post for Which Bayes Topic

| Topic | Blog post |
|---|---|
| Why become Bayesian? | journey |
| Sequential testing without penalty | bayes-seq |
| How to report Bayesian results | bayes-freq-stmts |
| Danger of hybrid Bayes-freq designs | hybrid |
| Multiple endpoints in RCTs | yborrow (partial PO) |
| Frequentist has unrealistic implicit prior | uprior |
| Multiple comparisons example | nfl |
| Historical controls | hxcontrol |
| Prior elicitation | BET Ch. 10 genrec (Dallow reference) |
| Bayesian OCs simulation | BET Ch. 9 + hbiostat.org/r/hmisc/gbayesseqsim |
