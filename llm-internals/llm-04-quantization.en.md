# Quantization: Running Models on Less Memory

> 📅 2026-08-01 · Deep Dive
> From the bits and ranges of FP16 / INT8 / INT4, to why quantization works, what GGUF is, and the real quality-vs-size tradeoffs.

---

How much memory does a 70B model need, just for its weights? At 16 bits per weight, it's 70 × 10⁹ × 2 bytes ≈ 140 GB — beyond most consumer hardware. This post unpacks quantization: how it stores models in fewer bits, why it almost never falls apart, and how to choose.

## First, understand the numbers

Model weights are floating-point numbers; different precisions mean different bit widths and value ranges:

| Type | Bits | Approximate range | Typical use |
|------|------|-------------------|-------------|
| FP32 | 32 | enormous, ~1e-38 to ~3e38 | training accumulation, numeric baseline |
| FP16 | 16 | narrow, ~1e-4 to ~6e4 | mainstream speedup for inference & training |
| BF16 | 16 | same range as FP32, lower precision | mainstream choice for stable training |
| INT8 | 8 | -128 to 127 (integer) | near-lossless inference quantization |
| INT4 | 4 | -8 to 7 (integer) | extreme compression, some loss |

Key intuition: **floating point "spends" bits on an exponent that can be very small or very large; integers spend the whole range on tick marks.** Quantization squeezes each floating-point weight onto an integer scale, then uses a small set of parameters (scale and zero-point) to reconstruct something close to the original value.

### How to calculate memory

A model's memory footprint is dominated by its weights, and the formula is simple:

```
memory (bytes) = number of parameters × bytes_per_param
```

For a common 7B model:

| Precision | bytes/param | 7B weights | vs FP16 |
|-----------|-------------|------------|---------|
| FP16 / BF16 | 2 | ~14 GB | 1× (baseline) |
| INT8 | 1 | ~7 GB | 0.5× |
| INT4 | 0.5 | ~3.5 GB | 0.25× |

Add the KV cache and activations and real requirements are higher, but weights are always the biggest share. That's why dropping a 7B model from FP16 to INT4 can turn an 8 GB GPU from a no-go into a smooth experience.

## Why quantization works

It sounds like "throwing away precision," yet it works well in practice, for three reasons:

1. **Weights are redundant.** Neural networks have enormous parameter counts that compensate for each other. Dropping every weight from 16 bits to 8 bits has, on average, a tiny effect on output. The model is a team: a few members misremembering numbers leaves the overall result roughly correct.
2. **Models are already robust to noise.** Training itself is full of noise (random batches, learning rates). Neural networks are naturally tolerant of small errors.
3. **Per-channel scaling.** Different channels have wildly different weight ranges. Giving each channel its own scale dramatically reduces the error.

### How quality is measured

Quantization loss is usually measured with perplexity or downstream task scores. For 7B–8B-class models, INT8 is nearly imperceptible and INT4 is usually acceptable; but **smaller and older models are more sensitive to quantization**, and the loss can become visible.

## The quantization family

* **Post-Training Quantization (PTQ):** convert after training; most common. `bitsandbytes`'s `load_in_4bit` does it in one line.
* **Quantization-Aware Training (QAT):** folds quantization error into training, better quality, but requires retraining and is expensive.
* **GGUF k-quants:** the quantization formats of the llama.cpp ecosystem (Q4\_K\_M, Q5\_K\_S, Q8\_0, …). The `_K` means "quantize important tensors in finer groups" — k-quant is what made INT4 quality respectable in recent years.

```python
# bitsandbytes example: convert on load, straight to 4-bit
from transformers import AutoModelForCausalLM, BitsAndBytesConfig

config = BitsAndBytesConfig(load_in_4bit=True, bnb_4bit_quant_type="nf4")
model = AutoModelForCausalLM.from_pretrained(
    "meta-llama/Llama-3.1-8B",
    quantization_config=config,
)
```

## GGUF and llama.cpp

The most common setup for running quantized models locally is **GGUF file format + llama.cpp**. GGUF packages the model weights, tokenizer, and quantization parameters into a single file. It's especially good on CPU, and supports layer offloading to GPU.

With Ollama, one line:

```bash
ollama run llama3.1:8b-instruct-q4_K_M
```

With llama.cpp manually:

```bash
# first download the Q4_K_M GGUF file (e.g. from Hugging Face)
llama-cli -m models/llama-3.1-8b-instruct.Q4_K_M.gguf \
  -p "Explain quantization in three sentences." -n 256
```

The `Q4_K_M` in the filename is a quality code: `4` is the bit width, `K` is the k-quant algorithm, `M` is the mixed strategy (key layers get a few more bits).

## The real tradeoff: quality vs size

There is no free lunch, but this lunch is almost free. Common options for Llama 3.1 8B:

| Quantization | Weights | Quality | Best for |
|--------------|---------|---------|----------|
| Q8\_0 (INT8) | ~8.5 GB | near original | enough memory, want quality |
| Q5\_K\_M | ~5.5 GB | very good | a great all-rounder for local use |
| Q4\_K\_M | ~4.9 GB | good | the best default when memory is tight |
| Q3\_K\_M | ~4 GB | acceptable | last resort, big models on tight budgets |
| Q2\_K | ~3.1 GB | clearly degraded | almost never recommended |

A decision tree for picking:

* **Can you fit FP16/INT8?** Use the highest precision. Don't overthink.
* **Only INT4 fits?** Prefer the k-quant `Q4_K_M` over the old-style `q4_0`.
* **Bigger models make quantization "cheaper."** A 70B Q4 loses only a little quality yet saves 100+ GB — large models tolerate quantization better.
* **Mind the KV cache.** The longer the context, the larger the KV cache; at long contexts, count it in — don't only look at the weights.

## Summary

| Concept | In one sentence |
|---------|-----------------|
| Precision | FP32 / FP16 / BF16 / INT8 / INT4 — different bits and ranges |
| Memory | params × bytes\_per\_param; a 7B goes from ~14 GB at FP16 to ~3.5 GB at INT4 |
| Why it works | weight redundancy + robustness to noise + per-channel scaling |
| GGUF / llama.cpp | the file format and inference engine for packaged quantized models; the mainstream local setup |
| Tradeoff | fewer bits = smaller and faster but slightly worse quality; big models tolerate it better |

Quantization is what lets models that "only run in the cloud" fit into laptops and phones. Understand the bits, the memory formula, and the k-quant naming, and you hold the key to choosing: **find your own balance between quality and size.**
