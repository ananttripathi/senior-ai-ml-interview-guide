# 9-Step ML System Design Framework

An alternative to NALM — based on the most cited framework from top GitHub repos (alirezadir/Machine-Learning-Interviews, 8k+ stars). Use whichever framework feels more natural; both lead to the same depth of coverage.

---

## Why Two Frameworks?

**NALM** (in [ml-system-design-framework.md](ml-system-design-framework.md)) is top-down and concise — good for quickly structuring your answer in the first 2 minutes.

**9-Step** is granular and sequential — better for staying organized during a 45-minute deep dive where the interviewer follows up on each step.

Most strong candidates blend both: use NALM to sketch the overview, then drill into each of the 9 steps.

---

## The 9 Steps

```
Step 1: Problem Formulation
Step 2: Metrics (offline + online)
Step 3: MVP Architecture
Step 4: Data Collection & Labeling
Step 5: Feature Engineering
Step 6: Model Development & Training
Step 7: Prediction Service (Serving)
Step 8: Online Testing & Deployment
Step 9: Scaling, Monitoring & Updates
```

---

## Step 1: Problem Formulation (~3 min)

Transform the business problem into an ML problem.

**Questions to answer:**
- What is the input? What is the output?
- Is this classification, regression, ranking, generation, or something else?
- What are the latency constraints? (real-time vs batch)
- What are the scale constraints? (QPS, data volume, user count)
- Are there fairness, safety, or regulatory constraints?

**Output of this step**: a one-sentence ML problem statement.

Example: "Given a user and their context (history, current session, time), predict a ranked list of content items maximizing 7-day retention, with p99 serving latency < 150ms."

---

## Step 2: Metrics (~3 min)

Define success before designing the system. Covers both offline evaluation and online business metrics.

### Offline metrics (what you optimize the model for)

| Task | Metric |
|------|--------|
| Binary classification | AUC-ROC, AUC-PR, F1, log loss |
| Ranking | NDCG@K, MAP, MRR, Recall@K |
| Regression | RMSE, MAE, MAPE |
| Generation | BLEU, ROUGE, BERTScore, human eval |
| Retrieval | Recall@K, Precision@K |

### Online metrics (what the business measures)

| System | Primary | Guardrails |
|--------|---------|-----------|
| Recommendation | 7-day retention, CTR | Content diversity, creator fairness |
| Ads CTR | Revenue, ROAS | Organic CTR, session length |
| Fraud detection | Fraud loss rate | False positive rate, user friction |
| Search ranking | Task completion, CTR@1 | Zero-results rate, latency |

**The key sentence**: "I'll optimize log loss offline to ensure calibrated probabilities, and measure CTR and downstream revenue online."

---

## Step 3: MVP Architecture (~5 min)

Sketch the simplest end-to-end architecture that could work. Don't start with the most complex model.

**Components every ML system has:**
```
Data Sources → Feature Store → Training Pipeline → Model Registry → Serving → Monitoring
```

Draw this on the whiteboard. Label each box. The MVP should be a complete, working system — not the final production system.

**Example MVP**: "We start with a logistic regression model, trained daily on last 30 days of click logs, served with precomputed features from a Redis cache. This gives us a baseline to beat."

**Then evolve**: "We can then replace logistic regression with a two-tower neural network for better personalization, and add real-time session features to the serving path."

Starting simple also shows good engineering judgment — not every problem needs a transformer.

---

## Step 4: Data Collection & Labeling (~5 min)

**Where does training data come from?**
- User interaction logs (clicks, purchases, views)
- Explicit feedback (ratings, likes, reviews)
- Human annotation (editorial labels, crowd-sourcing)
- Synthetic data (useful for cold start or safety testing)

**Label quality issues to discuss:**
- **Position bias**: items shown first get clicked more regardless of quality
- **Exposure bias**: you only observe labels for items the current system showed
- **Label noise**: user signals are noisy proxies (a click ≠ a true positive)
- **Label delay**: chargebacks arrive weeks after the transaction

**Train / validation / test split strategy:**
- Random split: only for truly i.i.d. data (rare)
- Time-based split: required for any temporal data (most recommender/ranking/fraud systems)
- User-based split: required if you want to measure cold-start performance

**Class imbalance handling:**
- Undersampling with calibration correction
- Class weights in the loss function
- Focal loss for extreme imbalance

---

## Step 5: Feature Engineering (~5 min)

Organize features into clear categories (shows you think systematically):

| Category | Examples | Update frequency |
|----------|---------|-----------------|
| User features | demographics, account age, taste embedding | Daily |
| Item/content features | category, description embedding, quality score | Daily/hourly |
| User × Item interaction | past CTR on this category, past purchases from this brand | Daily |
| Context features | time of day, device, location, query | Real-time |
| Real-time session features | what did user do in last 5 minutes | Real-time (streaming) |

**Key trade-off to mention**: online (real-time) vs offline (batch) features.

| | Offline features | Online features |
|-|-----------------|----------------|
| Latency | Precomputed, <5ms lookup | Computed on-demand, 10–100ms |
| Freshness | Stale (hours/days) | Fresh (seconds/minutes) |
| Complexity | High (complex aggregations OK) | Must be fast and simple |
| Storage | Feature store (Redis/DynamoDB for online, S3/DW for offline) | Computed in-flight |

**Train-serve skew**: the #1 production ML failure mode. Features must be computed *identically* in training and serving. Use a feature store that shares transformation logic.

---

## Step 6: Model Development & Training (~8 min)

This is where most candidates spend too much time. Be concise.

### Model selection framework

| If you have... | Consider... |
|----------------|-----------|
| Tabular features, structured data | XGBoost/LightGBM |
| Text input | Fine-tuned BERT / transformer |
| Image input | ResNet / ViT with transfer learning |
| User-item interactions | Two-tower, matrix factorization |
| Sequential user behavior | Transformer (SASRec, BERT4Rec) |
| Mixed tabular + embeddings | Wide & Deep, DCN v2 |

### Training infrastructure to mention

- **Data pipeline**: Spark for batch feature computation; Flink/Kafka Streams for real-time features
- **Training framework**: PyTorch / TensorFlow with distributed training (DDP, Horovod)
- **Experiment tracking**: MLflow, Weights & Biases — tracking hyperparameters, metrics, artifacts
- **Model registry**: store versioned models with metadata; enable rollback

### Training best practices

- **Temporal train/val/test split** (not random)
- **Hyperparameter tuning**: Bayesian optimization (Optuna) over random/grid search
- **Early stopping**: monitor validation loss; stop when it plateaus
- **Negative sampling** (for retrieval/ranking): how do you construct negatives?

---

## Step 7: Prediction Service (Serving) (~5 min)

Often underemphasized by candidates. This is where senior-level thinking shows.

### Serving architectures

**Online (synchronous, real-time)**:
```
User request → API Gateway → Feature retrieval (Redis) → Model inference → Response
Latency budget: 100–200ms total
```

**Batch (precomputed)**:
```
Daily job → Score all user-item pairs → Store top-K recs per user → Serve from cache
Latency: ~1ms (pure cache lookup)
Trade-off: stale, but fast and scalable
```

**Near-real-time (hybrid)**:
```
Background scoring job (every 15min) → Refresh top-K recs → Serve from cache
Real-time override: if user just interacted, adjust recommendations in-flight
```

### Inference optimization
- **Model quantization**: INT8 for 2–4× speedup, minimal quality loss
- **Model distillation**: serve a smaller student model that mimics the large teacher
- **Batching**: batch multiple concurrent requests into a single forward pass (vLLM-style for LLMs; similar for other models)
- **Feature caching**: cache recently computed user features to avoid redundant lookups

### Train-serve consistency
- Use a **feature store** that shares transformation logic between training and serving
- Deploy the same preprocessing code (not two separate implementations)
- Integration tests that run the serving pipeline end-to-end with known inputs

---

## Step 8: Online Testing & Deployment (~5 min)

### A/B testing design
- **Randomization unit**: user-level (not session/request-level) to avoid carryover
- **Primary metric**: one (resist testing 10 metrics)
- **Duration**: ≥2 weeks for weekly seasonality; longer for slow metrics (7-day retention needs 7+ days observation + buffer)
- **Guardrail metrics**: check nothing else broke

### Deployment safety
```
Shadow mode (0% traffic)  →  Canary 1%  →  Canary 5%  →  50%  →  100%
                         [24h validation]  [48h]        [1w]
Automated rollback if: latency > SLA, error rate > 1%, primary metric drops > threshold
```

### Interleaving (faster than A/B for ranking)
For ranking systems, show results from both models interleaved in one page. User's click signal is 100× more sensitive than A/B (every click is a comparison). Reach significance in hours instead of weeks.

---

## Step 9: Scaling, Monitoring & Updates (~5 min)

### Scaling considerations
- **Horizontal scaling**: add more inference servers behind a load balancer
- **Index sharding**: for ANN search, shard the item index across multiple servers
- **Feature store scaling**: Redis cluster for online features; S3/Iceberg for offline
- **Training at scale**: distributed training (data parallelism, model parallelism for large models)

### Monitoring (3 layers)
1. **Infrastructure**: latency, error rate, throughput (minutes)
2. **Data quality**: PSI on top features, missing rates (daily)
3. **Model quality**: prediction distribution drift, proxy metrics, true label reconciliation (days/weekly)

### Retraining strategy
- Scheduled: retrain daily regardless (handles slow drift)
- Triggered: PSI > 0.2 on key features → immediate retrain
- Online learning: streaming mini-batch updates for high-velocity systems (ads, real-time ranking)

---

## Comparison: NALM vs 9-Step

| NALM | 9-Step equivalent |
|------|-------------------|
| Need (5 min) | Steps 1–2: Problem + Metrics |
| Architecture (15 min) | Steps 3–6: MVP + Data + Features + Model |
| Learning (10 min) | Step 7–8: Serving + A/B testing |
| Monitoring (5 min) | Step 9: Scaling + Monitoring |

They cover the same ground. NALM is catchier for quick recall; 9-step is more granular for structured execution.

---

## Pacing Guide

| Step | Time | Risk if skipped |
|------|------|----------------|
| 1. Problem formulation | 3 min | Wrong system for wrong problem |
| 2. Metrics | 3 min | Can't evaluate the system |
| 3. MVP architecture | 5 min | No coherent design to discuss |
| 4. Data collection | 5 min | Interviewer thinks you ignore data quality |
| 5. Feature engineering | 5 min | No features = no ML |
| 6. Model development | 8 min | The fun part — but don't over-index |
| 7. Serving | 5 min | Most commonly rushed; senior signal |
| 8. Online testing | 5 min | Shows you care about real-world validation |
| 9. Monitoring | 5 min | Most commonly skipped; senior signal |
| **Buffer / Q&A** | 6 min | |
| **Total** | **50 min** | |
