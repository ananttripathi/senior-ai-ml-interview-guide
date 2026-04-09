# ML Coding from Scratch

A unique interview format at MAANG: implement ML algorithms using only NumPy (no sklearn). Reported at Google, Uber, Meta, and Apple for ML Engineer and Applied Scientist roles.

This file has complete, interview-ready implementations you should be able to write from memory.

---

## Gradient Descent

```python
import numpy as np

def gradient_descent(X, y, lr=0.01, n_iters=1000):
    """
    Linear regression via gradient descent.
    X: (n_samples, n_features)
    y: (n_samples,)
    Returns: weights w (n_features,), bias b (scalar)
    """
    n, d = X.shape
    w = np.zeros(d)
    b = 0.0

    for _ in range(n_iters):
        y_pred = X @ w + b          # (n,)
        residual = y_pred - y       # (n,)

        dw = (2 / n) * X.T @ residual   # (d,)
        db = (2 / n) * residual.sum()   # scalar

        w -= lr * dw
        b -= lr * db

    return w, b
```

**Follow-up**: "Add L2 regularization."
```python
dw = (2 / n) * X.T @ residual + 2 * lambda_ * w  # add regularization term
# bias is not regularized
```

**Follow-up**: "Implement mini-batch SGD."
```python
for epoch in range(n_epochs):
    indices = np.random.permutation(n)
    for start in range(0, n, batch_size):
        batch_idx = indices[start:start+batch_size]
        X_batch, y_batch = X[batch_idx], y[batch_idx]
        # ... same gradient computation on batch
```

---

## Logistic Regression

```python
def sigmoid(z):
    return 1 / (1 + np.exp(-z))

class LogisticRegression:
    def __init__(self, lr=0.01, n_iters=1000):
        self.lr = lr
        self.n_iters = n_iters
        self.w = None
        self.b = None

    def fit(self, X, y):
        n, d = X.shape
        self.w = np.zeros(d)
        self.b = 0.0

        for _ in range(self.n_iters):
            z = X @ self.w + self.b
            y_pred = sigmoid(z)         # predicted probabilities

            # Binary cross-entropy gradient
            dw = (1 / n) * X.T @ (y_pred - y)
            db = (1 / n) * (y_pred - y).sum()

            self.w -= self.lr * dw
            self.b -= self.lr * db

    def predict_proba(self, X):
        return sigmoid(X @ self.w + self.b)

    def predict(self, X, threshold=0.5):
        return (self.predict_proba(X) >= threshold).astype(int)
```

**Interview note**: the gradient of binary cross-entropy loss w.r.t. weights is elegantly `X.T @ (y_hat - y) / n` — interviewers expect you to derive this, not just write it.

---

## K-Means Clustering

```python
def kmeans(X, k, n_iters=100, tol=1e-4):
    """
    X: (n_samples, n_features)
    Returns: centroids (k, n_features), labels (n_samples,)
    """
    n, d = X.shape

    # k-means++ initialization
    centroids = [X[np.random.randint(n)]]
    for _ in range(k - 1):
        dists = np.array([min(np.linalg.norm(x - c)**2 for c in centroids) for x in X])
        probs = dists / dists.sum()
        centroids.append(X[np.random.choice(n, p=probs)])
    centroids = np.array(centroids)

    for _ in range(n_iters):
        # Assignment step
        dists = np.linalg.norm(X[:, None] - centroids[None], axis=2)  # (n, k)
        labels = dists.argmin(axis=1)   # (n,)

        # Update step
        new_centroids = np.array([X[labels == i].mean(axis=0) for i in range(k)])

        if np.linalg.norm(new_centroids - centroids) < tol:
            break
        centroids = new_centroids

    return centroids, labels
```

**Key details interviewers probe:**
- k-means++ initialization (not random): why? reduces number of iterations, avoids bad initializations
- Handling empty clusters (a centroid with no assigned points) — reinitialize it randomly
- Convergence criterion: centroid movement < tol, or label assignments unchanged

---

## Backpropagation (2-Layer Neural Net)

```python
def relu(z):
    return np.maximum(0, z)

def relu_grad(z):
    return (z > 0).astype(float)

class TwoLayerNet:
    def __init__(self, input_dim, hidden_dim, output_dim, lr=0.01):
        # He initialization for ReLU
        self.W1 = np.random.randn(input_dim, hidden_dim) * np.sqrt(2 / input_dim)
        self.b1 = np.zeros(hidden_dim)
        self.W2 = np.random.randn(hidden_dim, output_dim) * np.sqrt(2 / hidden_dim)
        self.b2 = np.zeros(output_dim)
        self.lr = lr

    def forward(self, X):
        self.X = X
        self.z1 = X @ self.W1 + self.b1        # (n, hidden)
        self.a1 = relu(self.z1)                  # (n, hidden)
        self.z2 = self.a1 @ self.W2 + self.b2   # (n, output)
        return self.z2  # raw logits

    def mse_loss(self, y_pred, y):
        return ((y_pred - y) ** 2).mean()

    def backward(self, y_pred, y):
        n = y_pred.shape[0]

        # Output layer gradient
        dL_dz2 = 2 * (y_pred - y) / n          # (n, output)

        dW2 = self.a1.T @ dL_dz2               # (hidden, output)
        db2 = dL_dz2.sum(axis=0)               # (output,)

        # Hidden layer gradient (chain rule through ReLU)
        da1 = dL_dz2 @ self.W2.T               # (n, hidden)
        dz1 = da1 * relu_grad(self.z1)         # (n, hidden)

        dW1 = self.X.T @ dz1                   # (input, hidden)
        db1 = dz1.sum(axis=0)                  # (hidden,)

        # Update
        self.W2 -= self.lr * dW2
        self.b2 -= self.lr * db2
        self.W1 -= self.lr * dW1
        self.b1 -= self.lr * db1
```

**What interviewers listen for:**
- You store intermediate values (z1, a1) in the forward pass for use in backward pass
- Chain rule application: `dL/dz1 = (dL/da1) * (da1/dz1)` where `da1/dz1 = relu'(z1)`
- Matrix dimension bookkeeping (most common mistake)

---

## AUC-ROC from Scratch

Reported as a real interview question at Uber and Google.

```python
def compute_auc(y_true, y_scores):
    """
    Compute AUC-ROC without sklearn.
    Uses the trapezoidal rule on the ROC curve.
    """
    # Sort by descending score
    desc_idx = np.argsort(y_scores)[::-1]
    y_true_sorted = y_true[desc_idx]

    n_pos = y_true.sum()
    n_neg = len(y_true) - n_pos

    tps, fps = [], []
    tp = fp = 0
    for label in y_true_sorted:
        if label == 1:
            tp += 1
        else:
            fp += 1
        tps.append(tp / n_pos)   # TPR
        fps.append(fp / n_neg)   # FPR

    # Add origin
    tprs = np.array([0] + tps)
    fprs = np.array([0] + fps)

    # Trapezoidal integration
    auc = np.trapz(tprs, fprs)
    return auc
```

**Alternative**: AUC equals the probability that a random positive scores higher than a random negative. This gives an O(n²) implementation:
```python
def auc_pairwise(y_true, y_scores):
    pos_scores = y_scores[y_true == 1]
    neg_scores = y_scores[y_true == 0]
    count = sum(p > n for p in pos_scores for n in neg_scores)
    count += 0.5 * sum(p == n for p in pos_scores for n in neg_scores)
    return count / (len(pos_scores) * len(neg_scores))
```

---

## Precision, Recall, F1 from Scratch

```python
def precision_recall_f1(y_true, y_pred):
    tp = ((y_pred == 1) & (y_true == 1)).sum()
    fp = ((y_pred == 1) & (y_true == 0)).sum()
    fn = ((y_pred == 0) & (y_true == 1)).sum()

    precision = tp / (tp + fp) if (tp + fp) > 0 else 0.0
    recall    = tp / (tp + fn) if (tp + fn) > 0 else 0.0
    f1 = (2 * precision * recall / (precision + recall)
          if (precision + recall) > 0 else 0.0)

    return precision, recall, f1
```

---

## Softmax + Cross-Entropy Loss

```python
def softmax(z):
    # Subtract max for numerical stability
    e = np.exp(z - z.max(axis=1, keepdims=True))
    return e / e.sum(axis=1, keepdims=True)

def cross_entropy_loss(y_pred_proba, y_true_onehot):
    """
    y_pred_proba: (n, C) — softmax probabilities
    y_true_onehot: (n, C) — one-hot labels
    """
    # Clip for numerical stability (avoid log(0))
    y_pred_proba = np.clip(y_pred_proba, 1e-9, 1 - 1e-9)
    return -(y_true_onehot * np.log(y_pred_proba)).sum(axis=1).mean()

def softmax_cross_entropy_grad(y_pred_proba, y_true_onehot):
    """Gradient of cross-entropy loss w.r.t. pre-softmax logits z."""
    # The gradient simplifies elegantly to (y_hat - y) / n
    n = y_pred_proba.shape[0]
    return (y_pred_proba - y_true_onehot) / n
```

---

## Cosine Similarity & Vector Search

```python
def cosine_similarity(a, b):
    return np.dot(a, b) / (np.linalg.norm(a) * np.linalg.norm(b) + 1e-9)

def batch_cosine_similarity(query, corpus):
    """
    query: (d,)
    corpus: (n, d)
    Returns similarity scores (n,)
    """
    query_norm = query / (np.linalg.norm(query) + 1e-9)
    corpus_norm = corpus / (np.linalg.norm(corpus, axis=1, keepdims=True) + 1e-9)
    return corpus_norm @ query_norm

def top_k_similar(query, corpus, k=5):
    scores = batch_cosine_similarity(query, corpus)
    top_k_idx = np.argpartition(scores, -k)[-k:]
    return top_k_idx[np.argsort(scores[top_k_idx])[::-1]]
```

---

## Decision Tree Split (Information Gain)

```python
def entropy(y):
    counts = np.bincount(y)
    probs = counts[counts > 0] / len(y)
    return -(probs * np.log2(probs)).sum()

def information_gain(y, y_left, y_right):
    n = len(y)
    weighted_entropy = (len(y_left) / n) * entropy(y_left) + \
                       (len(y_right) / n) * entropy(y_right)
    return entropy(y) - weighted_entropy

def best_split(X, y):
    best_ig, best_feature, best_threshold = -1, None, None
    for feature in range(X.shape[1]):
        thresholds = np.unique(X[:, feature])
        for threshold in thresholds:
            left_mask = X[:, feature] <= threshold
            if left_mask.sum() == 0 or (~left_mask).sum() == 0:
                continue
            ig = information_gain(y, y[left_mask], y[~left_mask])
            if ig > best_ig:
                best_ig, best_feature, best_threshold = ig, feature, threshold
    return best_feature, best_threshold
```

---

## Interview Tips for ML Coding

1. **State your approach first**: "I'll implement gradient descent with vectorized NumPy operations. Time complexity is O(n·d) per iteration."

2. **Handle edge cases out loud**: "I'm clipping probabilities to avoid log(0). I'm using numerical stability tricks for softmax."

3. **Name dimensions**: write `# (n, d)` comments next to matrix operations — shows you're tracking shapes.

4. **Test with a small example**: after writing, plug in a 3×2 matrix and trace through manually.

5. **Know the elegant gradients**:
   - Linear regression MSE → `X.T @ (y_hat - y) / n`
   - Logistic regression cross-entropy → `X.T @ (y_hat - y) / n` (same form!)
   - Softmax + cross-entropy → `(y_hat - y) / n`

6. **Common mistakes to avoid**:
   - Forgetting `/n` normalization in gradient
   - Using Python loops instead of vectorized NumPy (interviewer will ask you to optimize)
   - Not initializing weights properly (He for ReLU, Xavier for tanh/sigmoid)
