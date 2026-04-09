# ML System Design: Ads Click-Through Rate (CTR) Prediction

One of the most common ML design questions at Meta, Google, and Amazon. Ads CTR prediction is the backbone of the online advertising industry — getting it right is critical to billions of dollars of revenue.

---

## Problem Statement

Design an ML system that predicts the probability that a user will click on a given ad, given the context of the current page and the user's history.

**Business goal**: maximize revenue (ad spend × conversion) while maintaining user experience  
**ML task**: binary classification — predict P(click | user, ad, context)

---

## Step 1: Clarify Requirements

Ask before designing:
- **Scale**: how many ads per second? (Meta: ~10M+ auctions/second)
- **Latency**: how fast must prediction be? (<10ms for real-time bidding; <100ms for on-platform)
- **Advertiser types**: brand ads (CPM) vs performance ads (CPC/CPA)?
- **Surfaces**: feed, search, stories, sidebar? (different user intent)
- **Privacy constraints**: cookie deprecation, ATT (Apple), GDPR — limit on cross-app tracking?

---

## Step 2: Success Metrics

### Business metrics
- **Revenue**: total ad revenue per day
- **Advertiser ROI**: ROAS (Return on Ad Spend) for advertisers
- **User experience guardrails**: CTR on organic content (must not drop), session length, uninstall rate

### Proxy ML metrics
- **AUC-ROC**: ranking quality; can we correctly rank higher-quality ads above lower-quality?
- **Log loss (cross-entropy)**: calibration quality; are our predicted probabilities accurate?
- **Calibration (ECE)**: critical for bidding — advertisers bid based on our predicted CTR

**Why both AUC and log loss?** AUC measures ranking; log loss measures calibration. In ads, calibration matters because downstream bid prices are set based on the predicted CTR. A miscalibrated model costs advertisers real money.

---

## Step 3: Architecture

### The Ads Auction Pipeline

```
User event (page load, search query, etc.)
        ↓
[1] Ad Retrieval / Candidate Generation   → filter from billions of ads → ~thousands
        ↓
[2] CTR Prediction                        → score each candidate ad
        ↓
[3] Auction (rank by eCPM = CTR × bid)   → top K ads selected
        ↓
[4] Ad Serving                            → render to user
        ↓
[5] Click / No-click observed             → feedback to training pipeline
```

### Features

**User features (offline/batch):**
- Historical CTR by ad category, format, device
- Demographic segments (age bucket, gender, location)
- Interest embeddings (from previous engagement history)
- Long-term behavioral embedding (what has the user engaged with in the past 30 days?)

**Ad features (offline/batch):**
- Ad category, creative format (image/video/carousel)
- Advertiser, landing page quality score
- Ad age, historical CTR (global and by segment)
- Ad embedding (from image/text features of the creative)

**Context features (real-time):**
- Current page/surface (news feed position, search query)
- Time of day, day of week
- Device type (mobile vs desktop)
- Current session behavior (what did user do in last 5 minutes?)

**Cross (interaction) features:**
- User category affinity × ad category
- User's past CTR on this ad format
- User's past purchases from this advertiser

### Feature Engineering at Scale

**Embedding lookup tables**: user IDs and ad IDs are high-cardinality categoricals — embed them in dense vector space (32–256 dims). Learned end-to-end.

**Feature hashing**: for features with unknown cardinality, hash to a fixed bucket size to control memory.

**Feature freshness**: distinguish cold (batch) features from hot (real-time) features:
- Batch (updated hourly/daily): user long-term embeddings, ad quality scores
- Real-time (updated per-event): current session context, recent click history

---

## Step 4: Model Architecture

### Evolution of CTR Models

**Logistic Regression (2010s baseline)**
- Linear model on manually engineered cross-features
- Extremely fast to serve; interpretable
- Still used as a baseline and for simple surfaces

**Factorization Machines (FM)**
- Learns second-order feature interactions automatically
- `score = w₀ + Σwᵢxᵢ + Σᵢ<ⱼ<vᵢ,vⱼ>xᵢxⱼ`
- Better than LR; faster than deep nets

**Wide & Deep (Google, 2016)**
- Wide: logistic regression on crossed features (memorization)
- Deep: embedding + MLP (generalization)
- Both trained jointly; combines strengths of both

**Deep & Cross Network (DCN v2, 2021)**
- Cross network: explicit polynomial feature interactions of any degree
- Deep network: standard MLP
- State-of-the-art for industrial CTR at Google

**For 2024-2026 interviews**: DCN v2 or a transformer-based model is the expected answer. Explain the trade-offs vs simpler models.

### Production CTR model structure

```
[User features] → embedding lookup → user embedding (256d)
[Ad features]   → embedding lookup → ad embedding (256d)
[Context]       → embedding + dense → context embedding (128d)

All embeddings concatenated → Cross Network (DCN) → Deep Network (MLP)
                                                           ↓
                                                      sigmoid → P(click)
```

---

## Step 5: Training

### Training data

**Label**: click (1) or no-click (0) on an impression  
**Ratio**: clicks are rare — typically 1–5% CTR → class imbalance

**Negative sampling**: can't train on all non-clicked impressions (too many). Sample negatives at a ratio of 1:100 or 1:1000. Correct the model's output calibration after training (multiply by the negative sampling rate).

**Temporal split**: always train on past, evaluate on future. Never random split for ads — temporal ordering matters.

**Training data freshness**: user behavior changes rapidly. Models trained on week-old data degrade. Solution: near-real-time training on streaming data (minutes-old impressions).

### Position bias correction

Users click on ads shown in position 1 far more than position 5, regardless of ad quality. If you don't correct for this:
- Model learns "ads in position 1 are good" instead of "this ad is good"
- Deployed ranking is biased toward what would be shown first anyway

**Fix**: Inverse Propensity Scoring (IPS). Weight each training example by `1/P(position)`. Or train a separate position bias model and factor it out at serving time.

### Online learning / near-real-time training

At Meta and Google, CTR models are retrained continuously:
- New impressions and clicks arrive via Kafka stream
- Mini-batches constructed from recent data (last 10–30 minutes)
- Model weights updated via SGD on a parameter server
- Critical for capturing breaking news, trending events, seasonal changes

---

## Step 6: Calibration (Critical for Ads)

Raw CTR model outputs are NOT calibrated. Advertisers bid `eCPM = CTR_predicted × CPC_bid`. A model predicting CTR = 0.5% when true CTR = 0.1% causes advertisers to overbid and overpay.

**Calibrate after training** using Platt scaling or isotonic regression on a held-out set.

**Monitor calibration per segment**: calibration can be fine globally but broken for a specific advertiser vertical or user demographic. Measure ECE broken down by ad category, device type, etc.

---

## Step 7: Offline Evaluation

- **Log loss**: primary metric; measures calibration quality
- **AUC-ROC**: ranking quality
- **Relative Information Gain (RIG)**: normalized log loss improvement over a constant-rate baseline = `1 - log_loss(model) / log_loss(baseline)`
- **Slice analysis**: measure AUC and log loss by ad format, device type, advertiser vertical, user segment

**Holdout strategy**: time-based split. Train on days 1–13, evaluate on day 14. Check model degrades gracefully across rolling holdout days.

---

## Step 8: Online Evaluation and A/B Testing

**Primary metric**: revenue per thousand impressions (RPM)  
**Guardrails**: organic CTR (must not drop), session length, ad feedback rate (hides, negative reactions)

**Experiment design challenges for ads**:
- **Budget-constrained advertisers**: a better model shifts which users see ads; advertisers hit their daily budget faster → effect looks better short-term but normalizes over time. Need to run experiments long enough.
- **Auction competition effect**: improving CTR model changes auction dynamics, which affects all advertisers in the auction — including those in the control group (interference). Solution: holdout experiment (isolate entire user segments, not just individual users).

---

## Step 9: Monitoring

**Real-time (minutes)**:
- Predicted CTR distribution (PSI daily; alert if > 0.2)
- Click-through rate actual vs predicted (calibration monitor)
- Revenue per auction

**Daily**:
- AUC and log loss on rolling holdout
- Per-segment calibration check
- Feature serving latency

**Retraining triggers**:
- Log loss > threshold on rolling 7-day holdout → trigger retraining
- Any major platform event (new ad format, iOS update, policy change) → manual trigger

---

## Key Trade-offs to Discuss

| Trade-off | Options |
|-----------|---------|
| Model complexity vs latency | Deeper model → better CTR, but slower auction; distill to serve efficiently |
| Freshness vs stability | Real-time training helps recency but can overfit to noise |
| Global vs personalized calibration | Per-segment calibration is better but expensive to maintain |
| Privacy vs performance | Post-ATT world: less cross-app signal; need privacy-preserving embeddings |
| Exploration vs exploitation | Pure exploitation optimizes short-term CTR; exploration discovers better ads long-term |
