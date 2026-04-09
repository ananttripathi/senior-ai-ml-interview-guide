# Meta — ML/DS/AI Interview Guide

Roles: ML Engineer, Data Scientist, Research Scientist, AI Software Engineer  
Levels: E4 (mid), **E5 (senior — most common target)**, E6 (staff), E7 (principal)

---

## Interview Structure

| Round | Type | Duration |
|-------|------|----------|
| Recruiter screen | Background + fit | 30 min |
| Technical phone screen | ML fundamentals or coding | 45 min |
| Onsite 1 | ML system design | 45 min |
| Onsite 2 | Coding (LC medium) | 45 min |
| Onsite 3 | Product sense / ML metrics | 45 min |
| Onsite 4 | Behavioral (Meta values) | 45 min |
| Onsite 5 | Second technical (ML depth or second design) | 45 min |

---

## What Meta Uniquely Tests

### 1. Product Sense
Meta's biggest differentiator from other MAANG companies. Every ML role at Meta (including MLE) tests product sense — the ability to connect ML decisions to user and business outcomes.

**Product sense questions (Meta-specific)**:
- "How would you measure the success of Instagram Reels?"
- "Facebook is rolling out a new feature. What metrics would you use to evaluate its impact?"
- "Our engagement metric went up 5% but revenue went down 3% after an algorithm change. What happened and what do you do?"
- "How would you design the metric to measure the health of the Facebook News Feed?"

**Framework for product sense**:
1. **Clarify the goal**: what is the product trying to achieve? (connection, entertainment, commerce?)
2. **Define north star metric**: one primary metric that captures the core value (e.g., DAU/MAU ratio for engagement)
3. **Supporting metrics**: what sub-metrics drive the north star?
4. **Guardrail metrics**: what must not degrade? (revenue, integrity, creator satisfaction)
5. **Leading vs lagging**: short-term signals vs long-term impact

### 2. Metric investigation
Meta frequently asks: "This metric dropped/spiked X%. Walk me through your investigation."

**Framework**:
1. **Validate the signal**: Is the drop real or a measurement artifact? Check logging, instrumentation, data pipeline
2. **Segment the drop**: which surface? (mobile/desktop, geography, user segment, feature) — find where the drop is concentrated
3. **Check for external events**: platform changes, competitor actions, news events, holidays
4. **Check for internal changes**: recent code deploys, experiments running, policy changes
5. **Hypothesis test**: form a root cause hypothesis; check if it's consistent with all segments
6. **Remediation**: if causation identified, what's the fix? How quickly can you reverse it?

### 3. ML with social graph
Meta's unique asset is the social graph. Know how it's used:
- **Friend graph features**: engagement from friends on content as signal
- **Graph Neural Networks**: capturing social signal for ranking and recommendations
- **Network effects in experiments**: treatment of one user can affect their friends (SUTVA violation)
- **Graph-based fraud detection**: fraud rings, coordinated inauthentic behavior

### 4. Behavioral — Meta Values
Meta evaluates against their core values: Move Fast, Be Direct, Be Open, Build Social Value, Focus on Long-Term Impact.

**Move Fast**: "Tell me about a time you shipped something before it was perfect."
**Be Direct**: "Tell me about a time you gave critical feedback to a colleague or manager."
**Build Social Value**: "Tell me about a project where the ML outcome benefited users, not just the business metric."

---

## Meta ML Infrastructure

Know Meta's ML stack for senior MLE interviews:

- **PyTorch**: Meta's primary training framework (they built it)
- **FBLearner Flow**: internal ML pipeline orchestration
- **Manifold**: internal model serving and experimentation platform
- **Tupperware**: container orchestration for ML workloads
- **Scuba / Hive**: internal data analytics platforms

At senior level, you may be asked about your experience with large-scale ML pipelines. Relate your experience to these Meta concepts even if you used different tools.

---

## Integrity / Safety ML at Meta
Meta has large teams working on content integrity (hate speech, misinformation, coordinated manipulation). These roles heavily test:
- Content moderation system design
- Active learning and human-in-the-loop systems
- Handling multilingual, multimodal content at scale
- Adversarial dynamics (similar to fraud detection)

---

## Meta-Specific System Design Questions (Reported)

- "Design the Facebook News Feed ranking system"
- "Design a system to detect fake accounts on Instagram"
- "Design the Instagram hashtag recommendation system"
- "Design a system to rank comments on a Facebook post"
- "Design a system to predict which ads a user will engage with"
- "How would you build a system to detect coordinated inauthentic behavior at scale?"

---

## Meta Coding Style

- Expects LC medium difficulty; occasionally medium-hard for E6
- They care about clean, readable code — not just correctness
- Often have a "real-world" flavor: "Given a feed of events, compute the top K most active users in the last hour" (streaming + data structures)
- SQL questions common for DS track

---

## Tips for Meta Interviews

1. **Say "I" not "we"** in behavioral stories — Meta interviewers are trained to probe individual contribution.
2. **Product sense**: always anchor ML decisions to a user story. "This ranking change improves NDCG, which means users find relevant content faster, which increases session length and return rate."
3. **Move Fast culture**: Meta wants to hear about tradeoffs you made to ship faster. "Perfect is the enemy of good" is valued — show you can make pragmatic engineering decisions.
4. **The E5/E6 bar**: E5 is expected to own projects end-to-end with minimal guidance. E6 is expected to define the technical direction for an area, not just execute. Calibrate your stories accordingly.
5. **Integrity matters**: Meta takes platform integrity extremely seriously post-2016-2020. Acknowledge fairness, safety, and integrity considerations in your system designs.
