# Company-by-Company Interview Strategy

Use this alongside the detailed guides in `company-specific/`. This file focuses on strategy — how to prioritize your prep time for each company.

---

## How to Use This File

1. Identify your top 2–3 target companies
2. Read their detailed guide in `company-specific/`
3. Use the prep priority table below to allocate your time

---

## Google

**Roles**: ML Engineer (SWE track), Research Scientist, Applied Scientist, Data Scientist  
**Levels**: L5 (senior target), L6 (staff)  
**Detailed guide**: [company-specific/google/interview-guide.md](../company-specific/google/interview-guide.md)

### Prep priority (allocate extra time here vs other companies)
| Area | Priority | Notes |
|------|---------|-------|
| ML theory / math depth | ★★★★★ | Derivations expected; unique to Google |
| ML system design at scale | ★★★★★ | Must discuss billions-scale explicitly |
| Coding (LC medium–hard) | ★★★★☆ | High bar; clean code matters |
| Googleyness behavioral | ★★★☆☆ | Ambiguity, learning, collaboration |
| Statistics / experimentation | ★★★☆☆ | Less emphasis than Meta/Netflix |
| Product sense | ★★☆☆☆ | Less than Meta; still expected |

### Unique prep items
- Practice deriving: logistic regression gradient, attention mechanism, EM algorithm
- Read "Rules of Machine Learning" (Zinkevich, Google — free)
- Study Google's published papers: Wide & Deep, DCN v2, BERT, Word2Vec
- LeetCode: use Google tag, sort by frequency, 30+ problems minimum

### Interview mindset
Google values intellectual depth above all. Show curiosity. Go beyond the surface-level answer. When asked "what are the limitations of X?" — give specific, thoughtful limitations, not generic ones.

---

## Meta

**Roles**: ML Engineer, Data Scientist, Research Scientist, AI Software Engineer  
**Levels**: E5 (senior target), E6 (staff)  
**Detailed guide**: [company-specific/meta/interview-guide.md](../company-specific/meta/interview-guide.md)

### Prep priority
| Area | Priority | Notes |
|------|---------|-------|
| Product sense / ML metrics | ★★★★★ | Unique to Meta; tested for all roles |
| Metric investigation framework | ★★★★★ | "Metric dropped X%, investigate" |
| ML system design | ★★★★☆ | Social graph angle expected |
| Behavioral (Meta values) | ★★★★☆ | Move fast, be direct, build social value |
| Coding (LC medium) | ★★★☆☆ | Medium bar; clean code |
| ML theory depth | ★★★☆☆ | Less emphasis than Google |

### Unique prep items
- Practice 5+ "metric dropped" investigation walkthroughs
- Know: DAU/MAU, north star metrics, guardrail metrics, leading vs lagging indicators
- Study social graph ML: GNNs, network effects in experiments, social signal features
- Behavioral: map 10 stories to Meta values (Move Fast, Be Direct, Build Social Value)

### Interview mindset
Meta wants people who can connect ML decisions to user and business impact. In system design: always say "this improves NDCG, which means users find relevant content faster, which increases session length and return visits."

---

## Amazon

**Roles**: ML Engineer, Applied Scientist, Data Scientist  
**Levels**: L5 (senior target), L6 (staff)  
**Detailed guide**: [company-specific/amazon/leadership-principles.md](../company-specific/amazon/leadership-principles.md)

### Prep priority
| Area | Priority | Notes |
|------|---------|-------|
| Leadership Principles (behavioral) | ★★★★★ | Every round. 25 STAR stories minimum. |
| ML system design | ★★★★☆ | Amazon products: recommendations, forecasting, fraud |
| Coding (LC easy–medium) | ★★★★☆ | Lower bar than Google/Meta |
| ML theory | ★★★☆☆ | Solid fundamentals expected |
| Product sense | ★★★☆☆ | Customer Obsession connects ML to customers |
| Statistics | ★★★☆☆ | A/B testing, experimentation |

### Unique prep items
- Write 25 STAR stories, tag each to Leadership Principles (use template in behavioral Q&A)
- Hard requirement: 2+ stories per LP for the top 8 LPs (Ownership, Customer Obsession, Deliver Results, Dive Deep, Backbone, Bias for Action, Invent & Simplify, Earn Trust)
- Amazon system design focuses on their products: recommendation for Amazon.com, demand forecasting, fraud detection, Alexa ML
- Bar Raiser round: prepare for harder LP questions and deeper probing of all previous answers

### Interview mindset
STAR format is non-negotiable. Use "I" not "we." Every story must end with quantified results. Prepare for follow-up: "What would you do differently?" or "What was the most challenging part?" for every story.

---

## Apple

**Roles**: ML Engineer, AI/ML Engineer, Data Scientist, Research Scientist  
**Levels**: ICT4 (senior target), ICT5 (staff)  
**Detailed guide**: [company-specific/apple/interview-guide.md](../company-specific/apple/interview-guide.md)

### Prep priority
| Area | Priority | Notes |
|------|---------|-------|
| Domain expertise (team-specific) | ★★★★★ | Must tailor to specific team |
| On-device ML / model optimization | ★★★★★ | Apple-unique; quantization, CoreML |
| Privacy-preserving ML | ★★★★☆ | Differential privacy, federated learning |
| ML system design | ★★★★☆ | With on-device and privacy constraints |
| Coding (LC medium) | ★★★☆☆ | Standard bar |
| Behavioral (Apple values) | ★★★☆☆ | Craft, collaboration, ownership |

### Unique prep items
- Research your specific team deeply before the interview — ask your recruiter for details
- Know CoreML conversion: PyTorch → ONNX → CoreML
- Study differential privacy: local DP vs central DP, epsilon-delta notation, Laplace/Gaussian mechanism
- Federated learning: FedAvg algorithm, communication costs, non-i.i.d. challenges
- Model compression: quantization, pruning, distillation — know the trade-offs at each compression level

### Interview mindset
Privacy is Apple's brand identity — bring it up proactively in every system design. "Before I finalize the architecture, I want to consider: can we minimize what data leaves the device?" This signals you think like an Apple engineer.

---

## Netflix

**Roles**: Senior Data Scientist, ML Engineer, Research Engineer  
**Levels**: Senior, Senior II, Staff (pay-transparent)  
**Detailed guide**: [company-specific/netflix/interview-guide.md](../company-specific/netflix/interview-guide.md)

### Prep priority
| Area | Priority | Notes |
|------|---------|-------|
| Culture fit ("keeper test") | ★★★★★ | Can veto offer regardless of technical performance |
| Statistics / experimentation depth | ★★★★★ | Netflix invented CUPED; they test deeply |
| ML system design (personalization) | ★★★★☆ | Recommendation, thumbnails, search |
| Behavioral (Netflix values) | ★★★★☆ | Freedom & responsibility, candid feedback |
| ML theory | ★★★☆☆ | Strong fundamentals expected |
| Coding | ★★★☆☆ | Standard LC medium |

### Unique prep items
- Read Netflix Tech Blog (netflixtechblog.com) — 5+ articles before interviewing
- Know CUPED in depth: derive the variance reduction formula, not just the concept
- Understand contextual bandits: Thompson sampling, UCB, explore-exploit trade-offs
- Prepare culture stories: "tell me about a time you acted with freedom and responsibility"
- Know Netflix's recommendation system: row-level personalization, multiple algorithms per page, thumbnail A/B testing

### Interview mindset
Netflix culture interviews are substantive, not soft. They will probe whether you actually embody the values with specific examples. "I value direct feedback" is not enough — "here is the time I gave my manager critical feedback about a decision I thought was wrong, with this specific outcome" is what they want.

---

## Cross-Company Comparison

### Coding difficulty
`Google ≥ Meta > Netflix ≈ Apple > Amazon`

### ML theory depth
`Google >> Apple Research ≈ Netflix DS > Meta > Amazon`

### System design emphasis
`Google ≈ Meta ≈ Amazon > Netflix > Apple (team-dependent)`

### Behavioral weight
`Amazon >> Netflix > Meta > Apple ≈ Google`

### Product sense weight
`Meta >> Netflix > Google > Amazon > Apple`

### Privacy / safety emphasis
`Apple >> Google > Meta > Amazon > Netflix`

---

## Scheduling Strategy

If interviewing at multiple companies simultaneously:

**Ideal order**: Start with your 2nd/3rd priority companies (for practice), then your top choice company last when you're most prepared and have real offers in hand for negotiation leverage.

**Timing**: companies move at different speeds:
- Amazon: fastest (2–3 weeks from screen to offer)
- Google: slowest (6–10 weeks; committee review adds time)
- Meta: medium (3–5 weeks)
- Apple: medium-slow (4–8 weeks; team-specific variability)
- Netflix: medium (3–5 weeks)

**Exploding offer protection**: if you get an offer before you're done interviewing elsewhere, most companies will extend 1–2 weeks if you explain you have ongoing processes. Be transparent but professional.
