---
layout: post
title: "The LLMs & Gen AI Curriculum I Teach — From Neurons to Agents in Production"
date: 2026-04-21
---

![The LLM & Gen AI curriculum: 11 modules arranged as a learning path from neural network foundations through transformers, training, RAG, agents, and production evals](/assets/images/llm-curriculum-hero.svg)

People keep asking me what a *serious* LLM curriculum actually looks like. Not a "prompt engineering in 3 days" bootcamp, not a YouTube playlist, but the real path from *"what is a neuron"* to *"I can build, fine-tune, and evaluate a production agent."*

This is the syllabus I use when I teach engineers Generative AI end-to-end. 
If you're self-studying, treat this as a checklist. If a module name looks like jargon, that's the point — by the end of that module, it shouldn't.

---

## Module 1 — Foundations: How Neural Networks Actually Learn

You cannot understand LLMs without first understanding why a neural network learns at all. This module is the cost of entry.

- **Parameters, weights, and biases** — what's actually being "learned"
- **Forward pass, loss functions, and backpropagation** — the feedback loop
- **Activation functions** — ReLU, GELU, softmax, and why non-linearity matters
- **Learning paradigms** — supervised, self-supervised, contrastive, and reinforcement learning


**Project:** Build and train a neural network from scratch on MNIST. No frameworks, just NumPy (or PyTorch if you must). If you can't explain every line, you're not done.

---

## Module 2 — How LLMs See Language: Tokens, Vectors, Attention

LLMs don't read words. They read numbers. This module closes the gap between human language and model input.

- **Tokenization** — BPE, WordPiece, SentencePiece, and why "strawberry" has 3 r's but GPT can't count them
- **Vectorization & embeddings** — turning tokens into meaning-rich vectors
- **Positional encodings** — sinusoidal, learned, RoPE, ALiBi
- **The attention mechanism** — the single idea that made transformers win

---

## Module 3 — Inside the Transformer

Once attention clicks, the rest of the transformer is surprisingly mechanical. Open the box.

- **Q, K, V matrices** — what each one actually represents
- **Self-attention vs cross-attention**
- **Multi-head attention** — why multiple heads beat one big head
- **Feed-Forward Networks (FFNs)** — the "thinking" layer between attention blocks
- **Logits, softmax, and sampling** — how a model commits to the next token

---

## Module 4 — Build a Transformer From Scratch

Theory is cheap. This is the module where students go from "I've read the paper" to "I've *been* the paper."

- **Causal / masked attention** — the trick that makes decoders autoregressive
- **Autoregressive generation** — sampling one token at a time
- **Training loop mechanics** — optimizer, scheduler, gradient clipping
- **Debugging training runs** — loss curves, NaNs, exploding gradients

**Project:** Code a small transformer end-to-end, following *Attention Is All You Need*. Train it on a toy dataset. Sample from it. Feel the magic.

---

## Module 5 — The Full LLM Lifecycle

A production LLM is ~5% modeling and ~95% everything else. This module zooms out.

- **Data pipeline** — sourcing, deduplication, filtering, deduping again
- **Pre-training at scale** — distributed training, sharding, mixed precision
- **Post-training** — SFT, RLHF, DPO, constitutional AI
- **Evaluation** — benchmarks, leakage, and why benchmark scores lie
- **Inference infrastructure** — serving, batching, KV cache

**Keywords:** `pre-training` `post-training` `RLHF` `evaluation`

---

## Module 6 — Making LLMs Fast and Yours: Quantization & Fine-Tuning

Running a 70B model on a laptop, or teaching GPT to sound like your company — this module covers both.

- **KV cache** — the optimization that makes long context feasible
- **Quantization** — FP16, INT8, INT4, GPTQ, AWQ
- **Attention optimizations** — FlashAttention, PagedAttention
- **Fine-tuning approaches** — full fine-tune vs LoRA vs QLoRA
- **The fine-tuning workflow** — dataset prep → training → evaluation → deployment

---

## Module 7 — Retrieval-Augmented Generation (RAG)

Most real-world "AI features" are RAG, not fine-tuning. This is where theory meets the messy reality of enterprise data.

- **Chunking strategies** — fixed, recursive, semantic, hierarchical
- **Ingestion pipelines** — parsing, cleaning, enriching, indexing
- **Vector embeddings and vector databases** — Pinecone, Weaviate, pgvector, Qdrant
- **Search algorithms** — ANN, HNSW, IVF, and when each wins
- **Reranking and hybrid search** — BM25 + dense, cross-encoders

---

## Module 8 — RAG in Practice: Build It, Then Make It Safe

Coding a RAG app is easy. Making it safe, accurate, and production-ready is the hard part.

- **Advanced retrieval** — query rewriting, HyDE, multi-query
- **Reranking strategies** — cross-encoders, LLM-based rerankers
- **Input guardrails** — prompt injection defense, intent classification
- **Output guardrails** — PII redaction, hallucination checks, policy filters
- **Safety patterns** — jailbreak resistance, content filtering

**Project:** Build a RAG chatbot over a real document set using API calls. Break it. Then secure it.

---

## Module 9 — From LLMs to Agents

An LLM predicts the next token. An *agent* takes actions in the world. The jump is bigger than it looks.

- **LLM vs single agent vs multi-agent systems**
- **Tool calling** — function schemas, structured outputs
- **The ReAct pattern** — reasoning + acting loops
- **Prompt chaining, orchestration, routing**
- **Agent design patterns** — planner-executor, critic, reflection

**Project:** Build a customer support agent that can look up orders, issue refunds, and escalate to a human — safely.

---

## Module 10 — Context Engineering, Memory, MCP & Multi-Agents

The current frontier. Agents that *remember*, *collaborate*, and plug into the wider tool ecosystem.

- **Context engineering** — the discipline that replaced "prompt engineering"
- **Memory architectures** — short-term, long-term, episodic, semantic
- **MCP (Model Context Protocol)** — standardized tool/data access for agents
- **A2A (Agent-to-Agent) protocols** — how agents talk to each other
- **Multi-agent orchestration** — roles, hierarchies, debate, voting

**Project:** Build an MCP-enabled agent with persistent memory and an optimized agentic flow.

**Keywords:** `context engineering` `memory` `MCP` `multi-agent`

---

## Module 11 — Evals and Shipping AI to Production

The final module, and the one most courses skip. In production, **evals are your test suite**. No evals, no ship.

- **Why evals matter** — the single biggest lever against hallucinations
- **Eval types** — deterministic, reference-based, LLM-as-a-judge, human eval
- **Building an eval harness** — golden sets, regression testing, CI integration
- **The design-decision matrix** — fine-tuning vs prompting vs RAG vs agents
- **Production tradeoffs** — latency, cost, accuracy, safety

**Project:** Build your own LLM-as-a-judge eval system for one of the apps from earlier modules.

---
