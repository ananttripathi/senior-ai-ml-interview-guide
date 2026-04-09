# ML System Design: Search Ranking

Tested at Google, Amazon, LinkedIn, Airbnb, and Meta. The question may be: "Design Google Search ranking", "Design Amazon product search", or "Design a semantic search system."

---

## Problem Statement

Given a user query, retrieve and rank the most relevant documents/products from a large corpus, in real-time, at scale.

**Business goal**: user finds what they're looking for quickly → engagement, conversion, retention  
**ML task**: learning-to-rank (LTR) — order documents by predicted relevance to query

---

## Step 1: Clarify Requirements

- **Domain**: web search, e-commerce product search, internal enterprise search, video search?
- **Corpus size**: thousands vs billions of documents?
- **Query types**: keyword, natural language, voice, multimodal (image + text)?
- **Latency**: web search ≤ 200ms; enterprise search ≤ 500ms
- **Freshness**: how quickly do new documents need to appear in results?
- **Personalization**: same query, different results for different users?

---

## Step 2: Success Metrics

### Business metrics
- Click-through rate on search results
- Zero-results rate (bad: user got no results)
- Reformulation rate (bad: user had to rephrase query)
- Conversion rate (for e-commerce)
- Session satisfaction (did user come back? did they complete their task?)

### Offline ML metrics
- **NDCG@K** (Normalized Discounted Cumulative Gain): the standard for ranking quality
- **MAP** (Mean Average Precision): good for binary relevance (relevant/not relevant)
- **MRR** (Mean Reciprocal Rank): optimizes for finding the first relevant result
- **ERR** (Expected Reciprocal Rank): accounts for cascading nature of search (user stops after finding what they need)

---

## Step 3: Architecture

### Multi-Stage Retrieval Pipeline

```
User query: "noise canceling wireless headphones under $200"
        ↓
[Stage 1: Retrieval] — fast, approximate
  ├── BM25 / inverted index (keyword match)
  ├── Dense retrieval (bi-encoder ANN search)
  └── Union or RRF fusion → ~1000 candidates
        ↓
[Stage 2: Ranking] — expensive, precise
  ├── Pointwise/listwise neural ranker
  └── Scores all candidates → top 50
        ↓
[Stage 3: Re-ranking / Diversification]
  ├── Business rules (sponsored results, freshness boost)
  ├── Diversity (don't show 10 blue headphones)
  └── Final top 10
        ↓
[Serve results]
```

---

## Stage 1: Retrieval

### BM25 (Sparse / Keyword Retrieval)

The industry standard keyword retrieval algorithm. An improvement over TF-IDF:

```
BM25(q,d) = Σ IDF(t) × (tf(t,d) × (k₁+1)) / (tf(t,d) + k₁ × (1 - b + b × |d|/avgdl))

where:
  IDF(t) = log((N - df(t) + 0.5) / (df(t) + 0.5) + 1)
  tf(t,d) = term frequency of term t in document d
  |d| = document length, avgdl = average document length
  k₁ = 1.2–2.0 (saturation parameter)
  b = 0.75 (length normalization)
```

Strengths: exact keyword match, fast (inverted index), works well out-of-the-box  
Weaknesses: vocabulary mismatch ("sofa" vs "couch"), no semantic understanding

### Dense Retrieval (Bi-encoder)

```
Query tower: query → BERT-style encoder → query_embedding (768d)
Document tower: doc → BERT-style encoder → doc_embedding (768d)
Score = cosine_similarity(query_emb, doc_emb)
```

- Documents are pre-embedded offline → stored in ANN index (FAISS/HNSW)
- At query time: embed query → ANN search → top-K similar docs in <10ms
- Training: contrastive learning with hard negatives (BM25 negatives, in-batch negatives)

Strengths: semantic matching, handles paraphrases, cross-lingual  
Weaknesses: misses exact keyword matches, requires fine-tuning on domain data

### Hybrid Retrieval

Combine BM25 and dense retrieval with **Reciprocal Rank Fusion (RRF)**:

```python
def rrf(bm25_ranking, dense_ranking, k=60):
    scores = {}
    for rank, doc_id in enumerate(bm25_ranking):
        scores[doc_id] = scores.get(doc_id, 0) + 1/(k + rank + 1)
    for rank, doc_id in enumerate(dense_ranking):
        scores[doc_id] = scores.get(doc_id, 0) + 1/(k + rank + 1)
    return sorted(scores, key=scores.get, reverse=True)
```

RRF is robust and parameter-free. Typically outperforms both BM25 alone and dense alone.

### Query Understanding

Before retrieval, process the query:
- **Spelling correction**: "noyse canceling" → "noise canceling"
- **Query expansion**: "headphones" → add "earphones", "earbuds" (synonym expansion)
- **Intent classification**: navigational ("Amazon") vs informational ("best headphones") vs transactional ("buy headphones")
- **Named entity recognition**: extract product names, brands, model numbers
- **Query rewriting**: reformulate to a form that retrieves better documents (learned end-to-end)

---

## Stage 2: Neural Ranking (Learning-to-Rank)

### Cross-Encoder / Interaction Model

Unlike bi-encoders (separate embeddings), cross-encoders encode query and document *together*:

```
Input: [CLS] query [SEP] document [SEP] → BERT → CLS embedding → linear → score
```

- Much more accurate than bi-encoder (can model query-document interaction at token level)
- Too slow for full corpus retrieval (no pre-indexing) → use only on top-K candidates from Stage 1

### Learning-to-Rank Approaches

**Pointwise**: predict a relevance score per (query, doc) pair independently. Train with regression or classification loss. Treats ranking as a prediction problem.

**Pairwise**: train on pairs (doc_a, doc_b) — which is more relevant? RankNet, LambdaRank.
- LambdaRank: modifies gradients to directly optimize NDCG. Widely used in industry.

**Listwise**: optimize over the full ranked list. ListNet, ListMLE, ApproxNDCG. Most aligned with the evaluation metric but harder to train.

**Industry default**: LambdaRank or LambdaMART (gradient boosted trees with LambdaRank gradients). Used at Microsoft, Yahoo. LightGBM supports LambdaRank natively.

### Features for the Ranking Model

**Query features:**
- Query length, query type (question vs keyword)
- Query popularity (head vs torso vs tail query)
- Query embeddings

**Document features:**
- BM25 score for this query
- Document freshness (creation date, modification date)
- Document authority (PageRank or domain authority)
- Document length, density of query terms
- Historical CTR for this document on similar queries

**Query × Document features:**
- BM25 score
- Dense retrieval score
- Field-specific scores (title match vs body match — title match usually more important)
- Semantic similarity (cosine similarity of embeddings)

**User × Query features (personalization):**
- Has user visited this document before?
- User's historical CTR on this domain
- User's inferred topic interest overlap with query

---

## Stage 3: Diversity and Re-ranking

### Maximum Marginal Relevance (MMR)

Selects the next document that is most relevant to the query AND most different from already-selected documents:

```python
def mmr(candidates, selected, query_embedding, lambda_=0.5):
    # For each candidate, score = lambda * sim(doc, query) - (1-lambda) * max_sim(doc, selected)
    scores = {}
    for doc in candidates:
        relevance = cosine_sim(doc.embedding, query_embedding)
        redundancy = max([cosine_sim(doc.embedding, s.embedding) for s in selected], default=0)
        scores[doc] = lambda_ * relevance - (1 - lambda_) * redundancy
    return max(scores, key=scores.get)
```

### Business rules applied last
- Inject sponsored results in designated positions
- Apply freshness boost for news queries
- Remove duplicate domains (don't show 5 results from the same site)
- Apply safe search filters

---

## Training Data

### Relevance Labels

**Explicit labels**: editorial judgments from human raters (expensive; used by Google, Bing). Typically a 5-point scale: Perfect, Excellent, Good, Fair, Bad.

**Implicit labels from click logs**:
- Clicked: treat as positive (noisy — position bias)
- Skipped-over clicked: treat as negative
- Dwell time: clicked + stayed for > 30s = strong positive

**Position bias problem**: position 1 gets clicked more regardless of quality. Fix with:
- IPW (Inverse Propensity Weighting): weight clicks by 1/P(position shown)
- Pairwise debiasing: use "skipped-over clicked" as negatives (position-independent signal)

---

## Offline Evaluation

- **NDCG@5, NDCG@10**: primary metrics
- **Human evaluation**: for new retrieval systems, manual review of top-10 results on 500+ sample queries
- **Slice analysis**: evaluate separately on head queries (popular), torso, and tail. Tail queries are often under-optimized and matter for coverage.

---

## Online Evaluation

**A/B testing for search is hard**:
- **Query-level randomization**: same user can get A in one search and B in the next — carryover effects
- **User-level randomization** (preferred): consistently assign user to A or B across all their searches
- **Primary metric**: CTR@1, NDCG from clicks, task completion rate
- **Interleaving** (faster): for ranking, show results interleaved from A and B in one SERP; user click signal is 100× more sensitive than A/B

---

## Key Challenges and Trade-offs

### Freshness vs Quality
New documents haven't accumulated click signals or authority scores. Apply freshness boost for time-sensitive queries ("latest iPhone rumors") but not for evergreen queries ("how to tie a bowline knot").

### Head vs Tail Queries
- Head queries (~1,000 unique queries, 80% of traffic): well-optimized, lots of training data
- Tail queries (~billions of unique queries, 20% of traffic): little training data, critical for coverage
- Solution: zero-shot generalization via semantic embeddings; use document content rather than query-document co-occurrence signals

### Query-Document Vocabulary Mismatch
User queries are short (3–5 words); documents are long (hundreds of words). Dense retrieval helps but requires domain-specific fine-tuning. Consider query expansion and document field weighting (title > abstract > body).

### Personalization vs Privacy
Personalizing search with user history improves relevance but raises privacy concerns. Implement with:
- On-device personalization (user data never leaves device)
- Federated learning for aggregated personalization signals
- Differential privacy for user embeddings in the index
