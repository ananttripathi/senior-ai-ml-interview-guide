# ML System Design: Fraud Detection

A classic MAANG ML design question, especially at Amazon, PayPal, Stripe, and fintech teams. Tests your understanding of real-time ML, class imbalance, adversarial dynamics, and business trade-offs.

---

## Problem Statement

Design a fraud detection system that identifies fraudulent transactions in real-time while minimizing false positives (blocking legitimate users).

---

## Step 1: Clarify Requirements

- **Fraud type**: payment fraud, account takeover, promo abuse, ad fraud, marketplace fraud?
- **Latency**: real-time (< 200ms, decision must be made before transaction completes) vs near-real-time (minutes) vs batch (daily review)?
- **Action**: hard block vs soft block vs review queue?
- **Scale**: how many transactions/second?
- **Fraud rate**: typically 0.01%–1% (severe class imbalance)
- **Chargeback liability**: does the platform or the merchant bear the cost of fraud?

---

## Step 2: Success Metrics

### Business metrics
- **Fraud loss rate**: $ fraud losses / $ total transaction volume
- **False positive rate (FPR)**: fraction of legitimate transactions blocked → directly causes user churn
- **Chargeback rate**: must stay below payment network thresholds (Visa/Mastercard: < 1%)

### ML metrics
- **Precision @ operating threshold**: fraction of flagged transactions that are actually fraudulent
- **Recall @ operating threshold**: fraction of actual fraud caught
- **AUC-PR** (not AUC-ROC): for highly imbalanced datasets, PR curve is more informative
- **False Positive Rate @ 95% recall**: industry-standard benchmark — at what FPR do you catch 95% of fraud?

### The core trade-off

Every fraud system lives on the precision-recall curve. You cannot minimize both fraud losses AND user friction simultaneously. The operating point depends on:
- Transaction value distribution (high-value transactions warrant more friction)
- Customer segment (new users need more scrutiny; trusted long-term customers less)
- Fraud attack pattern maturity

---

## Step 3: Two-Stage Architecture

Fraud detection must be fast (real-time), accurate (high recall), and explainable (disputes).

```
Transaction event
        ↓
[Stage 1: Rule Engine]      → fast hard rules, < 1ms
  ├── Block if: new device + foreign country + high value + 3am
  ├── Block if: transaction velocity > 10 in 1 hour
  └── Allow/block/flag
        ↓ (flagged transactions)
[Stage 2: ML Scoring]       → gradient boosting / neural net, < 100ms
  ├── Score 0–1 (fraud probability)
  └── Decision: block / allow / challenge (2FA) / queue for human review
        ↓
[Stage 3: Human Review]     → for borderline cases
  └── Analyst decision → labeled training data
```

### Why keep the rule engine?

- **Explainability**: regulators and users require explanations for declined transactions. "Your transaction was blocked because..." is much easier with rules than with a black-box model.
- **Speed**: rules are instant (<1ms); ML models take 50–100ms.
- **Adversarial response**: when a new fraud attack pattern is detected, you can add a rule immediately without waiting for model retraining.
- **Trust**: rule-based blocks are defensible in disputes; model-based blocks require additional documentation.

---

## Step 4: Features

### User features (batch, updated daily)
- Account age, historical transaction volume, average transaction value
- Historical fraud flag rate, chargebacks
- Number of linked devices, linked accounts
- Geographic patterns (home country, typical merchant categories)

### Transaction features (real-time, computed per event)
- Transaction amount, merchant category, merchant country
- Transaction time (hour of day, day of week — fraud spikes at 3–5am)
- Payment method (card present vs card not present — CNP has higher fraud rates)
- IP address geolocation vs billing address distance

### Velocity features (real-time aggregates, last N minutes/hours)
- Number of transactions in last 1h, 24h, 7d
- Number of unique merchants in last 1h
- Total spend in last 1h
- Number of declined transactions in last 10 minutes (frustrated legitimate user vs probing fraudster)
- Number of new devices used in last 30 days

### Device / session features
- Device fingerprint (new device flag)
- Browser/app version, OS
- IP risk score (residential vs datacenter vs known VPN/proxy)
- Session behavior anomaly score (mouse movement patterns, typing rhythm — behavioral biometrics)

### Graph features (powerful, often underused)
- Is this merchant or user connected to known fraudulent accounts in the transaction graph?
- Fraud ring detection: shared device fingerprints, shared IPs, shared email domains
- Graph neural network embeddings can capture these relationships

---

## Step 5: Model Architecture

### Primary model: Gradient Boosting (XGBoost / LightGBM)

**Why GBT for fraud (not deep learning)?**
- Tabular data — GBT is state-of-the-art for tabular features
- Interpretable feature importances (important for disputes and compliance)
- Fast inference (<5ms per prediction)
- Robust to missing features at serving time
- No embedding tables needed — handles numeric and categorical directly

### Handling class imbalance

Fraud rate is typically 0.01%–0.1%. Naively trained model will predict everything as non-fraud.

Options:
1. **Scale_pos_weight** (XGBoost): set to `n_negative / n_positive` to rebalance
2. **Cost-sensitive learning**: assign higher misclassification cost to false negatives (missed fraud)
3. **Undersampling with calibration**: undersample majority class; adjust decision threshold after training
4. **Focal loss**: down-weights easy negatives, focuses learning on hard examples (originally from object detection, works well for fraud)

```python
# XGBoost with class weight balancing
model = xgb.XGBClassifier(
    scale_pos_weight=n_neg / n_pos,  # e.g., 999 for 0.1% fraud rate
    eval_metric='aucpr',             # PR AUC for imbalanced data
    ...
)
```

### Threshold tuning

A model outputs a probability score; the threshold determines the decision. Do NOT use 0.5. Tune the threshold on a held-out set to hit your target operating point:
- If recall is paramount (catch all fraud): lower the threshold
- If precision is paramount (don't frustrate users): raise the threshold
- Segment thresholds by transaction value (high-value transactions: lower threshold → more scrutiny)

### Ensemble approach (production)

Layer 1 (fast, coarse): gradient boosting on static features → fraud probability score
Layer 2 (slower, precise): neural network with velocity features and embeddings → refined score
Layer 3 (optional, async): graph neural network for fraud ring detection → runs in background, used for post-hoc flagging

---

## Step 6: Training Data

### Label construction

- **Positive labels**: confirmed fraud — chargebacks, verified fraud reports, analyst-reviewed fraud cases
- **Negative labels**: legitimate transactions (vast majority)
- **Ambiguous**: disputed but not yet resolved → exclude from training until resolved

**Label delay**: chargeback claims arrive 30–120 days after the transaction. This creates a challenge:
- Can't retrain immediately on new data (no labels yet)
- Old labeled data may not reflect current fraud patterns

**Solution**: use proxy labels for near-real-time feedback:
- Rapid decline by user ("I didn't make this transaction")
- System-level signals (multiple declines at same merchant in same hour)

### Train / validation split

**Never use random split for fraud.** Fraudsters learn from your model. Future fraud will be different from past fraud. Use time-based split:
- Train: all transactions from months 1–10
- Validation: month 11
- Test: month 12

Also simulate the label delay: when training on month 10, only use labels that would have been available by month 10 (not chargebacks that arrived in month 12).

---

## Step 7: Adversarial Dynamics

Fraud detection is different from most ML problems because the adversary **adapts to your model**. This is the single most important concept to discuss in a fraud design interview.

### How fraudsters adapt
- Fraudsters run their own transactions to probe your model and find the decision boundary
- Once they find patterns that get through, they scale those patterns
- Model accuracy decays faster than a typical ML model (weeks vs months)

### Mitigations

**Randomize the decision boundary**: add calibrated random noise to your fraud threshold. Makes it harder for fraudsters to identify the exact boundary through probing.

**Diverse signals that are hard to spoof**: behavioral biometrics (typing patterns, mouse movements), device fingerprinting — much harder to fake than IP or email.

**Velocity on novel dimensions**: fraudsters change email/IP/device, but their behavior patterns (merchant types, amounts, timing) often stay consistent. Build velocity features on many dimensions.

**Human-in-the-loop for new attack patterns**: automated anomaly detection to flag novel attack patterns; human analyst team that writes new rules and retrains models quickly.

**Feedback loop**: route some transactions to human review → analyst labels → retrain model → faster adaptation than waiting for chargebacks.

---

## Step 8: Evaluation

### Offline metrics
- **AUC-PR** (not AUC-ROC — fraud is highly imbalanced)
- **FPR @ 95% recall**: industry benchmark — "at what false positive rate do we catch 95% of fraud?"
- Time-based holdout, simulating label delay

### Online evaluation

A/B testing fraud systems is ethically and practically complex:
- You can't let fraud happen to a control group
- Shadow mode: run new model in parallel; compare scores to existing model without acting on new model's decisions
- Champion-challenger: route a small % (1–5%) of borderline transactions to a new model
- Monitor business metrics: fraud loss rate, FPR, chargeback rate

---

## Step 9: Monitoring

**Real-time (minutes)**:
- Score distribution (model output): if average fraud score spikes, new attack may be underway
- Block rate: sudden change → either new fraud wave or model misbehavior
- Rule hit rate: track which rules are firing and at what rate

**Daily**:
- Fraud loss rate (lagged — chargebacks arrive later)
- FPR estimated from user complaints/disputes
- Feature distribution drift (PSI on top features)

**Weekly**:
- Labeled data reconciliation: match transactions to chargeback outcomes; compute true recall/precision
- Model retraining assessment: has model quality degraded?

**Retraining cadence**: retrain weekly at minimum; daily for high-volume systems. The adversarial nature of fraud makes staleness especially costly.

---

## Key Trade-offs to Discuss

| Decision | Trade-off |
|----------|-----------|
| Hard block vs soft block (2FA challenge) | Hard block: no fraud possible, but user friction; soft block: some friction, some fraud gets through |
| Real-time vs near-real-time scoring | Real-time: catch fraud before it happens; near-real-time: can use more expensive models |
| Model explainability vs accuracy | GBT: less accuracy than deep net, more explainable for disputes |
| Recall vs precision | Catch more fraud (lower threshold) vs annoy fewer legit users (higher threshold) |
| Velocity feature window | Short window: catches fast fraud; long window: catches slow-burn account takeover |
| Automated vs human review | Fully automated: scalable but misses edge cases; human-in-loop: more accurate but slower and costly |
