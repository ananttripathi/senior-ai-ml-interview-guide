# Senior AI/ML/Data Science Interview Guide — MAANG

> A comprehensive, opinionated study guide for cracking **Senior Data Scientist**, **AI Engineer**, and **ML Engineer** roles at Meta, Apple, Amazon, Netflix, and Google.

---

## Who This Is For

Engineers and scientists with 5+ years of experience targeting senior-level roles (L5/L6 at Google, E5/E6 at Meta, L5/L6 at Amazon, ICT4/ICT5 at Apple, Senior/Staff at Netflix).

---

## Repository Structure

```
.
├── roadmap/
│   ├── 12-week-plan.md          # Week-by-week study schedule
│   ├── role-comparison.md       # DS vs AI Eng vs ML Eng — scope & expectations
│   └── company-breakdown.md     # Interview format per company
│
├── ml-fundamentals/
│   ├── supervised-learning.md
│   ├── unsupervised-learning.md
│   ├── deep-learning.md
│   ├── nlp.md
│   ├── computer-vision.md
│   ├── reinforcement-learning.md
│   ├── probability-statistics.md
│   └── feature-engineering.md
│
├── system-design/
│   ├── ml-system-design-framework.md
│   ├── recommendation-systems.md
│   ├── search-ranking.md
│   ├── ads-ranking.md
│   ├── fraud-detection.md
│   ├── nlp-systems.md
│   └── real-time-ml.md
│
├── coding/
│   ├── python-ml-patterns.md
│   ├── sql-for-data-science.md
│   ├── leetcode-strategy.md
│   └── pandas-numpy-cheatsheet.md
│
├── interview-qna/
│   ├── ml-fundamentals-qna.md
│   ├── statistics-probability-qna.md
│   ├── system-design-qna.md
│   ├── coding-qna.md
│   ├── product-sense-qna.md
│   └── behavioral-qna.md
│
├── company-specific/
│   ├── google/
│   ├── meta/
│   ├── amazon/
│   ├── apple/
│   └── netflix/
│
└── resources/
    ├── books.md
    ├── papers.md
    ├── courses.md
    └── mock-interview-checklist.md
```

---

## Interview Structure by Company

### Google (L5/L6)
| Round | Type | Duration |
|-------|------|----------|
| Recruiter screen | Background fit | 30 min |
| Technical phone screen | ML concepts + coding | 45 min |
| Onsite × 4–5 | ML design, coding (LC med/hard), Googleyness | 45 min each |

**Focus areas:** ML design (very deep), coding (data structures + algorithms), Googleyness behavioral. Expect questions on large-scale ML systems (YouTube, Search, Ads).

### Meta (E5/E6)
| Round | Type | Duration |
|-------|------|----------|
| Recruiter screen | Background | 30 min |
| Technical screen | ML concepts / coding | 45 min |
| Onsite × 4–5 | ML design, coding, product sense, behavioral | 45 min each |

**Focus areas:** Product sense is unique to Meta — connecting ML to business outcomes. Behavioral uses the STAR method against Meta's core values. Coding is LC medium.

### Amazon (L5/L6)
| Round | Type | Duration |
|-------|------|----------|
| Online assessment | Coding × 2 | 90 min |
| Phone screen | ML/technical | 60 min |
| Onsite (loop) × 5–7 | ML design, coding, LPs | 60 min each |

**Focus areas:** Leadership Principles (LPs) are **mandatory in every round** — each interviewer owns 2–3 LPs. ML system design for real Amazon products (recommendations, fraud, forecasting). Coding is LC easy–medium.

### Apple (ICT4/ICT5)
| Round | Type | Duration |
|-------|------|----------|
| Recruiter screen | Background | 30 min |
| Hiring manager call | Role fit | 45 min |
| Technical screens × 2 | ML + coding | 60 min each |
| Onsite × 5–6 | ML breadth, coding, domain expertise | 60 min each |

**Focus areas:** Very team-specific — research roles go deep on papers; product roles emphasize on-device ML, privacy-preserving techniques (differential privacy, federated learning). Coding is LC medium.

### Netflix (Senior/Staff)
| Round | Type | Duration |
|-------|------|----------|
| Recruiter screen | Background + culture | 30 min |
| Hiring manager screen | Role fit | 45 min |
| Technical screens × 2–3 | ML + coding or case study | 60 min each |
| Onsite × 4–5 | ML design, coding, business case | 60 min each |

**Focus areas:** Culture fit ("keeper test") is real — expect explicit cultural fit evaluation. ML design for recommendation and personalization. Compensation discussion is unusually transparent early.

---

## Core Technical Topics

### 1. Machine Learning Fundamentals
- Bias-variance tradeoff, overfitting, regularization (L1/L2/dropout)
- Supervised: linear/logistic regression, trees, boosting (XGBoost/LightGBM), SVMs, KNN
- Unsupervised: k-means, DBSCAN, PCA, t-SNE/UMAP, autoencoders
- Deep learning: backprop, optimizers (Adam/SGD), batch norm, residual connections
- Training tricks: learning rate schedules, early stopping, mixed precision, gradient clipping

### 2. Statistics & Probability
- Probability distributions, Bayes theorem, MLE/MAP
- Hypothesis testing, p-values, Type I/II errors, power analysis
- A/B testing design: CUPED, novelty effect, network effects, switchback tests
- Bayesian A/B testing, multi-armed bandits

### 3. ML System Design
- The NALM framework: **N**eed → **A**rchitecture → **L**earning → **M**onitoring
- Feature stores, training pipelines, model serving (latency vs throughput)
- Offline vs online evaluation; data freshness, feedback loops
- Embeddings, approximate nearest neighbors (FAISS, ScaNN)
- Model calibration, fairness, and responsible AI

### 4. NLP / LLMs
- Transformers, attention mechanisms, BERT/GPT family
- Fine-tuning vs prompting vs RAG
- Evaluation: BLEU, ROUGE, perplexity, human eval
- LLM serving: quantization, speculative decoding, KV cache

### 5. Coding (Python-first)
- Array/string manipulation, hash maps, two pointers, sliding window
- Trees, graphs (BFS/DFS), dynamic programming
- SQL: window functions, CTEs, aggregations, joins
- Pandas: groupby, merge, apply, vectorized operations

### 6. Product Sense (especially Meta, Netflix)
- Defining the right metric; north star vs guardrail metrics
- Trade-off analysis: precision vs recall, revenue vs user experience
- Experiment design and interpreting results

### 7. Behavioral
- STAR format for all stories (Situation, Task, Action, Result)
- Required for Amazon LPs: Ownership, Customer Obsession, Dive Deep, Invent & Simplify, etc.
- Google "Googleyness": ambiguity, cross-functional influence, learning from failure

---

## 12-Week Study Plan (Quick View)

| Week | Focus |
|------|-------|
| 1–2 | ML fundamentals review + statistics |
| 3–4 | Deep learning, NLP, transformers/LLMs |
| 5–6 | ML system design (2 case studies/week) |
| 7–8 | Coding — LeetCode patterns, SQL |
| 9 | Company-specific prep (LPs, product sense) |
| 10 | Mock interviews × 4 |
| 11 | Weak area targeted practice |
| 12 | Light review, rest, logistics |

Full week-by-week plan: [roadmap/12-week-plan.md](roadmap/12-week-plan.md)

---

## Key Resources

### Books
- *Designing Machine Learning Systems* — Chip Huyen (system design bible)
- *Hands-On ML with Scikit-Learn, Keras & TensorFlow* — Géron
- *The Elements of Statistical Learning* — Hastie et al. (theory depth)
- *ML Engineering* — Andriy Burkov

### Courses
- Stanford CS229 (ML fundamentals) — available free on YouTube
- Stanford CS224N (NLP with Deep Learning)
- FastAI Practical Deep Learning
- Chip Huyen's MLOps course (mlops.community)

### Interview-specific
- [ML Interview Book](https://huyenchip.com/ml-interviews-book/) — Chip Huyen (free)
- [Machine Learning System Design Interview](https://www.educative.io/courses/machine-learning-system-design) — Educative
- [Deep ML](https://www.deep-ml.com/) — coding problems specific to ML
- [Exponent](https://www.tryexponent.com/) — mock interviews

### Papers to Know
- Attention Is All You Need (Transformers)
- BERT, GPT-2/3, InstructGPT
- Deep & Cross Network (feature interactions for ads)
- Wide & Deep Learning (Google Play recommendations)
- Neural Collaborative Filtering

---

## Role Comparison at a Glance

| Dimension | Data Scientist | ML Engineer | AI Engineer |
|-----------|---------------|-------------|-------------|
| Primary skill | Analysis + modeling | Scalable ML systems | Applied AI/LLM products |
| Coding bar | Medium (SQL + Python) | High (SWE-level) | High (SWE-level) |
| Stats bar | High | Medium | Medium |
| System design | ML-focused | Infrastructure-heavy | LLM/API-heavy |
| Deliverable | Insights + models | Production ML pipelines | AI-powered features |

---

## Contributing

PRs welcome. Each file should follow the template in [resources/template.md](resources/template.md).

---

*Last updated: April 2026*
