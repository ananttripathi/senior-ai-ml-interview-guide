# LLM Fundamentals — Architecture, Training & Inference

The most tested LLM topic at MAANG as of 2024–2026. Missing from pre-2023 resources.

---

## Modern Decoder-Only Architecture (GPT-style)

### Block structure
```
Input tokens
    ↓
Token Embedding + Positional Encoding (RoPE)
    ↓
[× N decoder blocks]
  ├── RMSNorm
  ├── Multi-Head Grouped-Query Attention (GQA) + KV Cache
  ├── Residual connection
  ├── RMSNorm
  ├── SwiGLU FFN (×4 expansion)
  └── Residual connection
    ↓
RMSNorm → Linear → Softmax → next token probabilities
```

---

## Key Architecture Components

### RMSNorm vs LayerNorm
**LayerNorm**: normalizes to mean=0, var=1 using both mean and variance. Requires two passes.

**RMSNorm** (used in LLaMA, Mistral, Gemma):
```
RMSNorm(x) = x / RMS(x) · γ,   RMS(x) = sqrt(mean(x²))
```
- No mean subtraction — only scales by root mean square
- Cheaper to compute (~10% faster), empirically equivalent quality
- No learned bias term

**Interview question**: "Why do modern LLMs use RMSNorm over LayerNorm?"
Answer: computational efficiency with no quality loss; bias removal simplifies the learned transformation.

---

### Positional Encoding: RoPE vs Alternatives

**Absolute (sinusoidal, original Transformer)**: fixed sine/cosine by position. Cannot generalize beyond training context length.

**RoPE (Rotary Position Embedding)** — used by LLaMA, GPT-NeoX, PaLM:
- Encodes position as a rotation in 2D subspaces of the query/key vectors
- Relative positions naturally emerge from dot product after rotation
- Extrapolates beyond training context length (with theta scaling)
- Formula: `Q_rot = Q · R(θ·m)` where m is position, θ is frequency

**ALiBi (Attention with Linear Biases)**: adds a linear penalty to attention scores based on distance. Simpler, good extrapolation.

**Interview angle**: "How do you extend a model's context window?"
- YaRN: scales RoPE frequencies; achieves 128k+ context with minimal fine-tuning
- Sliding window attention (Mistral): each token attends to W nearest tokens only
- LongFormer: sparse attention patterns (local + global tokens)

---

### Grouped-Query Attention (GQA)

**Multi-Head Attention (MHA)**: H query heads, H key heads, H value heads. KV cache grows as H × L × d_head.

**Multi-Query Attention (MQA)**: 1 shared KV head. Maximum memory savings, slight quality drop.

**Grouped-Query Attention (GQA)** — used by LLaMA 2/3, Mistral, Gemma:
- G groups, each sharing 1 KV head
- H/G ratio is tunable: H queries per K/V pair
- Better quality-memory trade-off than MQA

Why it matters for inference: KV cache is the primary memory bottleneck at large batch sizes. GQA enables larger batches without OOM.

---

### SwiGLU Feed-Forward Network

Standard FFN: `FFN(x) = max(0, xW₁)W₂`

SwiGLU (used in PaLM, LLaMA):
```
SwiGLU(x) = (xW) ⊙ swish(xV)) · W₂
swish(x) = x · sigmoid(βx)
```
- Gate mechanism allows non-monotonic activation
- Empirically better than GELU/ReLU at same parameter count
- Uses ⅔ × 4d hidden size to keep parameter count constant

---

### KV Cache

During autoregressive generation, each new token attends to all previous tokens. Without caching, computing K and V for past tokens is O(n²). With KV cache:
- Store K, V tensors for every previously generated token
- Each new step: compute Q for new token only, retrieve cached K, V → O(n) per step

**Memory cost**: `2 × layers × heads × head_dim × seq_len × batch_size × dtype_bytes`

For LLaMA 3 70B (FP16): ~1.6 GB for batch=1, seq=8192. Scales linearly with sequence length.

**Paged Attention (vLLM)**: allocates KV cache in fixed-size pages (like virtual memory) rather than contiguous blocks. Enables:
- Efficient memory sharing between sequences (prefix caching)
- Dynamic allocation — no pre-allocating worst-case sequence length
- Enables continuous batching

---

## Inference Optimization

### Speculative Decoding
**Problem**: large models are memory-bandwidth-bound — per-step latency dominated by loading model weights, not compute.

**Solution**: use a small draft model (7B) to generate K tokens speculatively, then verify all K with the large model (70B) in a single forward pass.
- If draft tokens match: K tokens generated for the cost of ~1 large model pass
- If mismatch at position i: accept tokens 0..i-1, discard i onward
- Speedup: 2–4× for sampling-heavy tasks; less for greedy decoding

**Key constraint**: draft model and target model must share vocabulary; draft model distribution should be similar to target.

### Continuous Batching
Traditional batching: wait until all sequences finish before starting new ones → GPU idle when some sequences finish early.

Continuous batching: swap in new requests at the iteration level as sequences complete. Dramatically improves GPU utilization.

### Quantization for Inference

| Precision | Memory (70B model) | Quality impact |
|-----------|-------------------|----------------|
| FP32 | ~280 GB | Baseline |
| FP16 / BF16 | ~140 GB | Negligible |
| INT8 (LLM.int8) | ~70 GB | Minor (<1%) |
| INT4 (GPTQ/AWQ) | ~35 GB | Small (1–3%) |
| INT2 | ~17 GB | Significant |

**GPTQ**: post-training quantization; minimizes quantization error per layer using Hessian information. Good quality.

**AWQ (Activation-aware Weight Quantization)**: preserves salient weights (top 1% by activation magnitude) at FP16; quantizes the rest to INT4. Better than GPTQ on reasoning tasks.

**GGUF** (formerly GGML): format used by llama.cpp for CPU/edge inference. Mixed precision (different layers quantized differently).

**Interview**: "A team wants to serve LLaMA 3 70B on 4× A100 (80GB each). What's your plan?"
Answer: INT4 quantization → ~35GB + KV cache → fits on 1 A100 or use tensor parallelism across 2 GPUs. With FP16 + tensor parallel across all 4: max throughput for production. Trade-off depends on latency vs cost targets.

---

## Training: Pre-training → Alignment

### Full Pipeline

```
1. Pre-training       → Predict next token on trillions of web tokens (GPT-style)
2. SFT                → Fine-tune on high-quality instruction-following pairs
3. RLHF / DPO         → Align with human preferences
4. (Optional) RLAIF   → AI-generated feedback instead of human annotators
```

### SFT (Supervised Fine-Tuning)
- Train on (instruction, response) pairs formatted with a chat template
- Uses causal LM loss on response tokens only (mask the instruction)
- Data quality >> data quantity; 1K high-quality examples often beats 100K noisy ones

### RLHF (Reinforcement Learning from Human Feedback)
1. Collect human preference data: given two model responses, which is better?
2. Train a **reward model** on this data (Bradley-Terry pairwise model)
3. Fine-tune the LLM with PPO to maximize reward model score
4. KL penalty prevents policy from drifting too far from the SFT model

**Problems with RLHF**: reward hacking, reward model overoptimization, computationally expensive (requires 3 models: policy + ref + reward).

### DPO (Direct Preference Optimization)
- Eliminates the reward model; directly optimizes on preference pairs
- Reparameterizes the RLHF objective: the optimal policy IS the reward model
- Loss: `L_DPO = -E[log σ(β log π(y_w|x)/π_ref(y_w|x) - β log π(y_l|x)/π_ref(y_l|x))]`
- Simpler, more stable, comparable or better quality than PPO

### KTO (Kahneman-Tversky Optimization)
- Works with unpaired data (just "good" or "bad" labels, not pairs)
- Based on prospect theory: losses feel 2.25× more painful than equivalent gains
- Useful when you have unbalanced or unpaired preference data

---

## Parameter-Efficient Fine-Tuning (PEFT)

### LoRA (Low-Rank Adaptation)
For a weight matrix W ∈ ℝ^(d×k), add a low-rank bypass:

```
W' = W + BA,   where B ∈ ℝ^(d×r), A ∈ ℝ^(r×k), r << min(d,k)
```

- Only A and B are trainable (r×(d+k) params vs d×k)
- Typical r: 4–64; r=8 works well for most tasks
- Memory savings: ~100× fewer trainable parameters
- Merge A and B into W at inference → zero latency overhead

### QLoRA
= LoRA + quantized base model

- Load 70B model in 4-bit NF4 (NormalFloat4) quantization
- Train LoRA adapters in BF16 on top of the quantized base
- Allows fine-tuning 70B on a single A100 80GB
- Uses double quantization (quantize the quantization constants) for additional savings

### When to use what
| Technique | Use when |
|-----------|---------|
| Full fine-tuning | Small model (<7B), large dataset, maximum quality needed |
| LoRA (r=16-64) | Medium models, limited data, task-specific adaptation |
| QLoRA | Large models (>13B), single GPU constraint |
| Prefix Tuning | Frozen model, many tasks via soft prompts |
| RAG (no fine-tuning) | Knowledge injection; frequently updated facts |

---

## Mixture of Experts (MoE)

MoE replaces dense FFN layers with N expert FFN networks + a router:

```
output = Σ gate_i(x) · Expert_i(x)
```
- Router: linear layer → softmax → select top-K experts (usually K=2)
- Only K experts compute for each token → sparse activation
- 8 experts × 7B params each = 56B total params, but only ~14B active per token (Mixtral 8×7B)

**Benefits**: scales parameter count without proportional compute increase
**Challenges**: load balancing (auxiliary loss), routing instability, memory even for inactive experts

---

## Common Interview Questions

**"Explain how attention works and why it's better than LSTMs for long sequences."**
Attention: each position attends to all other positions in O(1) steps with full context. LSTMs: gradient must propagate through all intermediate time steps for long-range dependencies (vanishing gradient). Attention's weakness: O(n²) memory and compute in sequence length (addressed by FlashAttention, sparse attention).

**"What is the difference between encoder-only, decoder-only, and encoder-decoder models? When would you use each?"**
- **Encoder-only (BERT)**: bidirectional attention; best for classification, NER, embeddings where full context available
- **Decoder-only (GPT)**: causal/unidirectional; best for generation tasks; dominates modern LLMs
- **Encoder-decoder (T5, BART)**: encoder reads input, decoder generates output; best for seq2seq tasks (translation, summarization) where input and output are distinct

**"How does FlashAttention work?"**
Standard attention: computes full N×N attention matrix (O(N²) memory). FlashAttention tiles the computation — processes blocks of Q, K, V that fit in SRAM, never materializing the full attention matrix in HBM. Result: same output, 2–4× faster, O(N) memory. FlashAttention-3 uses async pipelining for H100 tensor cores.
