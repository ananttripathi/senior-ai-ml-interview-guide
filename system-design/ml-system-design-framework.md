# ML System Design Framework

The most important skill for senior ML roles. This framework works for any ML design question.

---

## The NALM Framework

Use this to structure any ML system design answer in 45 minutes.

```
N — Need          (5 min)   Define problem, constraints, success metrics
A — Architecture  (15 min)  Data, features, model, training pipeline
L — Learning      (10 min)  Offline eval, online eval, A/B testing
M — Monitoring    (5 min)   Drift, alerting, retraining triggers
```

Leave 10 minutes for follow-up discussion and trade-offs.

---

## Phase 1: Need (~5 min)

### Clarify the problem
Ask before designing. Don't assume.

- What is the business goal? (revenue, engagement, safety, retention)
- Who are the users? (scale, segments, geography)
- What are the latency constraints? (real-time <100ms? batch? near-real-time?)
- What are the throughput requirements? (QPS)
- Are there regulatory/fairness constraints?

### Define success metrics
Split into:
1. **Business metric**: the thing the company cares about (revenue, DAU, conversion)
2. **Proxy ML metric**: what you'll optimize (CTR, precision@K, NDCG, AUC)
3. **Guardrail metrics**: things you must not hurt (latency, churn, impression diversity)

Example: "For a recommendation system, business metric = 7-day retention. Proxy ML metric = NDCG@10. Guardrails = recommendation diversity (entropy), latency p99 < 150ms, no degradation in ads revenue."

---

## Phase 2: Architecture (~15 min)

### 2a. Data

**Training data:**
- Source of labels: explicit (ratings, explicit feedback) vs implicit (clicks, watch time, dwell time)
- Volume: how much data, how far back, freshness requirements
- Class imbalance: especially for ranking, fraud, safety
- Data collection: logs, human annotation, synthetic data

**Features:**
Organize into categories:

| Category | Examples |
|----------|---------|
| User features | demographics, tenure, historical behavior, embeddings |
| Item/content features | category, recency, popularity, embeddings |
| Context features | time of day, device, location, session context |
| Cross features | user × item interaction history |
| Real-time features | current session activity, recent clicks |

**Feature store consideration:** which features need to be precomputed (offline) vs computed at serving time (online)?

### 2b. Model Architecture

**Candidate generation (retrieval) vs ranking:**
Most real-world recommendation/search systems are two-stage:
1. **Retrieval**: fast approximate matching from millions of items → hundreds. Use ANN, BM25, or two-tower models.
2. **Ranking**: expensive model scores the hundreds → top K. Can use deep neural nets, gradient boosted trees.

**Model selection framework:**

| Criterion | Simpler model | Complex model |
|-----------|---------------|---------------|
| Data volume | Small | Large |
| Latency | Tight (<50ms) | Relaxed |
| Interpretability needed | Yes | No |
| Feature interactions | Linear | Non-linear |
| Team ML maturity | Low | High |

Start simple. Logistic regression with good features often outperforms complex models. Add complexity only when you have evidence it helps.

### 2c. Training Pipeline

```
Raw data → Data validation → Feature engineering → 
Training → Evaluation → Model registry → Shadow deployment
```

Key decisions:
- **Offline vs online training**: batch retraining (daily/weekly) vs streaming updates (Flink/Spark Streaming)
- **Training data window**: how far back? Recent data is more relevant; more data = more stable
- **Negative sampling**: for implicit feedback, how do you generate negatives? Random, hard negatives, in-batch?
- **Regularization**: prevent overfitting to popular items (popularity bias)

---

## Phase 3: Learning (Evaluation) (~10 min)

### Offline Evaluation

Choose metrics appropriate to your task:

| Task | Metrics |
|------|---------|
| Ranking/Recommendation | NDCG@K, MAP, MRR, Precision@K, Recall@K |
| Classification | AUC-ROC, AUC-PR, F1, calibration |
| Regression | RMSE, MAE, MAPE |
| NLP generation | BLEU, ROUGE, BERTScore, human eval |

**Critical: holdout strategy**
- Random split: only valid for i.i.d. data
- Time-based split: required for any temporal data (sequential user behavior)
- User-based split: for collaborative filtering (ensure no user in both train and test)

**Slice analysis**: never rely on aggregate metrics alone. Evaluate across:
- User segments (new vs returning, high vs low engagement)
- Content categories
- Platforms (mobile/desktop)
- Geographic regions

### Online Evaluation

A/B test design for the ML system:
1. **Randomization unit**: user-level for most systems (session-level can cause carryover)
2. **Primary metric**: choose one (resist the urge to test 10 metrics)
3. **Duration**: ≥2 full weeks to capture weekly seasonality
4. **Ramp-up**: 1% → 5% → 50% → 100% with go/no-go gates
5. **Interleaving** (advanced): for ranking, show results from both models in one page; user clicks signal preference. Much more sensitive than A/B.

---

## Phase 4: Monitoring (~5 min)

### What to monitor

**Data quality:**
- Feature distribution drift (KL divergence, PSI — Population Stability Index)
- Missing value rates
- Schema violations
- Label distribution shift

**Model performance:**
- Online metrics (CTR, conversion rate) in real-time
- Prediction distribution drift
- Latency percentiles (p50, p95, p99)
- Error rates

**Business impact:**
- Revenue, engagement, retention (can lag by days/weeks)

### Alerting thresholds
- Set separate thresholds for warning vs critical
- Consider seasonality (metrics naturally fluctuate on weekends, holidays)

### Retraining triggers
- Scheduled: retrain daily/weekly regardless (simplest; works for most systems)
- Performance-based: retrain when offline AUC drops below threshold
- Distribution-based: retrain when feature PSI > 0.2

---

## Common System Design Patterns

### Two-Tower Model (for Retrieval)
```
User tower: [user_id, user_features] → MLP → user embedding (128d)
Item tower: [item_id, item_features] → MLP → item embedding (128d)
Score = dot product(user_emb, item_emb)
```
- Train with in-batch negatives or sampled negatives
- Index item embeddings offline using ANN (FAISS, ScaNN)
- At serving: embed user → ANN lookup → top K candidates

### Feature Store Architecture
```
Offline store (S3/DW) ← batch features (user history, item stats)
Online store (Redis/DynamoDB) ← low-latency serving (< 10ms)
Stream processor (Flink) → real-time feature updates
```

### Shadow Deployment / Canary
1. Deploy new model in shadow mode (runs alongside production, predictions not served)
2. Compare offline metrics on live traffic
3. Canary: route 1% of traffic to new model
4. Gradually ramp up with automated rollback triggers

---

## Trade-offs to Discuss

Always acknowledge trade-offs. Interviewers want to see you think in trade-offs, not just propose solutions.

| Decision | Trade-off |
|----------|-----------|
| Real-time vs batch features | Freshness vs cost/latency |
| Model complexity | Accuracy vs latency/interpretability |
| Recall vs precision | Coverage vs quality |
| Personalization vs diversity | Engagement vs discovery/fairness |
| Fresh data vs more data | Recency vs stability |
| Online learning | Freshness vs training instability risk |

---

## 45-Minute Pacing Guide

```
0–5 min   : Clarify requirements, define metrics
5–10 min  : High-level architecture diagram
10–20 min : Data and feature design (detail here)
20–30 min : Model choice and training pipeline
30–38 min : Evaluation (offline + online)
38–43 min : Monitoring and retraining
43–45 min : Summary of trade-offs and alternatives considered
```

---

## Red Flags to Avoid

- Starting to code/design before clarifying requirements
- Proposing an overly complex solution without justification
- Forgetting about the serving infrastructure (only talking about training)
- Ignoring monitoring ("we'd retrain if performance drops" without specifics)
- Not discussing failure modes (what happens when a feature is missing? when the model fails?)
- Treating online and offline metrics as the same thing
