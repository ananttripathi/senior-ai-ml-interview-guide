# Causal Inference — Interview Q&A

Heavily tested for senior Data Scientist roles at Meta, Netflix, Airbnb, Uber, and Amazon. Most guides cover A/B testing but miss this. Knowing causal inference distinguishes a senior DS candidate.

The core question causal inference answers: **what would have happened if we had done something differently?** Standard ML predicts P(Y|X); causal inference estimates P(Y|do(X)) — the effect of *intervening* on X.

---

## Foundational Concepts

**[M] What's the difference between correlation and causation? Give a concrete ML example.**

Correlation: two variables move together. Causation: one variable directly causes the other.

ML example: in a recommendation system, users who see many recommendations (high exposure) tend to have higher engagement. But if you simply increase recommendation volume for all users (intervene), engagement might drop — because the high-exposure users were already highly engaged *before* seeing more recommendations (selection bias). Correlation was real; causation was the reverse of what it appeared.

The issue: **confounders**. A third variable (user engagement level) caused both high recommendation exposure AND high engagement. Standard ML learns the confounded association; causal inference isolates the effect of the intervention.

**[M] What is the Potential Outcomes Framework (Rubin Causal Model)?**

For each unit i and treatment T ∈ {0,1}:
- Y_i(1) = outcome if treated
- Y_i(0) = outcome if untreated

**Individual Treatment Effect (ITE)**: τ_i = Y_i(1) - Y_i(0)

The **fundamental problem of causal inference**: we only observe one potential outcome per unit. We can never observe both Y_i(1) and Y_i(0) for the same person at the same time.

**Average Treatment Effect (ATE)**: E[Y(1) - Y(0)] = E[Y(1)] - E[Y(0)]

The goal of most causal methods is to estimate ATE (or CATE — Conditional ATE — i.e., the effect for a specific subgroup).

---

## Methods

### Randomized Controlled Trials (A/B Testing)
The gold standard. Random assignment ensures treatment and control groups are identical in expectation, eliminating confounders. When A/B testing is impossible, use the observational methods below.

---

### Difference-in-Differences (DiD)

**When to use**: you have pre/post data for a treated group and a control group, but treatment was not randomly assigned.

**Assumption**: in the absence of treatment, both groups would have followed *parallel trends*.

**Estimator**:
```
DiD = (Treatment_post - Treatment_pre) - (Control_post - Control_pre)
    = (change in treated) - (change in control)
```

**Example**: You rolled out a new feature to users in California but not Texas (for regulatory reasons). You want to know the feature's effect on retention.
- DiD = (CA retention change) - (TX retention change)
- This removes time trends (both groups experience the same macroeconomic shocks)
- Assumption: CA and TX retention would have moved in parallel without the feature

**[H] What if parallel trends doesn't hold?**
- Test it visually: plot pre-treatment trends for both groups — they should be parallel
- Use synthetic control: construct a weighted combination of control units that best matches the treated unit's pre-period trajectory
- Event study design: estimate DiD at multiple pre-treatment periods (should show no effect) and post-treatment periods

---

### Instrumental Variables (IV)

**When to use**: there's a confounder you can't measure, but you have an instrument — a variable that affects treatment but affects the outcome *only* through the treatment.

**Requirements for a valid instrument Z**:
1. **Relevance**: Z is correlated with treatment X (testable: first-stage F-statistic > 10)
2. **Exclusion restriction**: Z affects Y *only* through X (not directly) — not directly testable; requires domain knowledge
3. **Independence**: Z is independent of the confounders

**Classic example**: effect of education (X) on earnings (Y), with ability as an unobserved confounder. Instrument: distance to nearest college (Z). People living closer to a college are more likely to get a degree (relevance), but proximity to college doesn't directly affect earnings except by making people more likely to attend (exclusion restriction).

**Two-Stage Least Squares (2SLS)**:
```
Stage 1: X̂ = α + β·Z + ε  (predict treatment from instrument)
Stage 2: Y = γ + δ·X̂ + η  (regress outcome on predicted treatment)
δ = IV estimate of causal effect
```

**IV in tech**: using a randomly assigned nudge (e.g., position of a UI element) as an instrument for user behavior (e.g., clicking on a feature).

---

### Regression Discontinuity Design (RDD)

**When to use**: treatment is assigned based on crossing a threshold of a running variable.

**Logic**: units just above and just below the threshold are nearly identical except for treatment status. Compare outcomes in a narrow bandwidth around the threshold.

**Example**: a credit card company gives premium rewards to customers with credit score ≥ 750. Comparing customers at 749 vs 751 is nearly a natural experiment — they're almost identical, but one group gets the treatment.

**Sharp RDD**: everyone above threshold gets treated. **Fuzzy RDD**: threshold increases probability of treatment (use IV techniques).

**Key assumptions**:
- Running variable distribution is smooth at the threshold (no manipulation/bunching)
- No other policies change at the same threshold

**Test for manipulation**: McCrary density test — check if there's a discontinuity in the density of the running variable at the cutoff.

---

### Matching / Propensity Score Methods

**When to use**: observational data where treatment was not random, but you have rich covariate information.

**Propensity score**: P(T=1 | X) — probability of being treated given covariates. Estimated with logistic regression or gradient boosting.

**Propensity Score Matching (PSM)**:
1. Estimate propensity score for each unit
2. For each treated unit, find a control unit with similar propensity score (nearest neighbor, caliper)
3. Compare outcomes between matched pairs

**Inverse Probability Weighting (IPW)**:
- Weight treated units by 1/P(T=1|X) and control units by 1/P(T=0|X)
- Creates a pseudo-population where treatment is independent of covariates
- More efficient than matching; sensitive to extreme propensity scores (trim at 0.05 and 0.95)

**Doubly Robust Estimators** (best practice):
- Combine outcome model + propensity model
- Consistent if *either* model is correctly specified (not both need to be perfect)
- Use `EconML` or `DoWhy` in Python

---

### Uplift Modeling (Heterogeneous Treatment Effects)

**The question**: not just "does the treatment work on average?" but "for *whom* does it work best?"

**Four user segments**:
| Segment | Behavior | Action |
|---------|---------|--------|
| **Sure things** | Convert with or without treatment | Don't waste treatment budget |
| **Persuadables** | Convert only *because* of treatment | Target these! |
| **Lost causes** | Won't convert regardless | Don't waste budget |
| **Sleeping dogs** | Convert without treatment, hurt by it | Definitely avoid treating |

**Uplift = CATE (Conditional Average Treatment Effect)**:
```
τ(x) = E[Y(1) - Y(0) | X = x]
```

**Methods**:
- **S-learner**: train one model on (X, T); CATE = model(X, T=1) - model(X, T=0)
- **T-learner**: train separate models for T=0 and T=1; CATE = model_T1(X) - model_T0(X)
- **X-learner** (best for imbalanced treatment): uses cross-fitting; handles when n_treated ≠ n_control
- **Causal Forest** (GRF): tree-based CATE estimation; provides confidence intervals; best for tabular data

**Evaluation** (no ground truth CATE available):
- **Qini curve / Uplift curve**: sort by predicted CATE; cumulative uplift should be monotone for a good model
- **AUUC** (Area Under Uplift Curve): single metric summary
- **Rank-weighted ATE**: weight by predicted rank

**Python libraries**: `EconML` (Microsoft), `CausalML` (Uber), `DoWhy` (Microsoft)

---

## Network Effects and Interference

A critical gap in standard A/B testing that comes up at Meta, LinkedIn, and Uber.

**SUTVA violation**: standard causal inference assumes no interference between units. This fails when:
- Users interact (social networks: my treatment affects your outcome)
- Shared resources (ride-share: more supply in one area affects wait times elsewhere)
- Viral features (one user sharing causes non-users to join)

**Solutions**:
- **Cluster randomization**: randomize at the cluster level (city, friend group) instead of user level. Reduces interference but increases variance.
- **Ego network randomization**: randomize users; measure at the ego-network level (user + friends)
- **Switchback experiments**: for marketplace effects — alternate treatment periods over time within the same unit (e.g., treat a city for 1h, control for 1h, repeat)
- **Graph-based methods**: model the network explicitly in the treatment effect estimator

---

## Common Interview Questions

**[M] "Our A/B test shows a lift, but we can't randomly assign users. How do you estimate the causal effect?"**

Answer: describe the observational setting, identify the confounder, then choose the appropriate method:
- If there's a threshold → RDD
- If there's a natural instrument → IV
- If you have rich pre-treatment covariates → matching / IPW / doubly robust
- If you have pre/post data → DiD

**[H] "How do you handle selection bias in a recommendation system evaluation?"**

Selection bias: the model recommends items it thinks users will like → only shows popular/likely-to-be-clicked items → logged data is biased toward items already known to perform well. New or diverse items are underexplored.

Solutions:
- **Inverse propensity scoring**: weight each interaction by 1/P(item was shown) to correct for the logging policy
- **Counterfactual evaluation**: use importance sampling to estimate what performance would be under a new policy using data from the old policy
- **Unbiased offline evaluation**: deploy a small fraction of traffic with random recommendations (exploration) to collect unbiased signal
