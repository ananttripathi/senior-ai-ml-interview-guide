# 12-Week Senior AI/ML Interview Study Plan

Assumes ~2–3 hours/day on weekdays, 4–5 hours on weekends. Adjust based on your baseline.

---

## Baseline Self-Assessment (Do This First)

Rate yourself 1–5 on each:

| Area | Self-Rating |
|------|-------------|
| ML fundamentals (bias-variance, regularization, tree models, etc.) | |
| Deep learning (backprop, CNN, RNN, Transformers) | |
| NLP / LLMs (BERT, GPT, fine-tuning, RAG) | |
| ML system design | |
| Statistics & A/B testing | |
| Python coding (LC medium) | |
| SQL | |
| Behavioral / storytelling | |

Spend extra time on anything rated ≤ 3.

---

## Week 1–2: ML Fundamentals + Statistics

### Goals
- Solid answers to any "explain X algorithm" question
- Comfortable with bias-variance, regularization, evaluation metrics
- Bayesian reasoning and A/B testing design

### Daily Plan (Week 1)
| Day | Topic | Resource |
|-----|-------|----------|
| Mon | Linear & logistic regression, gradient descent | CS229 Lecture 1–3 |
| Tue | Decision trees, random forests, bagging | Géron Ch. 6–7 |
| Wed | Gradient boosting (XGBoost, LightGBM) — internals | XGBoost paper |
| Thu | SVMs, kernel trick, regularization (L1/L2) | CS229 Lecture 7 |
| Fri | Evaluation: precision/recall/F1, ROC-AUC, calibration | Géron Ch. 3 |
| Sat | Practice: 10 ML fundamentals Q&A from `interview-qna/ml-fundamentals-qna.md` | |
| Sun | Review weak areas; write 3-sentence explanations of each topic | |

### Daily Plan (Week 2)
| Day | Topic | Resource |
|-----|-------|----------|
| Mon | Probability distributions, Bayes theorem, MLE/MAP | ESL Ch. 2 |
| Tue | Hypothesis testing, confidence intervals, p-values | Statistics textbook |
| Wed | A/B testing: design, CUPED, novelty effects, network effects | Kohavi's "Trustworthy Online Controlled Experiments" |
| Thu | Multi-armed bandits, Bayesian A/B testing | |
| Fri | Dimensionality reduction: PCA, t-SNE, UMAP | |
| Sat | Practice: 10 statistics Q&A from `interview-qna/statistics-probability-qna.md` | |
| Sun | Mock explain: pick 5 random topics, explain from scratch without notes | |

---

## Week 3–4: Deep Learning + NLP/LLMs

### Goals
- Explain backprop, optimizers, batch norm from scratch
- Understand Transformer architecture in detail
- Know BERT, GPT, fine-tuning, RAG, and LLM serving trade-offs

### Daily Plan (Week 3)
| Day | Topic | Resource |
|-----|-------|----------|
| Mon | Neural networks: forward pass, backprop, chain rule | CS229 Lecture 11 |
| Tue | Optimizers: SGD, momentum, Adam; learning rate schedules | FastAI Lesson 5 |
| Wed | CNNs: convolutions, pooling, ResNet, transfer learning | CS231n Lecture 5–7 |
| Thu | RNNs, LSTMs, GRUs — why they fail for long sequences | CS224N Lecture 6 |
| Fri | Batch norm, dropout, residual connections — intuition + math | Original papers |
| Sat | Implement a mini transformer from scratch (NumPy or PyTorch) | Andrej Karpathy's minGPT |
| Sun | Write explanations for 5 deep learning topics | |

### Daily Plan (Week 4)
| Day | Topic | Resource |
|-----|-------|----------|
| Mon | Transformer: self-attention, multi-head attention, positional encoding | "Attention Is All You Need" |
| Tue | BERT: masked LM, NSP, fine-tuning patterns | BERT paper |
| Wed | GPT family, InstructGPT, RLHF | InstructGPT paper |
| Thu | RAG: retrieval, chunking, embedding models, re-ranking | |
| Fri | LLM serving: quantization, speculative decoding, KV cache, vLLM | |
| Sat | Practice: design a document Q&A system end-to-end (RAG) | |
| Sun | LLM evaluation: BLEU, ROUGE, human eval, LLM-as-judge | |

---

## Week 5–6: ML System Design

### Goals
- Structured framework for any ML design question (NALM or equivalent)
- 6+ fully worked case studies
- Comfortable whiteboarding data flow, training pipeline, serving

### Framework (NALM)
1. **Need** — problem definition, constraints, success metrics
2. **Architecture** — data collection, features, model choice, training pipeline
3. **Learning** — offline evaluation, online evaluation, A/B test plan
4. **Monitoring** — data drift, model drift, feedback loops, retraining triggers

### Week 5 Case Studies
| Day | System | Key Challenge |
|-----|--------|---------------|
| Mon | YouTube recommendation (two-tower + ranking) | Scale, freshness, diversity |
| Tue | Ads click-through rate prediction | Low latency, calibration, position bias |
| Wed | Fraud detection | Class imbalance, adversarial shift, real-time vs batch |
| Thu | Search ranking (neural + BM25) | Query understanding, relevance vs freshness |
| Fri | Pinterest image similarity / visual search | Embedding retrieval, ANN, cold start |
| Sat | Review all 5 — identify where you got stuck | |
| Sun | Write up one system end-to-end in ML system design format | |

### Week 6 Case Studies
| Day | System | Key Challenge |
|-----|--------|---------------|
| Mon | Ride-share ETA prediction | Geographic features, real-time data |
| Tue | Content moderation (text + image) | False positives, human-in-loop |
| Wed | Personalized news feed ranking | Exploration-exploitation, engagement metrics |
| Thu | E-commerce demand forecasting | Temporal features, hierarchical forecasting |
| Fri | LLM-powered customer support chatbot | RAG, guardrails, latency |
| Sat | Mock design: "Design Netflix recommendations" (record yourself) | |
| Sun | Review recording; fill gaps in framework | |

---

## Week 7–8: Coding — LeetCode + SQL

### Goals
- Solve LC medium problems in <20 minutes
- Know key patterns cold: sliding window, two pointers, BFS/DFS, DP
- Complex SQL: window functions, CTEs, self-joins

### LeetCode Patterns (Week 7)
| Day | Pattern | Problems |
|-----|---------|----------|
| Mon | Arrays, hash maps | Two Sum, Group Anagrams, Top K Frequent |
| Tue | Sliding window, two pointers | Longest Substring Without Repeating, 3Sum |
| Wed | Trees: DFS/BFS | Max Depth, Level Order Traversal, LCA |
| Thu | Graphs: BFS/DFS/Union-Find | Number of Islands, Course Schedule |
| Fri | Binary search | Search in Rotated Array, Find Min in Rotated |
| Sat | Dynamic programming | Coin Change, Longest Common Subsequence |
| Sun | Timed practice: 3 mediums in 60 min | |

### ML-specific Coding (Week 8)
| Day | Topic | Problems |
|-----|-------|----------|
| Mon | Implement k-means from scratch | |
| Tue | Implement logistic regression + gradient descent | |
| Wed | Implement backprop for a 2-layer NN | |
| Thu | SQL: window functions, percentiles, cohort analysis | Mode Analytics SQL practice |
| Fri | Pandas: groupby, merge, pivot, time series | |
| Sat | Timed SQL practice: 5 problems in 45 min | |
| Sun | [Deep-ML.com](https://www.deep-ml.com/) — 10 ML coding problems | |

---

## Week 9: Company-Specific Prep

Pick your target companies. For each:
1. Read the company-specific file in `company-specific/`
2. Prepare STAR stories mapped to that company's values
3. Do 1 mock system design scoped to their products

### Amazon-specific (if targeting)
- Write out 25 STAR stories, tagged by Leadership Principle
- Practice: "Tell me about a time you disagreed with your manager" (Disagree & Commit)
- Practice: "Tell me about a time you made a decision with incomplete data" (Bias for Action)

### Meta-specific (if targeting)
- Practice "How would you measure success of [Facebook feature]?"
- Practice: "A metric dropped 10% — walk me through your investigation"

### Google-specific (if targeting)
- Study Googleyness: "Tell me about a time you had to work on an ambiguous problem"
- Deep ML design: expect follow-up questions on scale (billions of users)

---

## Week 10: Mock Interviews

### Schedule
| Day | Type | Format |
|-----|------|--------|
| Mon | ML fundamentals mock | Interviewing.io or peer |
| Tue | Coding mock | LeetCode mock interview |
| Wed | ML system design mock | Peer or Exponent |
| Thu | Behavioral mock | Peer, record and review |
| Fri | Full loop simulation | 4 back-to-back mocks |
| Sat | Rest | |
| Sun | Review all feedback; prioritize top 3 weaknesses | |

---

## Week 11: Targeted Weak Area Practice

Based on mock feedback, focus on your bottom 2–3 areas. Use this week to close gaps.

- Bad at system design depth? Do 3 more case studies with a timer.
- Slow on coding? Do 30 min timed sprints daily.
- Weak on behavioral? Record yourself; rewrite stories.

---

## Week 12: Light Review + Logistics

### Mon–Wed
- Review your personal Q&A summaries (don't learn new material)
- Re-read your strongest STAR stories
- Light coding — 1–2 warm-up problems/day, no pressure

### Thu–Fri
- Rest. Sleep 8+ hours.
- Logistics: confirm interview times, test your setup (Zoom/CoderPad)

### Interview Day
- Eat well, arrive early
- Ask clarifying questions before diving in
- Think out loud — interviewers want to see your reasoning
- It's OK to say "let me think for a moment"

---

## Tracking Template

Copy this weekly:

```
Week N checklist:
[ ] Day 1:
[ ] Day 2:
[ ] Day 3:
[ ] Day 4:
[ ] Day 5:
[ ] Weekend practice:
[ ] Weekly reflection: what was hardest? what improved?
```
