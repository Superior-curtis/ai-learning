# Apple Silicon vs NVIDIA: Running Models on Your Hardware

> 📅 2026-08-01 · Deep Dive
> Unified memory vs VRAM, Metal vs CUDA: from bandwidth to memory, understand what your machine can actually run.

---

The last post covered quantization and GGUF; this one gets back to hardware. The first question most people ask is: "Can my MacBook run it?" The answer comes down to two things: how much memory you have, and how fast that memory is. This post lays out the Apple Silicon and NVIDIA paths in one go.

***

## Two fundamentally different architectures

### Apple Silicon: unified memory

The Mac's M-series chips (M1/M2/M3/M4) put the CPU, GPU, and memory on the same die. The CPU and GPU **share the same memory pool** — no copying data back and forth.

**Advantages:**

* The memory pool is your "VRAM": a 64GB MacBook effectively has 64GB available for models
* Memory capacity decides what you can run, and large capacity is relatively cheap

### NVIDIA: dedicated VRAM

An NVIDIA card is a separate board on PCIe with its own dedicated video memory (VRAM). Data moves between the card and main memory over the bus.

**Advantages:**

* Very high memory bandwidth and low latency
* A mature ecosystem: CUDA, cuDNN, TensorRT

**Limitations:**

* VRAM is fixed: buy 24GB and you have 24GB, no upgrades later

***

## Metal vs CUDA: two software stacks

Beyond the hardware, what you can actually run is decided by software.

### CUDA (NVIDIA)

CUDA is NVIDIA's general-purpose parallel computing platform and the "standard" of the entire AI ecosystem. GPU support in PyTorch, TensorFlow, and vLLM is almost always CUDA-first.

### Metal / MLX / MPS (Apple)

Apple's story is more fragmented:

* **Metal**: the low-level GPU API; llama.cpp uses this path
* **MLX**: Apple's own array framework, deeply optimized for Apple Silicon
* **MPS**: PyTorch's backend on Mac, corresponding to `torch.device("mps")`

The good news: llama.cpp supports both Metal and CUDA. **The same tools and the same GGUF files run on both** — only the acceleration layer underneath differs.

***

## Memory bandwidth is the real key

LLM inference is a *memory-bound* workload: every token generated reads the entire model's weights once. So **speed depends mostly on memory bandwidth, not compute power**.

### Approximate bandwidth by platform

| Platform | Memory bandwidth (approx.) |
|----------|---------------------------|
| M1 / M2 base | ~68-100 GB/s |
| M1 Pro / M2 Pro | ~200 GB/s |
| M1 Max / M3 Max | ~400 GB/s |
| M4 Max | ~546 GB/s |
| M2 Ultra | ~800 GB/s |
| RTX 4090 | ~1000 GB/s |
| RTX 5090 | ~1790 GB/s |
| H100 | ~3350 GB/s |

The higher the bandwidth, the more tokens per second for the same model. An RTX 4090 has roughly 2.5x the bandwidth of an M3 Max.

***

## Models that run great on a Mac

If your goal is **personal use, development assistance, or experimentation**, Apple Silicon is genuinely a great experience:

* 7B-8B quantized models (Q4): most M-series chips hit 15-40 tok/s — plenty smooth
* 14B: fluid on high-end chips (Max / Ultra), slower but usable on Pro
* 32B: usable on Max / Ultra, moderate speed
* 70B: only loadable on 64GB+ Max / Ultra, speed drops to single-digit tok/s

**Key numbers:** a 7B model at Q4 is about 4-5GB — a 32GB Mac handles it easily; a 70B model at Q4 is about 40GB, requiring 64GB+.

### Mac strengths

* Portable: take the laptop with you, AI everywhere
* Efficient: far less power draw than a maxed-out GPU
* Quiet: no extra PSU or cooling needed
* Large memory options: up to 128GB

***

## Scenarios that really need NVIDIA

Some jobs are painful on a Mac and genuinely want a real GPU:

* **Training and fine-tuning**: especially full-parameter fine-tuning — CUDA plus large VRAM is almost a requirement
* **70B+ models**: need high bandwidth and lots of memory
* **Multi-user servers**: serving many people concurrently needs high throughput
* **Maximum speed**: latency-sensitive, per-token applications
* **Complex ecosystem tools**: many frameworks only have a CUDA path

### NVIDIA strengths

* Bandwidth and throughput far beyond a Mac
* Multi-GPU scaling (NVLink, multiple cards)
* A complete toolchain: vLLM, TensorRT, DeepSpeed
* Industry standard with the most community documentation

### NVIDIA limitations

* VRAM is fixed and expensive: 8-32GB consumer, only 48GB+ on workstations
* Power and cooling: a 4090 pulls around 450W at full load
* Requires a desktop or server environment

***

## Side-by-side comparison

| Aspect | Apple Silicon | NVIDIA GPU |
|--------|--------------|-----------|
| Memory model | Unified (RAM is VRAM) | Dedicated VRAM |
| Memory ceiling | Up to 128GB (can run 70B+) | 32GB max consumer |
| Bandwidth | 100-800 GB/s | 300-3350 GB/s |
| Software stack | Metal / MLX / MPS | CUDA / cuDNN / TensorRT |
| Best for | 7B-32B, personal use | 70B+, training, serving |
| Power | Low (10-70W) | High (200-575W) |
| Portability | Built into the laptop | Needs a slot / host |

***

## How to choose, by need

**Pick a Mac if:**

* You mainly run 7B-14B, occasionally 32B
* You need portability, low power, and quiet
* You want one machine for daily development plus AI

**Pick NVIDIA if:**

* You fine-tune or train models
* You run 70B+ or serve multiple users
* You want the absolute best tokens-per-second
* You already have, or are willing to invest in, a workstation

### The most practical path

If you'll touch both worlds, prefer cross-platform tooling like **llama.cpp + GGUF** so the same workflow moves smoothly between Mac and NVIDIA. Get the pipeline working on your Mac first, then migrate to a GPU box when you need more scale.

***

## Summary

* How big a model you can run depends on **memory capacity**; how fast depends on **memory bandwidth**
* Mac's unified memory makes "big capacity" cheap — great for personal use and mid-size models
* NVIDIA's VRAM is expensive and fixed, but bandwidth and the software ecosystem lead
* llama.cpp is the piece both sides share
