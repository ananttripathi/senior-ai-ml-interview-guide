# Statistics & Probability — Interview Q&A

Critical for Data Scientist roles; important for all. Heavy at Meta, Netflix, Google DS tracks.

---

## Probability Fundamentals

**[E] What is Bayes' theorem? Give a practical ML example.**

P(A|B) = P(B|A) · P(A) / P(B)

Example: spam detection.
- P(spam) = 0.2 (prior: 20% of emails are spam)
- P("free" in email | spam) = 0.8
- P("free" in email | not spam) = 0.1
- P("free" in email) = 0.8·0.2 + 0.1·0.8 = 0.24
- P(spam | "free" in email) = (0.8 · 0.2) / 0.24 = 0.67

In ML: Naïve Bayes extends this to many features with the conditional independence assumption.

**[M] What is the difference between MLE and MAP estimation?**

- **MLE (Maximum Likelihood Estimation)**: find parameters θ that maximize P(data | θ). No prior on parameters. With enough data, converges to the true parameters. Can overfit with small datasets.
- **MAP (Maximum A Posteriori)**: find θ that maximizes P(θ | data) ∝ P(data | θ) · P(θ). Incorporates a prior belief about parameters. With a Gaussian prior on weights, MAP = L2 regularization. With a Laplace prior, MAP = L1 regularization.

When to use: MLE when you have lots of data and no domain knowledge. MAP when you have prior beliefs or limited data.

**[M] Explain the Central Limit Theorem and why it matters for A/B testing.**

CLT: the sampling distribution of the mean converges to a normal distribution as sample size increases, regardless of the population distribution (given finite variance).

Why it matters for A/B: most A/B test metrics (conversion rate, revenue per user) are averages. CLT lets us use z-tests and t-tests on these averages even when the underlying distribution is non-normal (e.g., revenue is heavily right-skewed). With large enough sample sizes, our test statistics follow a normal distribution, making inference valid.

---

## Hypothesis Testing

**[E] What is a p-value? What does "p = 0.03" mean?**

The p-value is the probability of observing results at least as extreme as your data, **assuming the null hypothesis is true**. It is **not** the probability the null hypothesis is true.

p = 0.03 means: if there were truly no effect (H₀ true), you'd see a difference this large or larger only 3% of the time due to random chance alone. At α = 0.05 threshold, you'd reject H₀.

Common misconceptions:
- p = 0.03 does NOT mean there's a 97% chance the alternative is true
- p > 0.05 does NOT prove the null is true

**[M] What is statistical power and how does it affect experiment design?**

Power = P(reject H₀ | H₀ is false) = 1 - β (where β = Type II error rate).

Power is the probability of detecting a true effect. Conventional target: 80%.

Power increases with:
- Larger sample size
- Larger effect size (easier to detect)
- Higher α threshold (but increases Type I errors)
- Lower variance in metric

In A/B testing: calculate required sample size **before** running the experiment using minimum detectable effect (MDE), α, and desired power. Running tests until "significant" (peeking) inflates Type I errors.

**[M] What's the difference between Type I and Type II errors? In fraud detection, which is worse?**

- **Type I error (false positive)**: reject H₀ when it's true. In fraud: flagging a legitimate transaction as fraud. Hurts user experience.
- **Type II error (false negative)**: fail to reject H₀ when it's false. In fraud: missing an actual fraudulent transaction. Hurts the business.

In fraud detection: Type II errors (missed fraud) are typically worse — direct financial loss. But Type I errors can't be ignored — blocking legitimate users causes churn. The trade-off depends on the fraud rate, transaction value, and recovery cost.

**[H] A senior PM says "our A/B test was significant so we should ship it." What questions do you ask?**

1. **What was the primary metric?** Was it the right one? Or did we test many metrics and pick the one that was significant (multiple testing)?
2. **What was the sample size and was the test pre-registered?** Did we stop early when we saw p < 0.05?
3. **What were the guardrail metrics?** Did we check for negative effects on retention, revenue, or other key metrics?
4. **What was the effect size?** Statistical significance ≠ practical significance. A 0.001% improvement in click rate may be noise at a business scale.
5. **Was the experiment population representative?** Any novelty effects? Segment imbalances between treatment/control?
6. **How long did it run?** Did it capture weekly seasonality (at minimum 2 weeks for most products)?
7. **Are there interaction effects?** Were multiple experiments running simultaneously on the same population (interference)?

---

## A/B Testing Design

**[M] Design an A/B test to measure the impact of a new recommendation algorithm.**

1. **Define success metric**: primary = click-through rate (CTR) or 7-day retention; guardrail = session length, revenue, churn
2. **Randomization unit**: user-level (not session-level) to avoid carryover effects
3. **Sample size calculation**: based on baseline CTR (e.g., 5%), MDE (e.g., 5% relative lift = 0.25% absolute), α = 0.05, power = 80%
4. **Experiment duration**: minimum 2 weeks to capture day-of-week effects; longer if user behavior has weekly cycles
5. **Holdout**: stratify by user segments (tenure, platform, geography) to ensure balanced groups
6. **Traffic split**: 50/50 for maximum power; smaller treatment group if there's risk of degradation
7. **Launch checklist**: verify randomization is working (AA test or SRM check), monitor for novelty effects in first 48h

**[H] What is the CUPED technique and when does it help?**

CUPED (Controlled-experiment Using Pre-Experiment Data) reduces variance in the treatment effect estimate by using a pre-experiment covariate (e.g., the same metric measured in the week before the experiment).

Adjusted metric: Ŷ = Y - θ(X - E[X])
where X is the pre-experiment covariate, θ = Cov(Y,X)/Var(X)

Why it helps: variance reduction of (1 - ρ²) where ρ is the correlation between pre and post metrics. For highly correlated metrics (ρ ≈ 0.9), variance can drop by 80%, effectively giving you the same power with far fewer users (or equivalently, shorter experiment runtime).

When to use: when you have pre-experiment data for the same metric and the metric is stable over time. Works best for engagement metrics (retention, session length), less so for new users.

**[H] Your A/B test shows different results by platform (mobile vs desktop). What do you do?**

This is a heterogeneous treatment effect (HTE) / interaction effect. Steps:
1. **Confirm it's not noise**: check if the interaction is statistically significant (interaction term in regression or separate significance tests with Bonferroni correction)
2. **Understand the mechanism**: is there a feature that behaves differently on mobile? UI difference? Different user intent?
3. **Business decision options**:
   - Ship to the segment where you see lift; hold back for others
   - Investigate and build a platform-specific version
   - Run a follow-up experiment targeting the problematic segment
4. **Guard against multiple testing**: if you slice by many dimensions, some will be significant by chance. Pre-specify your segment analysis.

---

## Distributions & Sampling

**[M] What is the difference between a Gaussian and a Poisson distribution? Give a use case for each.**

- **Gaussian (Normal)**: continuous, symmetric, parameterized by μ and σ. Use for: heights, exam scores, residuals in linear regression (by assumption), most aggregate metrics (CLT).
- **Poisson**: discrete, counts of events in a fixed interval, parameterized by λ (mean = variance = λ). Use for: number of requests per second, number of fraud events per day, clicks per session.

Key property of Poisson: mean = variance. If you observe mean << variance, use Negative Binomial instead (overdispersion).

**[M] How would you sample from an imbalanced dataset for training?**

Options:
1. **Oversampling**: duplicate or synthesize minority class examples (SMOTE). Risk: overfitting to duplicated samples.
2. **Undersampling**: remove majority class examples. Risk: losing useful signal.
3. **Class weights**: assign higher loss weight to minority class. Most common in practice — no data modification needed, just adjust loss function.
4. **Threshold tuning**: keep default training data, but tune decision threshold post-training using PR curve.
5. **Stratified sampling**: ensure class distribution is preserved in train/val/test splits.

For severe imbalance (>100:1), combine class weights with undersampling for computational efficiency.
