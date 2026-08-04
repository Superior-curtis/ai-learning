# llama.cpp and GGUF, Explained

> 📅 2026-08-01 · Core Mechanics
> Dissecting llama.cpp and GGUF: quantization levels, the single-file format, and how to actually run inference.

---

Ollama is nice, but it's just the wrapper. The real workhorse underneath is an inference engine like llama.cpp. In this post we open it up: what llama.cpp is, what a GGUF file looks like, how quantization works, and finally how to actually run inference.

***

## What is llama.cpp

llama.cpp is an open-source project written in C/C++ with the goal of running large language models on ordinary consumer hardware — including CPUs with no dedicated GPU, Apple Silicon, and NVIDIA GPUs.

**Its main features:**

* Pure C/C++, no Python dependency — fast startup, low memory overhead
* CPU inference, Metal on Mac, CUDA on NVIDIA
* Reads GGUF-format quantized models
* Ships a CLI, a server, and bindings for many languages

### Why it matters

Before llama.cpp, running a 7B model usually required a high-end GPU. llama.cpp proved that with the right format and quantization, an ordinary laptop can produce acceptable quality and speed. It's a cornerstone of the local AI wave.

***

## GGUF: a single-file model format

GGUF is the model file format used by llama.cpp, and the de facto standard for local open-source models.

### Why GGUF exists

A raw Hugging Face model usually contains:

* Several `safetensors` weight files (potentially tens of GB)
* Config files like `tokenizer.json` and `config.json`
* Without any one of them, the model won't load

GGUF packs all of this into a **single file** that needs no external references to load. That design makes "download one file → start inferencing" possible.

### What's inside a GGUF file

* The model's tensor (weight) data
* Architecture and hyperparameters (layer count, dimensions, activation functions)
* Complete tokenizer data
* Quantization information (which quant, how many bits per tensor)

Because all the info is in the file, GGUF is self-describing — the engine doesn't need to know the model structure ahead of time.

***

## Quantization: trading a little quality for a lot of memory

Raw weights are usually stored as 16-bit floats (FP16). A 7B model's weights alone are about 14GB — too much for most people.

The idea behind quantization is simple: compress weights from 16 bits down to 4, 5, or 8 bits. The cost is slightly lower output quality, but memory usage drops dramatically.

**Memory formula:**

```
memory needed ≈ parameters × bits per weight / 8
```

A 7B model at Q4 (~4.5 bits/weight) → roughly 4GB.

***

## Common quantization levels

| Level | Approx. bits/weight | Approx. size (7B model) | Use case |
|-------|---------------------|--------------------------|----------|
| Q4_0 | ~4.1 | ~3.9GB | Smallest reasonable option, speed first |
| Q4_K_M | ~4.8 | ~4.4GB | The recommended default, best quality/size balance |
| Q5_K_M | ~5.5 | ~5.0GB | When you want better quality and have memory |
| Q6_K | ~6.6 | ~5.9GB | Close to original quality |
| Q8_0 | ~8.5 | ~7.6GB | Nearly lossless, needs more memory |
| F16 | 16 | ~14GB | Raw weights, for training/fine-tuning |

### How to choose

* Limited or unsure about memory → Q4_K_M
* Quality first, enough memory → Q5_K_M or Q6_K
* Just testing the tooling → Q4_0

***

## Building llama.cpp from source

Install CMake and a compiler toolchain first, then:

```bash
git clone https://github.com/ggml-org/llama.cpp
cd llama.cpp
cmake -B build
cmake --build build --config Release
```

After the build, the main binaries live in `build/bin/`:

* `llama-cli`: command-line inference
* `llama-server`: an HTTP server
* `llama-quantize`: the quantization tool

### Via Homebrew (macOS)

```bash
brew install llama.cpp
```

This gives you `llama-cli` directly, no compilation needed.

***

## Downloading a GGUF model

GGUF files usually live on Hugging Face. Here's Llama-2-7B-Chat in Q4_K_M as an example:

```bash
mkdir -p models
wget https://huggingface.co/TheBloke/Llama-2-7B-Chat-GGUF/resolve/main/llama-2-7b-chat.Q4_K_M.gguf -O models/llama-2-7b-chat.Q4_K_M.gguf
```

Many official models now ship GGUF files directly — just search "GGUF" on the Hugging Face page.

***

## Running inference with llama-cli

```bash
./build/bin/llama-cli \
  -m ./models/llama-2-7b-chat.Q4_K_M.gguf \
  -p "The sky is blue because" \
  -n 128
```

* `-m`: the model file
* `-p`: the prompt
* `-n`: maximum tokens to generate

### Common options

| Option | Effect |
|--------|--------|
| `-t` | Number of CPU threads |
| `-ngl N` | Offload the first N layers to the GPU (`99` on Mac = offload all) |
| `-c N` | Context length (e.g. `4096`) |
| `--temp X` | Temperature |
| `-i` | Interactive mode |

### Full GPU acceleration on Mac

```bash
./build/bin/llama-cli -m ./models/model.gguf -p "Hello" -n 64 -ngl 99
```

`-ngl 99` means offload as many layers as possible to Metal.

***

## Starting llama-server

llama.cpp also ships an OpenAI-compatible server:

```bash
./build/bin/llama-server -m ./models/llama-2-7b-chat.Q4_K_M.gguf --port 8080
```

Then call it with curl:

```bash
curl http://localhost:8080/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"model": "local", "messages": [{"role": "user", "content": "Hello!"}]}'
```

`/v1/chat/completions` is an OpenAI-compatible endpoint, so many existing tools can switch over directly.

***

## Converting and quantizing your own model

If you have a Hugging Face-format model, you can convert it to GGUF and quantize it yourself:

```bash
# 1. Convert to GGUF (F16)
python3 convert_hf_to_gguf.py ./my-hf-model --outfile mymodel-f16.gguf

# 2. Quantize to Q4_K_M
./build/bin/llama-quantize mymodel-f16.gguf mymodel-Q4_K_M.gguf Q4_K_M
```

`convert_hf_to_gguf.py` lives at the root of the llama.cpp repository.

***

## Summary

* llama.cpp is the low-level engine that makes local inference practical
* GGUF packs a model into a single self-describing file
* Quantization trades a little quality for a large memory saving
* `Q4_K_M` is the safe default for almost everyone
* Running llama-cli once gives you the core of local inference
