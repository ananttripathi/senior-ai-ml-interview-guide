# RAG Systems (Retrieval-Augmented Generation)

The most-tested applied AI topic in MAANG interviews 2024–2026. Expected for AI Engineer, ML Engineer, and senior DS roles that touch LLMs.

---

## What is RAG?

RAG grounds LLM responses in retrieved external knowledge instead of relying solely on parametric (trained-in) knowledge.

```
User query
    ↓
[Retrieval] → find relevant documents from a corpus
    ↓
[Augmentation] → inject retrieved docs into the prompt
    ↓
[Generation] → LLM generates answer grounded in documents
```

**When to use RAG vs fine-tuning:**

| Scenario | Use RAG | Use Fine-tuning |
|----------|---------|-----------------|
| Knowledge changes frequently | ✓ | ✗ (stale after training) |
| Need source citations | ✓ | ✗ |
| Domain-specific *style or behavior* | ✗ | ✓ |
| Small training budget | ✓ | ✗ |
| Need to inject 100k+ documents | ✓ | ✗ (context too large) |
| Task-specific format (JSON output, code) | ✗ | ✓ |

---

## Full RAG Pipeline

```
Offline (indexing):
  Raw documents → Chunking → Embedding → Vector index

Online (retrieval):
  Query → Query embedding → ANN search → top-K chunks
       → (optional) re-ranking → inject into prompt → LLM → answer
```

---

## Stage 1: Chunking

How you split documents into chunks dramatically affects retrieval quality.

### Fixed-size chunking
Split every N tokens with M-token overlap:
```
chunk_size = 512 tokens
overlap = 50 tokens
```
- Simple, fast
- Arbitrary boundaries can cut mid-sentence or mid-concept
- Good baseline; works well for homogeneous documents

### Sentence-level chunking
Use a sentence boundary detector (spaCy, NLTK) to split at natural boundaries.
- Better semantic coherence per chunk
- Chunk sizes vary — some sentences are trivially short

### Recursive / hierarchical chunking
Try large chunks first; if too large, split recursively with smaller separators: `\n\n` → `\n` → `.` → ` `
- LangChain's `RecursiveCharacterTextSplitter` implements this
- Good for diverse document types (markdown, code, prose mixed together)

### Semantic chunking
Embed each sentence; split when cosine similarity between adjacent sentences drops below a threshold.
- Most context-aware approach
- Slower to compute at indexing time
- Best for long documents with distinct topic sections

### Document-specific chunking
- **PDF**: extract by section headers; respect table/figure boundaries
- **Code**: split by function/class boundaries
- **HTML**: strip tags first; split by `<h2>`, `<p>` structure
- **Structured data** (CSV/JSON): convert rows to natural language before chunking

### Chunk size trade-offs

| Smaller chunks | Larger chunks |
|----------------|---------------|
| Higher precision retrieval | More context per retrieved chunk |
| More chunks to search | Fewer chunks to search |
| Risk: loses surrounding context | Risk: dilutes relevance with noise |
| Good for: precise Q&A | Good for: summarization, reasoning |

**Typical range**: 256–1024 tokens. Use 512 with 50-token overlap as a strong default.

---

## Stage 2: Embedding Models

The embedding model converts text to a dense vector. Quality matters enormously — a 20–40% accuracy gap between weak and strong embedding models has been reported.

### Selecting an embedding model

| Model | Dimensions | Notes |
|-------|-----------|-------|
| `text-embedding-3-small` (OpenAI) | 1536 | Fast, cheap, good general purpose |
| `text-embedding-3-large` (OpenAI) | 3072 | Best OpenAI quality |
| `BAAI/bge-large-en-v1.5` | 1024 | Strong open-source, good for fine-tuning |
| `intfloat/e5-large-v2` | 1024 | Strong open-source |
| `voyage-2` (Anthropic/Voyage) | 1024 | Strong for code + technical docs |
| Domain-specific fine-tuned | variable | Best for specialized corpora |

**Key principle**: embed queries and documents with the same model. Some models have separate `query:` and `passage:` prefixes — use them.

### When to fine-tune the embedding model
- General-purpose model performs poorly on your domain (medical, legal, code)
- You have labeled retrieval pairs (query, relevant_doc) to train on
- Use contrastive learning: pull relevant pairs together, push negatives apart

---

## Stage 3: Vector Index / ANN Search

After embedding all chunks, you need fast approximate nearest-neighbor (ANN) retrieval.

### Indexing algorithms

| Algorithm | Speed | Accuracy | Memory | Use case |
|-----------|-------|----------|--------|---------|
| Flat (exact) | Slowest | 100% | Low | Small corpus (<100K) |
| IVF (Inverted File) | Fast | ~95% | Medium | Large corpus |
| HNSW | Very fast | ~97% | High | Production default |
| PQ (Product Quantization) | Fast | ~90% | Very low | Memory-constrained |
| ScaNN | Very fast | ~97% | Medium | Google-scale |

**HNSW** (Hierarchical Navigable Small World) is the production standard. Used by FAISS, Weaviate, Qdrant, Pinecone by default.

### Vector DB selection

| DB | Best for | Notes |
|----|---------|-------|
| Pinecone | Managed, serverless | Zero infra ops; expensive at scale |
| Weaviate | Hybrid search built-in | GraphQL API; good multi-tenancy |
| Qdrant | Performance, filtering | Best filtering on metadata |
| Chroma | Local/dev | Simple API; not production-scale |
| pgvector | Existing Postgres stack | Avoid for >1M vectors |
| FAISS | Self-hosted, research | Library not a server; no persistence |

---

## Stage 4: Hybrid Search

Pure vector search misses keyword-exact matches. Pure BM25 misses semantic matches. Hybrid combines both.

```
Hybrid score = α × BM25_score + (1-α) × vector_score
```

Or use **Reciprocal Rank Fusion (RRF)**:
```
RRF_score = Σ 1 / (k + rank_i)   for each retriever i
```
RRF is robust, parameter-free, and outperforms fixed-weight fusion in practice.

**When hybrid beats pure vector:**
- Exact product names / IDs / codes (vector search struggles with out-of-vocabulary terms)
- Short, precise queries ("Python 3.11 release date")
- Technical documentation with specific terminology

**When pure vector beats hybrid:**
- Semantic paraphrase matching ("affordable housing" ↔ "low-cost homes")
- Cross-lingual retrieval
- Long, conceptual queries

---

## Stage 5: Re-ranking

The initial retrieval returns top-K by approximate similarity. A re-ranker re-scores these K candidates more accurately before sending to the LLM.

### Cross-encoder re-ranker
- Takes (query, document) as a *pair* and outputs a relevance score
- Much more accurate than bi-encoder (embedding model) because it can model interaction between query and document tokens
- Slower: runs inference per candidate, not once per query
- Typical pipeline: retrieve top-50 → re-rank → send top-5 to LLM

**Models:**
- `BAAI/bge-reranker-large` — strong open-source cross-encoder
- `cross-encoder/ms-marco-MiniLM-L-12-v2` — fast, good quality
- Cohere Rerank API — managed, high quality

### Maximal Marginal Relevance (MMR)
Selects documents that are relevant AND diverse:
```
score = λ · similarity(doc, query) - (1-λ) · max_similarity(doc, selected_docs)
```
Prevents returning 5 near-identical chunks from the same section of a document.

---

## Advanced RAG Architectures

### Self-RAG
The model generates special tokens to decide:
1. **Should I retrieve?** (maybe the answer is in parametric knowledge)
2. **Is the retrieved doc relevant?**
3. **Is my generated answer grounded in the retrieved doc?**

Improves faithfulness by making the model self-critique its retrieval and generation.

### Corrective RAG (CRAG)
After retrieval, a lightweight evaluator scores retrieved documents. If all are low-quality, it triggers a web search as fallback before generating.

### Graph RAG (Microsoft, 2024)
For corpora where entities and relationships matter (knowledge graphs):
1. Extract entities and relationships from all documents → build a knowledge graph
2. At query time, retrieve subgraphs relevant to the query
3. Summarize subgraph + relevant chunks → pass to LLM

**When to use**: legal documents, scientific papers, enterprise knowledge bases with complex cross-document relationships.

### HyDE (Hypothetical Document Embedding)
1. Ask the LLM to generate a *hypothetical* answer to the query (without retrieval)
2. Embed the hypothetical answer
3. Use that embedding for retrieval (instead of the query embedding)

**Why it works**: the hypothetical answer lives in the same embedding space as real documents; queries and documents can be stylistically very different.

---

## RAG Evaluation (RAGAS Framework)

Four dimensions to evaluate a RAG system:

| Metric | Measures | Formula |
|--------|---------|---------|
| **Faithfulness** | Is the answer supported by the retrieved context? | fraction of answer claims that appear in context |
| **Answer Relevance** | Does the answer address the question? | embedding similarity of answer ↔ question |
| **Context Precision** | Are the retrieved chunks relevant? | fraction of retrieved chunks that are relevant |
| **Context Recall** | Are all relevant chunks retrieved? | fraction of relevant chunks that were retrieved |

### LLM-as-judge evaluation
Use a stronger LLM (GPT-4o, Claude Opus) to score:
- Faithfulness: "Does this answer contradict the provided sources?"
- Completeness: "Does this answer fully address the question?"
- Groundedness: "Is every factual claim in the answer found in the sources?"

Scale with a 1–5 rubric; measure inter-rater agreement with human labels.

---

## Production RAG Failure Modes

These are the most important to know for system design interviews.

### 1. Retrieval miss
The relevant document exists but wasn't retrieved.
- **Diagnosis**: compute recall@K on a golden test set
- **Fix**: improve chunking, switch embedding model, add hybrid search, increase K

### 2. Hallucination with correct retrieval
Model ignores or misinterprets retrieved docs and makes up information.
- **Diagnosis**: faithfulness score drops while context recall is high
- **Fix**: strengthen system prompt ("Answer ONLY based on the provided context"), add citations requirement, use Self-RAG

### 3. Context overflow / lost in the middle
LLMs struggle to use information in the middle of a long context (documented in research).
- **Diagnosis**: performance degrades as K (retrieved chunks) increases
- **Fix**: reduce K, use re-ranking to surface the most relevant chunk first, use smaller chunk size

### 4. Stale index
New documents added to the corpus aren't reflected in the index.
- **Fix**: design an incremental indexing pipeline; re-embed and upsert new/modified documents on a schedule or via webhooks

### 5. Query-document distribution mismatch
Queries are short questions; documents are long prose. Embedding models trained on symmetric pairs may not handle this well.
- **Fix**: HyDE, query expansion, or fine-tune embedding model on asymmetric pairs

### 6. Poor chunk boundaries
Chunk splits across a table, code block, or multi-sentence argument; relevant info in two different chunks, neither retrieved.
- **Fix**: document-aware chunking; overlap between chunks; "parent-child" retrieval (small child chunk retrieved, larger parent chunk sent to LLM)

---

## Parent-Child Retrieval Pattern

A powerful production pattern:
- Index **small chunks** (256 tokens) for precise retrieval
- When a small chunk is retrieved, fetch its **parent chunk** (1024 tokens) for the LLM context
- Best of both worlds: precise retrieval + rich context for generation

```
Document
  └── Parent chunk (1024 tokens)  ← sent to LLM
        ├── Child chunk A (256 tokens)  ← indexed + retrieved
        ├── Child chunk B (256 tokens)
        └── Child chunk C (256 tokens)
```

---

## RAG System Design — Interview Cheat Sheet

"Design a document Q&A system for a 10,000-page enterprise knowledge base."

**Offline pipeline:**
1. Ingest → extract text (PDFs: pymupdf/pdfplumber; HTML: BeautifulSoup)
2. Chunk: recursive chunking, 512 tokens, 50-token overlap
3. Embed: `bge-large-en-v1.5` or `text-embedding-3-large`
4. Index: HNSW in Weaviate or Qdrant; store chunk + metadata (source, page, section)
5. Incremental re-indexing pipeline on document updates

**Online pipeline:**
1. Query → embed query
2. Hybrid search: BM25 + vector, RRF fusion, top-50 candidates
3. Re-rank: cross-encoder, top-5
4. Prompt assembly: system prompt + retrieved chunks with citations
5. LLM generation: Claude Sonnet / GPT-4o with temperature=0 for factual Q&A
6. Post-processing: extract citations, validate against context

**Monitoring:**
- Retrieval: precision@5 and recall@5 on golden test set (weekly)
- Generation: faithfulness score via LLM-as-judge (sampled daily)
- Latency: p95 end-to-end response time
- User signals: thumbs up/down, follow-up questions, session length
