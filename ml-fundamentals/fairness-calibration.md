# ML Fairness, Calibration & Uncertainty

Increasingly probed in senior interviews at Google, Meta, and Apple. Mostly missing from pre-2023 guides. Expected for any model that makes consequential decisions (ads, credit, hiring, content moderation).

---

## Calibration

### What is calibration?

A model is **well-calibrated** if its predicted probabilities match empirical frequencies. If a model says "70% chance of rain" across 1000 forecasts, it should actually rain ~700 times.

### Why calibration matters more than you think

High AUC does NOT imply good calibration. A model with AUC = 0.95 can be badly miscalibrated — it correctly ranks positive vs. negative, but the absolute probabilities are wrong.

When calibration matters:
- **Expected value calculations**: pricing, bidding, risk scoring
- **Decision thresholds**: "flag if p > 0.8" only makes sense if p=0.8 really means 80% chance
- **Multi-model composition**: downstream systems that take your model's score as input
- **User-facing confidence**: "90% confidence" displayed to users

### Expected Calibration Error (ECE)

```python
def ece(y_true, y_pred_proba, n_bins=10):
    """
    y_true: binary labels (0/1)
    y_pred_proba: predicted probabilities
    """
    bins = np.linspace(0, 1, n_bins + 1)
    ece = 0.0
    n = len(y_true)

    for i in range(n_bins):
        mask = (y_pred_proba >= bins[i]) & (y_pred_proba < bins[i+1])
        if mask.sum() == 0:
            continue
        bin_accuracy = y_true[mask].mean()       # fraction truly positive in bin
        bin_confidence = y_pred_proba[mask].mean()  # average predicted prob in bin
        ece += (mask.sum() / n) * abs(bin_accuracy - bin_confidence)

    return ece
```

ECE < 0.05 is generally considered well-calibrated. Plot a **reliability diagram** (predicted probability vs. actual fraction positive per bin) to visualize calibration.

### Fixing miscalibration

**Platt Scaling**: fit a logistic regression on held-out calibration set, using model scores as input:
```python
from sklearn.linear_model import LogisticRegression
from sklearn.calibration import CalibratedClassifierCV

calibrated_model = CalibratedClassifierCV(base_model, method='sigmoid', cv='prefit')
calibrated_model.fit(X_calib, y_calib)
```

**Isotonic Regression**: non-parametric; fits a monotone step function. Better for large calibration sets. Can overfit with small sets.

**Temperature Scaling** (for neural networks): divide logits by a single learned scalar T before softmax:
```
p_calibrated = softmax(logits / T)
```
T > 1 → flatter distribution (more uncertain). T < 1 → sharper. Tune T on a held-out set.

---

## Fairness

### Why fairness matters in interviews

Models making decisions about people (ads targeting, loan approval, job recommendations, content moderation) must not discriminate based on protected attributes (race, gender, age, religion, etc.). Senior candidates are expected to know the metrics and their trade-offs.

### Fairness metrics (the unhappy truth: you can't satisfy all of them simultaneously)

**Group fairness metrics:**

| Metric | Definition | Formula |
|--------|-----------|---------|
| **Demographic Parity** | Equal positive prediction rates across groups | P(Ŷ=1\|A=0) = P(Ŷ=1\|A=1) |
| **Equalized Odds** | Equal TPR and FPR across groups | P(Ŷ=1\|Y=1,A=a) same for all a; same for FPR |
| **Equal Opportunity** | Equal TPR only (weaker than equalized odds) | P(Ŷ=1\|Y=1,A=0) = P(Ŷ=1\|Y=1,A=1) |
| **Calibration across groups** | Equal calibration per group | P(Y=1\|Ŷ=p,A=a) = p for all a |
| **Individual Fairness** | Similar individuals get similar predictions | d(x,x') small → d(f(x),f(x')) small |

### The impossibility theorem

Chouldechova (2017) and Kleinberg et al. (2016) proved that **demographic parity, equalized odds, and calibration across groups cannot all hold simultaneously** when base rates differ across groups. This is a fundamental trade-off, not a solvable engineering problem.

**Interview answer**: "These fairness definitions are mutually exclusive when base rates differ across groups. The choice of which fairness criterion to optimize is a business and ethical decision, not purely a technical one. I'd involve legal, policy, and affected communities in that decision."

### Measuring bias in practice

```python
from sklearn.metrics import confusion_matrix

def fairness_metrics(y_true, y_pred, group):
    """Compute TPR and FPR for each group."""
    results = {}
    for g in np.unique(group):
        mask = (group == g)
        tn, fp, fn, tp = confusion_matrix(y_true[mask], y_pred[mask]).ravel()
        results[g] = {
            'tpr': tp / (tp + fn),   # recall
            'fpr': fp / (fp + tn),
            'ppv': tp / (tp + fp),   # precision
            'positive_rate': (tp + fp) / mask.sum()
        }
    return results
```

### Mitigating bias

**Pre-processing** (fix the data):
- Reweighting: assign higher loss weight to underrepresented group-label combinations
- Resampling: oversample minority group × positive class combinations
- Data augmentation: generate synthetic examples for underrepresented intersections

**In-processing** (fix the training):
- Add fairness constraints to the loss function (e.g., equalized odds constraint via Lagrangian)
- Adversarial debiasing: train a predictor to predict Y while an adversary tries to predict protected attribute A from the predictions
- Fair representations: learn embeddings where protected attributes are not recoverable

**Post-processing** (fix the predictions):
- Adjust decision threshold per group to equalize TPR (equalized odds post-processing)
- Reject option classification: for uncertain predictions near the boundary, apply group-specific adjustments
- Calibrate per-group separately

### Intersectionality

Bias compounds at intersections of protected attributes. A model fair on gender and race separately may still discriminate against Black women as a subgroup. Always evaluate fairness metrics at intersections of protected attributes for high-stakes models.

---

## Uncertainty Quantification

Senior interviewers increasingly ask about knowing *when a model doesn't know*. Critical for high-stakes applications (medical diagnosis, autonomous driving, financial decisions).

### Types of uncertainty

**Aleatoric uncertainty** (irreducible): inherent randomness in the data. Even with infinite data, some prediction uncertainty remains.

**Epistemic uncertainty** (reducible): uncertainty due to limited data or model limitations. Can decrease with more data.

### Conformal Prediction

The practical, distribution-free way to produce prediction sets with guaranteed coverage.

For classification:
1. On a calibration set, compute **nonconformity score** per sample: `score_i = 1 - ŷ_i[y_i]` (1 minus the predicted probability of the true class)
2. Compute the (1-α) quantile of scores: `q̂ = quantile(scores, 1-α)`
3. At test time: prediction set = all classes y where `1 - ŷ[y] ≤ q̂`

**Guarantee**: for any new test sample, P(true label ∈ prediction set) ≥ 1-α, regardless of the model or data distribution.

```python
def conformal_prediction_set(calib_proba, calib_labels, test_proba, alpha=0.1):
    # Step 1: nonconformity scores on calibration set
    n = len(calib_labels)
    scores = 1 - calib_proba[np.arange(n), calib_labels]
    
    # Step 2: (1-alpha) quantile with finite-sample correction
    q_hat = np.quantile(scores, np.ceil((n+1)*(1-alpha))/n)
    
    # Step 3: prediction set for test samples
    prediction_sets = test_proba >= (1 - q_hat)
    return prediction_sets
```

**Why it matters for interviews**: conformal prediction provides rigorous uncertainty guarantees without any distributional assumptions. For risk-sensitive systems (medical, legal), this is the production-ready approach to uncertainty.

### Bayesian Approaches

**Monte Carlo Dropout** (Gal & Ghahramani 2016): keep dropout active at test time; run N forward passes; variance of predictions = epistemic uncertainty.

```python
def mc_dropout_uncertainty(model, X, n_samples=50):
    model.train()  # keep dropout active
    preds = np.array([model(X).detach().numpy() for _ in range(n_samples)])
    return preds.mean(axis=0), preds.var(axis=0)  # mean prediction, uncertainty
```

**Deep Ensembles**: train N independent models with different initializations; disagreement = uncertainty. More expensive but more reliable than MC dropout.

---

## Interview Questions

**[M] "Your fraud model has higher false positive rate for a specific demographic. What do you do?"**

1. **Quantify**: measure TPR, FPR, PPV per demographic group; compute ECE per group
2. **Root cause**: is the training data underrepresented for this group? Are features proxying for protected attributes? Is the base rate different?
3. **Business/legal consultation**: determine which fairness criterion is appropriate (legal requirements may apply)
4. **Technical remediation**: per-group threshold calibration as a fast fix; long-term: rebalanced training data, fairness constraints, audit the feature set for proxy discrimination
5. **Monitor ongoing**: add per-group fairness metrics to production monitoring dashboard

**[H] "How would you audit a model for fairness before deploying it?"**

Pre-deployment fairness audit:
1. Define which protected attributes and which fairness criteria are in scope (legal + ethical consultation)
2. Ensure test set has sufficient representation of all protected groups and intersections
3. Measure: demographic parity, equalized odds, calibration across groups, individual fairness (similarity-based)
4. Slice-based analysis: evaluate performance across all meaningful subgroups
5. Feature audit: identify features that proxy for protected attributes (feature importance + correlation analysis)
6. Adverse impact analysis: 4/5ths rule — if positive rate for protected group < 80% of highest-rate group, flag for legal review
7. Document findings; establish ongoing monitoring with the same fairness metrics in production
