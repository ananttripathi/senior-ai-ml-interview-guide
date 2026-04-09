# Google — ML/DS/AI Interview Guide

Roles: ML Engineer (SWE-ML track), Research Scientist, Applied Scientist, Data Scientist, AI Engineer

Levels: L4 (junior), **L5 (senior — most common target)**, L6 (staff), L7 (principal)

---

## Interview Structure

Typically 5–6 onsite rounds (virtual or in-person) after 1–2 phone screens.

| Round | Type | Notes |
|-------|------|-------|
| Recruiter screen | Background + fit | 30 min; basic ML background |
| Technical phone screen 1 | ML fundamentals | 45–60 min |
| Technical phone screen 2 | Coding (LC medium) | 45 min (SWE-ML track) |
| Onsite 1 | ML system design | 45 min; deep dive |
| Onsite 2 | Coding (LC medium–hard) | 45 min |
| Onsite 3 | ML depth / research | 45 min; theory + derivations |
| Onsite 4 | Googleyness + Leadership | 45 min; behavioral |
| Onsite 5 | Second coding or second ML design | 45 min |

---

## What Google Uniquely Tests

### 1. Breadth-first ML knowledge
Google interviewers are known for **unexpected, novel questions** — not just the standard interview playbook. They probe breadth: "When would you use DBSCAN over k-means?", "What's the VC dimension of a linear classifier in d dimensions?", "Derive the EM update equations for a GMM."

**Implication**: don't just grind company-tagged LeetCode. Study broadly across all ML topics.

### 2. Math depth
Google expects you to derive things, not just name them. Expect:
- "Derive the gradient of cross-entropy loss with respect to softmax inputs"
- "What's the closed-form solution for linear regression? When does it not exist?"
- "Prove that the optimal Bayes classifier minimizes the 0-1 loss"

**Prep**: be able to derive the gradient for your 5 most important loss functions. Know linear algebra (eigendecomposition, SVD), probability (MLE/MAP derivations), and calculus.

### 3. ML system design at Google scale
Google designs for billions of users. When asked "Design YouTube recommendations," the answer must account for:
- YouTube has 800M+ videos, 2B+ users, 500 hours uploaded per minute
- Serving: < 100ms latency globally
- Training: petabyte-scale data; distributed training across thousands of TPUs
- Freshness: trending videos need to appear in recommendations within minutes

**Implication**: always discuss scale explicitly. "How does your design change at 100× scale?" is a near-certain follow-up.

### 4. Googleyness (behavioral)
Google evaluates a specific set of behavioral attributes:
- **Comfort with ambiguity**: working on problems without clear requirements
- **Learning from failure**: intellectual honesty about mistakes
- **Collaborative**: cross-functional influence, supporting teammates
- **Emergent leadership**: taking initiative without being asked
- **Innovation**: proposing new approaches, not just executing assigned tasks

### 5. Coding quality
Google's coding bar is high. They expect:
- Clean, production-quality code (not just "it works")
- Correct time and space complexity analysis
- Edge case handling discussed upfront
- LC hard difficulty for L6+; LC medium–hard for L5

---

## Google-Specific ML Topics

### TFX (TensorFlow Extended)
Google's ML production platform. Know the components:
- **ExampleGen**: ingests and splits data
- **StatisticsGen + SchemaGen**: data validation
- **Transform**: feature engineering (runs same code in training and serving — prevents train-serve skew)
- **Trainer**: model training
- **Evaluator**: model evaluation with fairness metrics
- **Pusher**: deployment to serving

### Google's ML system papers (know the key ideas)
- **MapReduce** (Dean & Ghemawat 2004): foundation of distributed ML training
- **Bigtable** (Chang et al. 2006): feature store precursor
- **DistBelief / TensorFlow**: distributed deep learning systems
- **Wide & Deep** (Cheng et al. 2016): recommendation system architecture
- **DCN v2** (Wang et al. 2021): deep cross network for feature interactions
- **BERT** (Devlin et al. 2018): Google's foundational LLM
- **LaMDA / Bard / Gemini**: Google's LLM progression

### TPUs
Google trains models on TPUs (Tensor Processing Units), not GPUs. At L5/L6:
- Know what a TPU is and why it's designed differently from a GPU (systolic array, designed for matrix multiply throughput not general compute)
- Know the tradeoffs: TPUs are excellent for large, fixed-size matrix operations; GPUs are more flexible

---

## Googleyness Stories — What Works

**Comfort with ambiguity**: "I was given a vague project to 'improve search quality.' I started by defining what 'quality' meant quantitatively, ran a user study to identify the top 3 pain points, and proposed a 3-month roadmap. Nobody had done that scoping before."

**Collaborative**: "I noticed a colleague was blocked on a data pipeline issue that I had solved before. I volunteered 3 hours of my time to pair-program with them even though it wasn't my project. The fix unblocked their quarterly deadline."

**Innovation at scale**: "Standard NER models couldn't handle our proprietary product catalog. I proposed using a zero-shot classification approach with instruction-tuned embeddings. Saved 6 weeks of annotation work and achieved similar F1."

---

## The Google Loop — Tips

1. **Expect follow-up questions on every answer**. Interviewers at Google are trained to probe deeper. If you say "I'd use XGBoost," expect "Why not LightGBM? What would change your answer?"

2. **Draw diagrams for system design**. Google interviewers expect a whiteboard-level architecture diagram within the first 5 minutes of a design question.

3. **Think out loud about scale**. Unprompted, mention "at Google scale, this approach would need to handle 10B events/day, which changes the architecture because..."

4. **Coding**: write pseudocode first, then code. Say your complexity before writing. Clean up your code before time runs out.

5. **Google uses a committee hiring system**: each interviewer scores you independently, then a hiring committee reviews all feedback. You can't win over one interviewer and compensate for a bad round — you need to be consistently strong across all rounds.

---

## Common Google ML Interview Questions (Reported)

- "Implement k-means from scratch in Python (NumPy only)"
- "How would you design the YouTube watch-next recommendation system at scale?"
- "Explain the attention mechanism. Why does it work better than RNNs for translation?"
- "What's the difference between L1 and L2 regularization? When would you use each?"
- "Design a system to detect fake news articles in real-time"
- "You have a dataset with 99% negatives. How do you train a reliable model?"
- "A model trained in January performs poorly in July. What are possible causes and how would you investigate?"
- "Implement binary search. Now make it work on a rotated sorted array."
- "Given a 10B-row table, how would you compute the 95th percentile efficiently?"

---

## Resources Specific to Google Prep

- **Google's ML Crash Course** (free): developers.google.com/machine-learning — covers TF and Google's ML philosophy
- **Rules of Machine Learning** (Martin Zinkevich, Google): 43 practical rules for production ML from Google's experience — must read
- **Designing ML Systems** (Chip Huyen): covers TFX and Google-style system design
- **Leetcode Google tag**: sort by frequency, focus on medium difficulty
