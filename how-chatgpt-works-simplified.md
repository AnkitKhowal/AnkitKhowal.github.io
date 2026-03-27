# How ChatGPT Actually Works — A Simple Guide

You type a message, press Enter, and words start appearing. Between that keypress and the first word showing up, an enormous amount of machinery kicks in. This post breaks it all down in plain language.

---

## 1. The Training Data

ChatGPT learned from reading a massive chunk of the internet — roughly **13 trillion "tokens"** (think of a token as roughly one word).

Most of this data starts as raw web pages scraped from the internet. But raw web pages are messy — full of ads, menus, spam, and duplicate content. So before training, the data goes through a cleaning pipeline:

1. **Strip the junk** — remove HTML tags, navigation bars, footers
2. **Remove duplicates** — the same article often appears on hundreds of sites
3. **Filter by language** — keep only the languages you want
4. **Quality check** — remove spam, low-quality, and inappropriate content
5. **Remove personal info** — strip out things like phone numbers or addresses

After all this filtering, only about **10% of the original data survives**. Quality matters enormously — two identical models trained on different quality data will perform very differently.

---

## 2. Turning Text into Numbers (Tokenization)

The model can't read English. It reads **numbers**. So every piece of text gets converted into a sequence of integers before the model sees it.

The word "Unbelievable" might become one number, or it might be split into "un" + "believable" — two numbers. This splitting system is called **BPE (Byte Pair Encoding)**. It learns which character combinations appear most frequently and groups them together.

**Example:**

> "How does ChatGPT work?" → `[4438, 1587, 91985, 990, 30]` (5 tokens)

Fewer tokens = less work for the model = faster and cheaper responses.

---

## 3. The Brain: The Transformer

The transformer is the architecture that powers ChatGPT. Think of it as a pipeline:

```
Numbers go in → [Embedding] → [Transformer Blocks × 96] → [Output Layer] → Probabilities come out
```

### What each part does:

- **Embedding**: Converts each token number into a rich mathematical representation (a list of ~4,096 numbers that capture meaning)
- **Transformer Blocks**: The core processing. Each block has two jobs:
  - **Attention**: Lets tokens "look at" other tokens to understand context. This is how "bank" knows whether it means a river bank or a financial bank — by looking at surrounding words.
  - **Feed-Forward Network (FFN)**: Processes each token individually to refine its meaning
- **Output Layer**: Produces a probability for every possible next word (~50,000 options)

### Attention in plain English

Imagine you're reading "The cat sat on the mat." When processing "sat," the model needs to know *who* sat. Attention lets "sat" peek at all previous words and decide: "cat" is the most relevant one.

Each token asks three questions:
- **Query**: "What am I looking for?"
- **Key**: "What do I have to offer?"
- **Value**: "What information do I carry?"

Tokens with matching queries and keys pay more attention to each other.

### The Context Window

This is the model's "short-term memory" — everything it can see at once. GPT-4o can see up to **128,000 tokens** at a time. Once that fills up, the oldest messages get dropped. The model literally forgets them.

---

## 4. Pretraining: Learning to Predict the Next Word

Training is conceptually simple: show the model a sentence like "The capital of France is ___" and ask it to guess the next word. If it guesses wrong, adjust its internal numbers slightly. Repeat this **trillions of times**.

The model is never told grammar rules, facts, or how to code. All of that **emerges** from learning to predict what comes next. To predict well, you need to understand language, facts, reasoning, and structure.

**Key insight**: The model doesn't memorize the internet. It learns *patterns that predict* the internet. That's why it "knows" that Paris is in France — not because it stored that fact, but because "Paris" is statistically the right next word in many contexts.

**This is also why it sometimes makes things up (hallucinates)**: it always generates the most *probable* next word, not necessarily the most *true* one.

### Scale comparison

| Model | Training Cost | Time |
|-------|-------------|------|
| GPT-2 (2019) | ~$50,000 | ~1 week |
| GPT-4 (2023) | $100,000,000+ | Months |

---

## 5. Post-Training: Making It Useful

After pretraining, the model is just a very sophisticated autocomplete. Ask it a question and it continues your text rather than answering it. Post-training turns it into a helpful assistant.

### Step A: Supervised Fine-Tuning (SFT)

Human contractors write ~15,000 examples of ideal conversations:

> **User**: What's the capital of France?
> **Ideal response**: The capital of France is Paris.

The model trains on these examples so it learns to *answer* questions, not just continue text.

### Step B: RLHF (Learning from Human Preferences)

SFT teaches format, but "good" is subjective. RLHF teaches *quality*:

1. The model generates several different responses to the same question
2. Human raters **rank** them from best to worst
3. A "reward model" is trained to predict which responses humans prefer
4. The main model is updated to produce responses the reward model scores highly

Think of it like training a dog: you can't explain what "good behavior" means in words, but you can say "yes, that!" or "no, not that" until the dog figures it out.

### DPO: A Simpler Alternative

Newer models (Llama 3, Mistral) skip the reward model entirely. **DPO (Direct Preference Optimization)** directly trains the model to prefer good responses over bad ones using the same ranking data but a simpler process. Same result, less complexity.

---

## 6. Thinking Models (o1, DeepSeek-R1)

Normal models answer immediately. Thinking models **show their work** first:

> **Normal model**: "237 × 194 = 45,978"
>
> **Thinking model**: "Let me break this down... 237 × 200 = 47,400. 237 × 6 = 1,422. 47,400 − 1,422 = 45,978."

The thinking happens inside special `<think>` tokens. The model learned that working through steps improves accuracy — especially for math, logic, and code. For simple questions, thinking just wastes time and money.

---

## 7. How It Picks the Next Word (Decoding)

The model doesn't just pick "the best" word. It outputs probabilities for all ~50,000 possible next tokens and then uses a strategy to choose one.

| Strategy | How it works | Best for |
|----------|-------------|----------|
| **Greedy** | Always pick the most probable word | Factual Q&A |
| **Top-P (Nucleus)** | Pick from the smallest set of words covering 90% probability | Chat, creative writing |
| **Temperature** | Controls randomness. Low = predictable. High = creative. | Tuning creativity vs accuracy |

**Temperature** is the main knob:
- `temperature=0` → Always picks the top word (deterministic, boring)
- `temperature=0.7` → Balanced (default for chat)
- `temperature=1.0` → More creative and varied

---

## 8. What Happens When You Press Enter

Here's the full journey of a single message, step by step:

| Step | What happens | Time |
|------|-------------|------|
| 1. **You press Enter** | Browser sends your message to OpenAI's servers | — |
| 2. **Edge network** | Handles encryption, blocks attacks, routes to nearest data center | ~20ms |
| 3. **API Gateway** | Checks your API key, applies rate limits | ~5ms |
| 4. **Build context** | Assembles system prompt + conversation history + your message | ~10ms |
| 5. **Safety check (input)** | A fast classifier scans for harmful content | ~12ms |
| 6. **Prefill** | Model reads your entire prompt in one pass and builds an internal memory (KV cache) | ~50ms |
| 7. **Decode** | Model generates words one at a time, streaming each back to you | ~4,000ms |
| 8. **Safety check (output)** | Another classifier scans the response as it streams | ~8ms |
| 9. **Post-processing** | Convert numbers back to text, count tokens for billing | ~2ms |

**Total time to first word**: ~130ms
**Total time for a ~400-word response**: ~4 seconds

Words stream back as they're generated — that's why you see them appear one by one instead of waiting for the full response.

---

## 9. Making It Fast and Cheap (Optimizations)

Running these models is extremely expensive. Here are the main tricks used to make it practical:

### KV Cache
Without caching, the model would redo all its work for every new word. The KV cache remembers previous calculations, so each new word only needs one small computation instead of reprocessing everything.

### Continuous Batching
Instead of waiting for one user's request to finish before starting the next, the system handles many requests simultaneously. When one finishes, a new one immediately takes its slot. This keeps GPUs busy 80-90% of the time instead of 20-30%.

### Quantization (Compression)
Model weights are normally stored as precise decimal numbers (4 bytes each). By rounding them to less precise formats, you can shrink the model dramatically:

| Precision | 7B Model Size | Quality Impact |
|-----------|--------------|----------------|
| Full (FP32) | 28 GB | Baseline |
| Half (BF16) | 14 GB | Negligible |
| INT8 | 7 GB | Negligible |
| INT4 | 3.5 GB | Small (~10%) |

This is how models run on laptops and phones.

### Speculative Decoding
A small, fast model guesses the next 5 words. The big, smart model checks all 5 at once. If 4 out of 5 are right, you just generated 5 words in the time it normally takes to generate 1. Result: **2-3x faster** with no quality loss.

### Flash Attention
A clever rearrangement of the math so that it happens in the GPU's fast memory (SRAM) instead of slow memory (HBM). Same answer, **2-4x faster**.

### Prompt Caching
If every request starts with the same system prompt ("You are a helpful assistant..."), why reprocess it every time? Cache it once, reuse for all requests. OpenAI charges **50% less** for cached input tokens.

### Model Routing
Not every question needs the most powerful model. A tiny classifier routes "What's 2+2?" to a cheap model and "Debug this kernel module" to the expensive one. Companies save **40-60%** this way.

---

## 10. How Do You Know If a Model Is Good? (Evaluation)

| Method | Pros | Cons |
|--------|------|------|
| **Benchmarks** (MMLU, HumanEval) | Standardized, fast | Labs optimize for them; scores inflate |
| **Chatbot Arena** | Real users, real tasks, blind comparison | Slow to collect data |
| **Human evaluation** | Directly measures helpfulness | Expensive; raters prefer longer answers |

**Chatbot Arena** is generally the most trusted — real users compare two anonymous models on their own questions, and an ELO rating system ranks them.

---

## The Big Picture

```
Web pages → Clean & filter → Tokenize → Pretrain (predict next word)
  → Fine-tune (follow instructions) → RLHF (align to preferences)
  → Serve through production infrastructure → Stream words to your browser
```

The core idea is remarkably simple: **predict the next word**. Do that well enough, on enough data, with a big enough model, and you get something that can write code, explain physics, and pass the bar exam.

But it's still just predicting the most likely next word — not retrieving truth. That's both the magic and the limitation.

---

## Quick Glossary

| Term | Meaning |
|------|---------|
| **Token** | A word or piece of a word, converted to a number |
| **Transformer** | The neural network architecture that powers LLMs |
| **Attention** | The mechanism that lets words "look at" other words for context |
| **Pretraining** | Learning language by predicting the next word on internet text |
| **SFT** | Teaching the model to answer questions using example conversations |
| **RLHF** | Improving quality by learning from human preferences |
| **KV Cache** | Saved calculations that prevent redundant work during generation |
| **Quantization** | Compressing model weights to use less memory |
| **TTFT** | Time to First Token — how quickly the first word appears |
| **Hallucination** | When the model confidently generates something false |
