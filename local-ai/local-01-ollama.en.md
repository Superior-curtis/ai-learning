# Run Your First Local Model with Ollama

> 📅 2026-08-01 · Getting Started
> From installation to pulling a model to your first conversation — get your first local LLM running from scratch.

---

This is the first post in the "Local AI" series. The goal is simple: get a real conversational language model running on your own computer, fully offline, without spending a cent on API calls.

***

## What is Ollama

Ollama is a tool that turns "running a local model" into a single command. You don't need to know CUDA, compile C++ yourself, or handle GGUF files — it wraps downloading models, starting an inference server, and providing a chat interface into one package.

**What Ollama handles for you:**

* Downloading models with one command
* Automatically picking the CPU / GPU backend
* A built-in chat interface and REST API
* Support for macOS, Linux, and Windows

### Why start with Ollama

Because it has the lowest barrier to entry. Other tools (like llama.cpp) are more powerful and flexible, but they require more low-level knowledge. Get the feeling of "local models actually work" with Ollama first, then go deeper — the learning curve is much smoother that way.

***

## Installing Ollama

### macOS (Homebrew)

```bash
brew install ollama
```

### Linux and other macOS

```bash
curl -fsSL https://ollama.com/install.sh | sh
```

### Windows

Download the installer from ollama.com/download and run it — Ollama starts in the background automatically.

### Verify the installation

```bash
ollama --version
```

The first time you run `ollama run`, if the server isn't started yet, Ollama starts it in the background for you.

***

## Pull your first model

Ollama's model library lives at ollama.com/library. The selection rule is: "is your memory big enough?" Start with a beginner-friendly 3B model:

```bash
ollama pull llama3.2
```

This model is about 2GB and runs even on older machines with only 8GB of RAM.

If your machine has even less memory, switch to a smaller 1.5B model:

```bash
ollama pull qwen2.5:1.5b
```

### List downloaded models

```bash
ollama list
```

`ollama list` shows the model name, tag, size, and last-modified time.

***

## Start your first conversation

```bash
ollama run llama3.2
```

When you see the `>>>` prompt, the model is loaded and ready. Just type and chat:

```
>>> In one sentence, what is a "cache"?
A cache is a place that stores frequently used data for faster access.

>>> /bye
```

Type `/bye` to end the session.

### Pass a prompt as an argument

If you don't want interactive mode, pass a prompt directly:

```bash
ollama run llama3.2 "Why is the sky blue?"
```

This form is great for quick tests — it doesn't drop you into the interactive interface.

### Tune parameters during a session

Ollama's interactive mode supports `/set` commands to adjust generation parameters on the fly:

```
>>> /set temperature 0.7
>>> /set num_ctx 8192
```

* `temperature`: higher means more random output, lower means more deterministic.
* `num_ctx`: the context length; larger lets the model remember more, but uses more memory.

***

## Managing running models

### See which models are loaded

```bash
ollama ps
```

### Stop a specific model

```bash
ollama stop llama3.2
```

Models unload automatically after 5 minutes of inactivity, so you rarely need to stop them manually.

***

## Calling the local model via API

What Ollama runs in the background is a local server listening on `http://localhost:11434`. Any program that speaks an OpenAI-compatible interface can hit this endpoint directly.

```bash
curl http://localhost:11434/api/chat -d '{
  "model": "llama3.2",
  "messages": [{"role": "user", "content": "Hello!"}],
  "stream": false
}'
```

The `message.content` field in the returned JSON is the model's answer. `stream: false` means the full response comes back at once, which is easier to read.

***

## Building your own model with a Modelfile

Ollama provides a file format called a Modelfile that lets you define a system prompt and default parameters. Create a file first:

```
FROM llama3.2

SYSTEM You are a senior TypeScript engineer. Keep answers short, specific, and include examples.
PARAMETER temperature 0.2
```

Then create it as a new model:

```bash
ollama create mycoder -f Modelfile
```

After that, use it directly:

```bash
ollama run mycoder
```

***

## Next steps

You've now run your first local model. From here you can go in three directions:

* Want to understand how the model works under the hood? Read the next post: llama.cpp and GGUF.
* Want to know what size model your Mac can run? Check the hardware comparison post.
* Want the model to read your own documents? Read the final post in the series: local RAG.

Get comfortable with `ollama run` first — every later post builds on it.
