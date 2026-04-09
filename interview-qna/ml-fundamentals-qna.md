# ML Fundamentals — Interview Q&A

Questions are tagged by difficulty: [E] Easy, [M] Medium, [H] Hard.

---

## Bias, Variance & Regularization

**[E] What is the bias-variance tradeoff?**

Bias is error from wrong assumptions in the learning algorithm — high bias underfits. Variance is error from sensitivity to small fluctuations in training data — high variance overfits. Increasing model complexity reduces bias but increases variance. The goal is to find the sweet spot that minimizes total error on unseen data.

**[M] How does L1 vs L2 regularization differ? When would you use each?**

L2 (Ridge) adds λ∑w² to the loss, shrinking all weights toward zero but rarely to exactly zero. L1 (Lasso) adds λ∑|w|, which creates sparse solutions (some weights go to exactly zero) due to the non-differentiability at zero.

- Use L1 when you believe many features are irrelevant (want automatic feature selection)
- Use L2 when you expect most features to matter but want to prevent large weights
- Use Elastic Net (L1 + L2) when you have correlated features and want some sparsity

**[M] Why does increasing regularization increase bias?**

Higher regularization constrains the model more, preventing it from fitting complex patterns in the data. This means the model's predictions are pulled toward simpler patterns (e.g., the mean), introducing systematic error (bias). The model is less flexible, so it may miss real signal in the data.

**[H] A model performs well on training data but poorly on a hold-out test set. Walk me through how you'd diagnose and fix this.**

1. **Confirm it's overfitting**: plot learning curves (train vs val loss over epochs/data size). Large gap = overfitting.
2. **Check data leakage first**: is test data somehow influencing training? (feature from the future, target-based encoding without CV, etc.)
3. **Reduce model complexity**: fewer layers/trees/features
4. **Add regularization**: L1/L2, dropout
5. **Get more training data** (most reliable fix if possible)
6. **Feature selection**: remove irrelevant features
7. **Ensemble methods**: bagging reduces variance

---

## Tree-Based Models

**[E] Why is gradient boosting generally better than random forests?**

Gradient boosting is a sequential ensemble that builds each tree to correct the residual errors of all previous trees, directly optimizing the loss. Random forests are parallel ensembles that average independent trees. Boosting typically achieves lower bias on structured/tabular data because each iteration targets remaining error, but it's more prone to overfitting and requires careful tuning of learning rate and tree depth. Random forests are more robust out-of-the-box.

**[M] Explain how XGBoost handles missing values.**

XGBoost learns a default direction for each split — during training, it tries both directions (left and right branch) for missing values and picks whichever yields the better gain. This direction is stored as part of the model. At prediction time, missing values automatically go to the learned default branch.

**[M] What is the difference between boosting and bagging?**

- **Bagging** (Bootstrap Aggregating): train multiple models in parallel on different bootstrap samples of the training data. Average predictions. Reduces variance. Example: Random Forest.
- **Boosting**: train models sequentially, each correcting errors of the previous. Reduces bias. More prone to overfitting than bagging. Examples: AdaBoost, Gradient Boosting, XGBoost.

**[H] How does LightGBM differ from XGBoost and when would you prefer it?**

LightGBM uses two key innovations:
1. **Gradient-based One-Side Sampling (GOSS)**: retains samples with large gradients (high error) and randomly samples those with small gradients. Reduces data while keeping informative examples.
2. **Exclusive Feature Bundling (EFB)**: bundles mutually exclusive (rarely non-zero simultaneously) features to reduce feature dimensionality.

Result: LightGBM trains much faster on large datasets and high-cardinality features. Prefer LightGBM when dataset is large (>100K rows) or you have many categorical features. Use XGBoost when you want more established behavior and have time to tune.

---

## Model Evaluation

**[E] When would you prefer F1 over accuracy?**

When classes are imbalanced. Accuracy can be 99% if you always predict the majority class on a 99/1 dataset. F1 = 2·(Precision·Recall)/(Precision+Recall) balances the cost of false positives and false negatives. Use F1 (or precision/recall separately) when both types of errors matter or when the positive class is rare.

**[M] What's the difference between AUC-ROC and AUC-PR?**

- **AUC-ROC**: area under the Receiver Operating Characteristic curve (TPR vs FPR). Insensitive to class imbalance — even a poor model on an imbalanced dataset can have high AUC-ROC because the large TN count keeps FPR low.
- **AUC-PR**: area under the Precision-Recall curve. More informative for imbalanced datasets because it focuses on the positive class only.

Rule of thumb: use AUC-PR when positives are rare (fraud detection, disease diagnosis). Use AUC-ROC when classes are roughly balanced.

**[M] What is calibration and why does it matter for ML models?**

A model is calibrated if its predicted probabilities match empirical frequencies. If a model predicts 80% probability for an event, it should actually happen ~80% of the time across all such predictions.

Why it matters: many downstream decisions use raw probabilities (expected value calculations, bid pricing, risk scoring). A miscalibrated model gives wrong probabilities even if ranking is correct (good AUC-ROC but bad calibration).

Fix with: Platt scaling (logistic regression on model outputs) or isotonic regression on a held-out calibration set.

**[H] Describe how you would design an offline evaluation framework for a ranking model.**

1. **Dataset construction**: collect query-document pairs with relevance labels (click-through, explicit rating, or editorial judgments). Use stratified sampling to ensure diverse query types.
2. **Metrics**: NDCG@K (normalized discounted cumulative gain), MAP (mean average precision), MRR (mean reciprocal rank).
3. **Holdout strategy**: time-based split (not random) to prevent leakage; evaluate on a time window that matches production deployment gap.
4. **Baseline comparison**: compare against current production model using A/B-equivalent offline metrics.
5. **Slice analysis**: evaluate across query types, user segments, content categories to catch regressions in subgroups.
6. **Correlation to online**: validate offline metrics are correlated with online A/B metrics before fully trusting them.

---

## Unsupervised Learning

**[E] How does k-means clustering work and what are its limitations?**

K-means iteratively: (1) assigns each point to the nearest centroid, (2) recomputes centroids as the mean of assigned points, until convergence. Limitations:
- Must specify k in advance
- Sensitive to initialization (use k-means++)
- Assumes spherical, equal-size clusters
- Sensitive to outliers
- Doesn't work well with non-linearly separable clusters (use DBSCAN or GMM instead)

**[M] When would you use PCA vs t-SNE vs UMAP?**

| Method | Use case | Preserves |
|--------|----------|-----------|
| PCA | Feature reduction for modeling, preprocessing | Global structure, linear relationships |
| t-SNE | Visualization (2D/3D only) | Local neighborhood structure |
| UMAP | Visualization + can be used for modeling | Both local and global structure (better than t-SNE) |

PCA is the only one you'd use as a preprocessing step in a production pipeline (deterministic, fast, invertible). t-SNE/UMAP are for exploration and visualization.

---

## Neural Networks

**[M] Explain vanishing gradients and how to solve them.**

During backprop, gradients are multiplied repeatedly through layers. With activation functions like sigmoid/tanh (derivatives < 1), gradients shrink exponentially with depth, making early layers learn very slowly or not at all.

Solutions:
- **ReLU activation**: derivative is 1 for positive inputs, avoids saturation
- **Batch normalization**: normalizes layer inputs, keeps activations in healthy range
- **Residual connections**: skip connections allow gradients to flow directly to earlier layers (ResNet)
- **Careful initialization**: Xavier/He initialization keeps variance stable across layers
- **Gradient clipping**: for RNNs, clip large gradients to prevent explosion

**[M] What does batch normalization actually do? Why does it help?**

Batch norm normalizes each feature across the mini-batch to have zero mean and unit variance, then applies learned scale (γ) and shift (β) parameters.

Benefits:
- Reduces internal covariate shift (layer inputs changing distribution as weights update)
- Allows higher learning rates (activations stay in healthy range)
- Acts as a regularizer (adds slight noise from batch statistics)
- Reduces sensitivity to initialization

Caveat: batch norm behaves differently at train vs inference time (uses running mean/variance at inference). With very small batch sizes, use Layer Norm instead.

**[H] A training loss goes down but validation loss spikes after epoch 5. What do you investigate?**

1. **Classic overfitting**: reduce model capacity, add regularization, get more data
2. **Learning rate too high**: loss spikes after initial progress. Try reducing LR or using warmup
3. **Data leakage in validation set**: check for overlap or contamination
4. **Distribution shift between train/val**: check that splits are done correctly (especially for time-series data)
5. **Batch norm issue**: if using batch norm with small batch size or if validation batch size is 1
6. **Label noise in validation set**: inspect validation labels manually
7. **Gradient explosion**: look at gradient norms; add clipping

---

## Feature Engineering

**[M] How do you handle categorical variables with high cardinality (e.g., user ID with 10M values)?**

- **Target/mean encoding**: replace category with mean of target variable. Needs cross-validation to prevent leakage.
- **Embeddings**: learn dense representations (useful when the category has semantic meaning, e.g., product ID in a recommendation system)
- **Hashing**: hash to fixed-size buckets. Simple but causes collisions.
- **Frequency encoding**: replace with log(count) — useful when frequency correlates with target
- **Clustering**: group rare categories together, keep top K frequent ones

Avoid one-hot encoding for high cardinality — creates sparse, high-dimensional features that slow training.

**[M] What is data leakage and how do you detect it?**

Data leakage is when information from outside the training window (or from the target) flows into features, causing artificially high performance that doesn't hold in production.

Types:
- **Target leakage**: feature derived from the target variable (e.g., "was_refunded" to predict churn when the refund happens after churn)
- **Temporal leakage**: using future data to predict the past (not doing time-based train/test splits)
- **Train-test contamination**: preprocessing (e.g., scaling, imputation, encoding) fit on the full dataset before splitting

Detection:
- Suspiciously high performance (AUC > 0.99 on a hard problem)
- Feature importance shows unexpected features at top
- Performance drops significantly in production vs offline eval
- Manual review of feature definitions against event timestamps
