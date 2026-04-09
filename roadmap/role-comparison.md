# Role Comparison: Data Scientist vs ML Engineer vs AI Engineer

Understanding the distinction helps you target the right role and tailor your preparation.

---

## The Core Difference

| | Data Scientist | ML Engineer | AI Engineer |
|--|---------------|-------------|-------------|
| **Primary output** | Insights, models, experiments | Production ML systems | AI-powered product features |
| **Closest analog** | Applied researcher / analyst | Software engineer who does ML | Software engineer building with AI APIs/models |
| **Coding bar** | Medium (Python + SQL) | High (SWE-level algorithms + systems) | High (SWE-level + LLM/API patterns) |
| **Statistics bar** | High | Medium | Medium |
| **ML theory bar** | High | Medium-High | Medium |
| **Systems bar** | Low-Medium | High | High |
| **Product intuition** | High (Meta, Netflix DS) | Medium | High |

---

## Data Scientist

### What you actually do
- Define metrics and success criteria for product decisions
- Run A/B tests and analyze results
- Build models to power products (ranking, recommendations, forecasting)
- Partner with PMs and engineers to translate business questions into ML problems

### Interview focus
- Statistics and experimentation: hypothesis testing, A/B testing design, causal inference
- SQL: complex queries, funnel analysis, cohort analysis
- Python: data manipulation, model training (sklearn/xgboost)
- ML concepts: feature engineering, model selection, evaluation
- Product sense: how does your model/metric connect to user value?

### Companies that hire DS separately from MLE
- Meta (strong DS culture; DS owns metrics and experimentation)
- Netflix (DS does both analysis and modeling)
- Google (splits into DS, Applied Scientist, and MLE tracks)
- Amazon (Applied Scientist is the research-heavy DS analog)

---

## ML Engineer

### What you actually do
- Build and own the full ML pipeline: data ingestion → training → evaluation → serving
- Write production-quality code that runs at scale
- Optimize models for latency, throughput, and cost
- Collaborate with DS (who might own the model) and platform engineers

### Interview focus
- Coding: LC medium–hard, data structures, algorithms
- ML system design: feature stores, training pipelines, model serving, monitoring
- ML concepts: enough to implement and debug models
- Distributed systems basics: Spark, data pipelines, streaming (Kafka/Flink)
- Model optimization: quantization, distillation, batching strategies

### Differentiator from DS
You will be expected to write code like a software engineer. Expect LC-style coding rounds identical to SWE interviews at the same level.

---

## AI Engineer

### What you actually do
- Build products and features powered by LLMs and foundation models
- Design prompting strategies, fine-tuning pipelines, RAG systems
- Integrate AI APIs (OpenAI, Anthropic, Gemini, internal models) into applications
- Own reliability, latency, and cost of AI-powered features

### Interview focus
- LLM fundamentals: Transformer architecture, fine-tuning, RAG, prompt engineering
- System design: LLM serving, evaluation pipelines, guardrails, vector databases
- Coding: SWE-level, often with Python async patterns, API design
- Product: how to measure if an AI feature is "working"
- Emerging: multimodal systems, agentic systems, tool use

### This role is newer
Many companies don't have a formal "AI Engineer" title yet — you'll find this work under ML Engineer, Applied AI, or Research Engineer. The fastest-growing role track as of 2025–2026.

---

## MAANG Role Matrix

| Company | Data Scientist | ML Engineer | AI/Applied AI |
|---------|---------------|-------------|---------------|
| **Google** | Data Scientist, Applied Scientist | ML Engineer, Software Engineer (ML) | Research Engineer, AI Engineer |
| **Meta** | Data Scientist (strong; owns metrics) | ML Engineer | Research Engineer |
| **Amazon** | Data Scientist | ML Engineer | Applied Scientist (research-heavy) |
| **Apple** | Data Scientist | ML Engineer | AI/ML Engineer (often merged) |
| **Netflix** | Data Scientist (modeling + analysis) | ML Engineer | (often within DS/MLE track) |

---

## Which Role Should You Target?

### Target Data Scientist if:
- You love experimentation and causal inference
- SQL and Python analysis is your comfort zone
- You want to directly influence product strategy
- You prefer breadth of business questions over depth of systems

### Target ML Engineer if:
- You have strong software engineering fundamentals
- You enjoy building reliable, scalable systems
- You want to own the full lifecycle from data to production
- Algorithms and coding come naturally

### Target AI Engineer if:
- You're excited about LLMs and foundation models
- You want to build AI-native products
- You're comfortable with fast-changing landscape and ambiguity
- You have strong SWE skills + ML intuition

---

## Leveling Guide

### What "Senior" means at each company

**L5 at Google / E5 at Meta / L5 at Amazon / ICT4 at Apple / Senior at Netflix:**
- Owns projects end-to-end with minimal guidance
- Influences technical direction of team
- Mentors junior engineers
- Communicates clearly with cross-functional partners

**L6 at Google / E6 at Meta / L6 at Amazon / ICT5 at Apple / Staff at Netflix:**
- Owns multi-team technical direction
- Drives org-wide technical decisions
- Operates with significant ambiguity
- Business impact is clear and measurable

At senior level, every interview question has an implicit "at scale" dimension. Always ask: what changes when this system handles 10x the load or 1000x more data?
