# ML System Design: Recommendation Systems

One of the most common ML design questions at MAANG. Appears as: "Design YouTube recommendations", "Design Netflix's homepage", "Design a product recommendation system."

---

## Problem Statement

**Generic ask**: Design a recommendation system for [platform] that surfaces the most relevant [items] for each [user].

---

## Step 1: Clarify Requirements

Questions to ask:
- What is the inventory size? (thousands vs millions vs billions of items)
- What is the latency requirement? (homepage load: <200ms; email digest: no real-time constraint)
- What signals do we have? (explicit ratings? implicit clicks/watch time? social graph?)
- Cold start situation: how do we handle new users and new items?
- What is the business goal? (engagement? revenue? content diversity? creator satisfaction?)
- Are there fairness or legal constraints? (e.g., must not filter based on protected attributes)

---

## Step 2: Two-Stage Architecture

For any recommendation system at MAANG scale, you need two stages:

```
[Millions of items]
        ↓
  [Retrieval / Candidate Generation]  → ~100–1000 candidates
        ↓
  [Ranking]                           → top ~10–50 items
        ↓
  [Re-ranking / Business Logic]       → final output
```

### Why two stages?
- Full corpus scoring with a deep model is too slow at serving time
- Retrieval uses simple, fast models to filter; ranking uses expensive models for precision

---

## Stage 1: Candidate Retrieval

### Approach A: Collaborative Filtering
- Matrix factorization (SVD, ALS): decompose user-item interaction matrix into user and item latent factors
- Good for: systems with rich interaction history
- Weakness: cold start (new users/items have no embeddings)

### Approach B: Two-Tower Neural Network (state-of-the-art)
```
User Tower                    Item Tower
──────────                    ──────────
user_id                       item_id
user_age, gender              item_category
watch_history                 item_embedding
device_type                   creation_date
      ↓                             ↓
    MLP                           MLP
      ↓                             ↓
 user_emb (128d)             item_emb (128d)
                ↘             ↙
              dot product / cosine similarity
```

Training: contrastive learning with in-batch negatives or sampled negatives. The model learns to push similar user-item pairs together and dissimilar pairs apart.

Serving:
1. Pre-compute item embeddings offline → index in ANN (FAISS, ScaNN, Weaviate)
2. At query time: compute user embedding → ANN search → top K candidates in <10ms

### Approach C: Content-Based Retrieval
- Item metadata → item embeddings (TF-IDF, BERT embeddings for text; ResNet for images)
- Retrieve items similar to user's recently interacted items
- Useful for new item cold start (no interaction history needed)

### Approach D: Graph-based (Pinterest Pinnersage, Twitter)
- Model user-item-user relationships as a graph
- Walk the graph (random walk, GraphSAGE) to find candidate items
- Captures community effects and social signal

---

## Stage 2: Ranking Model

### Features

**User features:**
- Historical behavior: clicked items, categories, dwell time
- Demographics (age, location, language)
- User embeddings from retrieval model
- Real-time session context (what did they just watch/click?)

**Item features:**
- Category, tags, description embedding
- Popularity (global, segment-specific)
- Recency (publication date, virality score)
- Item embedding from retrieval model

**Cross (interaction) features:**
- User × item category affinity
- User × creator affinity
- User × topic embedding similarity

### Model Choices

| Model | Pros | Cons |
|-------|------|------|
| Gradient Boosted Trees (XGBoost/LightGBM) | Fast, interpretable, handles tabular well | Doesn't learn feature interactions automatically |
| Deep neural net (Wide & Deep, DCN) | Learns complex interactions, handles embeddings | Slower to train/serve, needs more data |
| Two-stage: GBT for speed + NN for complex features | Balance of speed and quality | More complex to maintain |

**Wide & Deep (Google Play):**
- Wide: linear model for memorization (learned cross-product features)
- Deep: embedding-based MLP for generalization
- Jointly trained

**Deep & Cross Network (DCN v2):**
- Cross network: explicitly models feature interactions of any order
- Deep network: standard MLP
- Better than Wide & Deep at capturing interaction patterns

### Training Target

What you optimize depends on business goal:

| Target | Use case |
|--------|---------|
| CTR (click-through rate) | Maximize clicks |
| Watch time | Maximize engagement (YouTube, Netflix) |
| Multi-task: CTR + watch time + save rate | Balance multiple objectives |
| Conversion rate | E-commerce |

**Multi-task learning**: train a single model with multiple prediction heads. Main tower shared, task-specific heads on top. Prevents optimizing one metric at the expense of others.

---

## Stage 3: Re-ranking & Business Logic

After ranking, apply:
- **Diversity**: don't show 10 videos from the same creator. Inject diversity using MMR (Maximum Marginal Relevance) or Determinantal Point Processes (DPP).
- **Freshness boost**: apply a decay-adjusted score boost to recent content.
- **Business rules**: filter blacklisted items, enforce safe search, apply legal constraints.
- **Exploration**: ε-greedy or Thompson sampling to occasionally surface new items (avoids filter bubbles).

---

## Cold Start Problem

### New user cold start
- Onboarding: ask users to select interests (explicit signal)
- Fallback to popular items in their geography
- Use any available context (device type, time of day, referral source)
- Update personalization rapidly with first interactions (online learning or frequent retraining)

### New item cold start
- Use content-based features for immediate retrieval (no interaction history needed)
- Exploration: route to a small fraction of users with high curiosity score
- "Item age" as a feature with a fresh item score boost
- Once items accumulate interactions, transition to collaborative signals

---

## Offline Evaluation

Metrics:
- **NDCG@K** (Normalized Discounted Cumulative Gain): measures ranking quality; discounts items ranked lower
- **Recall@K**: fraction of relevant items retrieved in top K
- **Hit Rate@K**: does the user interact with at least one recommended item in top K?
- **Diversity**: intra-list diversity (average pairwise dissimilarity)
- **Coverage**: fraction of item catalog ever recommended

Holdout: user-based split (train on users A-Z, test on held-out users) or time-based split (train on t<T, test on t≥T).

---

## Online Evaluation

Primary A/B metrics:
- Engagement rate (CTR, watch time, save rate)
- Session depth (items interacted per session)
- Return rate (did the user come back tomorrow?)

Guardrail metrics:
- Content diversity (entropy of recommended categories)
- Creator distribution (are a few creators dominating?)
- Latency p99 (recommendation must not slow down page load)
- Revenue (if there are ads, recommendation shouldn't cannibalize)

---

## Monitoring & Retraining

**Signals to monitor:**
- Prediction score distribution drift (PSI)
- Click-through rate real-time monitoring
- Recommendation freshness (average age of recommended items)
- Coverage decay (are we recommending fewer unique items over time?)

**Retraining strategy:**
- Daily retraining is standard for most production systems
- Near-real-time: use streaming features + online learning for session context
- Evaluate: shadow deploy new model, compare on 7-day holdout before ramp-up

---

## MAANG-Specific Angles

### YouTube (Google)
- Scale: 800M videos, 2B users
- Challenge: diversity vs engagement (filter bubbles), watch time optimization led to recommendation of extreme content
- Key design: multi-objective ranking (not just CTR), separate safety/quality signal layer

### Netflix
- Challenge: cold start (new shows), multiple user profiles per household, international content
- Key design: row-level personalization (each row has a different algorithm), strong A/B testing culture

### Meta (Facebook/Instagram)
- Challenge: social graph signal, freshness (posts are ephemeral unlike videos), creator economy balance
- Key design: Graph Neural Networks for social signal, multi-task learning across like/comment/share/click

### Amazon
- Challenge: purchase intent vs browsing, long-tail items, "customers also bought" vs "you might like"
- Key design: session-based recommendation (what are they shopping for right now?), explicit vs implicit feedback
