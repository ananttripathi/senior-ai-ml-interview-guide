# Production ML Monitoring

One of the biggest gaps in candidate preparation. Most resources mention "monitoring" as a checkbox — interviewers at MAANG probe this deeply for senior roles. You need to know the 3-layer architecture, statistical tests, and runbook design.

---

## Why Monitoring Fails in Practice

80–90% of ML production failures originate from data problems, not model problems. Yet most candidates spend 70% of their system design answer on model architecture and 5% on monitoring. Interviewers notice this immediately.

Common failure pattern: "We'd retrain the model if performance drops."
Senior signal: "We monitor at three layers — infrastructure, data distribution, and model output — with specific alerting thresholds and runbooks for each."

---

## The 3-Layer Monitoring Architecture

```
Layer 1: Infrastructure Health
  └── Is the serving system up? Latency, error rates, throughput

Layer 2: Data / Input Distribution
  └── Are the features the model receives drifting from training?

Layer 3: Model Output / Prediction Quality
  └── Are predictions behaving as expected? Is real-world performance degrading?
```

Each layer has different latency: Layer 1 alerts in seconds, Layer 2 in hours/days, Layer 3 in days/weeks (needs labels).

---

## Layer 1: Infrastructure Monitoring

Standard SRE-level monitoring — the same as any software service.

| Metric | Alert threshold (example) | Tool |
|--------|--------------------------|------|
| Serving latency p50/p95/p99 | p99 > 200ms | Prometheus + Grafana |
| Request error rate | > 1% over 5 min | PagerDuty |
| Model server CPU/GPU utilization | > 85% for 10 min | CloudWatch |
| Prediction throughput | drops > 20% from baseline | Datadog |
| Feature store read latency | p99 > 50ms | Internal |

These are table stakes. Interviewers move quickly past this to layers 2 and 3.

---

## Layer 2: Data / Input Distribution Monitoring

### Population Stability Index (PSI)

PSI measures how much the distribution of a feature has shifted between a reference (training) period and the current period.

```
PSI = Σ (Actual_i - Expected_i) × ln(Actual_i / Expected_i)

where each i is a bin of the feature's value range
```

| PSI value | Interpretation | Action |
|-----------|---------------|--------|
| < 0.1 | No significant change | No action |
| 0.1 – 0.2 | Moderate shift | Investigate |
| > 0.2 | Major shift | Retrain or rollback |

**Implementation:**
```python
def psi(expected, actual, buckets=10):
    """
    expected: reference distribution (training data)
    actual: current distribution (production data)
    """
    breakpoints = np.quantile(expected, np.linspace(0, 1, buckets + 1))
    
    expected_pcts = np.histogram(expected, bins=breakpoints)[0] / len(expected)
    actual_pcts   = np.histogram(actual,   bins=breakpoints)[0] / len(actual)
    
    # Clip to avoid log(0)
    expected_pcts = np.clip(expected_pcts, 1e-4, None)
    actual_pcts   = np.clip(actual_pcts,   1e-4, None)
    
    return ((actual_pcts - expected_pcts) * np.log(actual_pcts / expected_pcts)).sum()
```

### Kolmogorov-Smirnov (KS) Test

Non-parametric test for whether two samples come from the same distribution.

```python
from scipy.stats import ks_2samp

stat, p_value = ks_2samp(training_feature, production_feature)
# p_value < 0.05 → reject null hypothesis (distributions differ)
```

- More statistically rigorous than PSI
- Sensitive to the most extreme deviation between CDFs
- Use alongside PSI: PSI for severity, KS for statistical significance

### Chi-Squared Test (for categorical features)

```python
from scipy.stats import chi2_contingency

observed = np.array([prod_counts_per_category])
expected = np.array([train_counts_per_category * prod_total / train_total])
chi2, p_value, dof, _ = chi2_contingency(np.vstack([observed, expected]))
```

### What features to monitor

**High priority** (monitor daily):
- Features with high importance (top 10 by feature importance / SHAP values)
- Features known to be volatile (time-based, external API-derived)
- Features that have caused incidents before

**Medium priority** (monitor weekly):
- All numerical features: mean, std, % missing, % out-of-range
- All categorical features: distribution, % new categories (OOV)

**Lower priority** (monitor monthly):
- Stable features with low importance

### Missing value monitoring

```python
missing_rate = df.isnull().mean()
# Alert if missing_rate[feature] > baseline_missing_rate[feature] * 2
```

Sudden spike in missing values often means an upstream data pipeline broke — a very common production incident.

---

## Layer 3: Model Output / Prediction Monitoring

### Prediction distribution monitoring

Monitor the distribution of raw model outputs (scores/probabilities), not just labels.

- **Score distribution drift**: if your fraud model used to output scores averaging 0.12, and now averages 0.35 — something changed even if accuracy looks fine
- Use PSI on the prediction score distribution the same way as input features

### Concept drift vs data drift

| Type | Definition | Example |
|------|-----------|---------|
| **Data drift** (covariate shift) | P(X) changes; P(Y|X) unchanged | User demographics shifted; model still correct per user type |
| **Concept drift** | P(Y|X) changes; model's learned relationship is now wrong | Fraud patterns changed; old features no longer predictive |
| **Label drift** | P(Y) changes; class balance shifted | More fraudulent transactions after a new attack vector |

Data drift is detectable without labels. Concept drift requires labels to detect (harder).

### Proxy metrics (when labels are delayed)

For many systems, ground truth labels arrive days or weeks later (e.g., whether a recommended item was eventually purchased). In the meantime, use proxy metrics:

| System | Label (delayed) | Proxy metric (immediate) |
|--------|-----------------|--------------------------|
| Recommendation | 7-day purchase | Click-through rate |
| Fraud detection | Confirmed fraud (days later) | Dispute rate |
| Loan default prediction | Default (months later) | Early payment missed |
| Content ranking | Long-term engagement | Watch time per session |

Monitor proxy metrics in near-real-time; reconcile with true labels in batch.

### Business metric monitoring

The ultimate truth. Even if model metrics look fine, check:
- Revenue
- User retention
- Conversion rate
- Session length

A model can maintain AUC while catastrophically failing business metrics (e.g., if score calibration shifts and downstream bid prices break).

---

## Retraining Triggers

Avoid the vague answer "we'd retrain if performance drops." Be specific:

### Scheduled retraining (most common in practice)
- Retrain on a fixed cadence regardless of drift: daily, weekly, or monthly
- Simple, predictable, handles slow drift automatically
- Risk: retrains even when unnecessary (wasted compute); misses sudden drift between retrain cycles

### Performance-based triggers
- Alert if offline AUC on a rolling holdout window drops below threshold (e.g., AUC < 0.85)
- Requires labels to be available quickly (or use proxy metric)

### Distribution-based triggers
- Alert if PSI > 0.2 on any top-10 feature
- Trigger retraining immediately on major distribution shift
- No labels required → can catch data drift before it affects model performance

### Hybrid (recommended for senior interviews)
```
If PSI > 0.2 on any key feature:
  → Trigger immediate retraining

Else if offline AUC < threshold (checked weekly):
  → Trigger retraining

Else:
  → Scheduled weekly retraining regardless
```

---

## Alerting Runbook Design

Every alert needs a corresponding runbook. The absence of runbooks is a senior-level red flag.

### Example runbook: "Feature X PSI > 0.2"

```
Alert: Feature 'user_age_bucket' PSI = 0.31 (threshold: 0.2)
Severity: WARNING

Runbook:
1. Check upstream data pipeline for 'user_age_bucket':
   - Query: SELECT COUNT(*) FROM feature_store WHERE feature='user_age_bucket' AND ts > NOW()-1h
   - Expected: > 10,000 rows; if 0 → data pipeline failure, escalate to data eng
2. Check if drift is sustained (> 2 hours) or transient (single spike)
   - If transient: likely a pipeline hiccup, monitor for 1h before action
   - If sustained: proceed to step 3
3. Pull distribution comparison (training vs last 24h):
   - Expected: PSI < 0.1 in historical data
   - Check if new user cohort or geo expansion explains the shift
4. If drift is genuine and sustained:
   - Trigger manual retraining job (link to pipeline)
   - Notify ML team Slack channel
5. If model metrics (proxy CTR) also degraded:
   - Consider rollback to previous model version while retraining
   - Escalate to on-call ML engineer
```

### Alert fatigue prevention

- Distinguish **warning** (PSI 0.1–0.2, human review) from **critical** (PSI > 0.2, trigger action)
- Account for seasonality: weekends, holidays naturally shift distributions — adjust baselines
- Use rolling baselines rather than static training-time baselines for slow-drifting features
- Batch low-severity alerts into a daily digest; page immediately only for critical

---

## Canary Deployment & Shadow Mode

### Shadow deployment
New model runs in parallel with production but predictions are NOT served to users.
- Compare new model outputs to production model outputs on live traffic
- No user impact; detect regressions before they affect users

### Canary deployment
Route 1–5% of live traffic to the new model.
- Small blast radius if the new model degrades
- Automated rollback if key metrics drop beyond threshold (e.g., CTR drops > 5%)

### Ramp schedule
```
Shadow (0%) → Canary 1% → Canary 5% → Canary 20% → 50% → 100%
              [24h]        [48h]        [1 week]     [1w]
              
At each stage: check Layer 2 (feature dist) + Layer 3 (predictions + business metrics)
Automated rollback if: p99 latency > SLA, or error rate > 1%, or primary metric drops > 3%
```

---

## Interview Cheat Sheet

**"How would you monitor this ML system in production?"**

Structure your answer in 3 layers:
1. **Infrastructure**: latency, error rate, throughput — standard SRE monitoring
2. **Data drift**: PSI on top features (daily), KS test for statistical significance, missing value rate alerts
3. **Model quality**: prediction score distribution, proxy metrics in real-time, true label reconciliation in batch

Then cover:
- **Retraining triggers**: scheduled + distribution-based + performance-based hybrid
- **Runbooks**: every alert has a documented response procedure
- **Deployment safety**: shadow → canary → full rollout with automated rollback
