# Build a Private Local RAG Stack

> 📅 2026-08-01 · Getting Started
> Combine a local embedding model, a vector store, and a local LLM into a RAG pipeline where privacy and cost are yours to control.

---

In the first three posts we learned to run models and understood the hardware. This final post of the series stitches those skills into a real application: **local RAG**. Get an LLM to read your own documents — and keep the data on your machine, always.

***

## Why local RAG

RAG (Retrieval-Augmented Generation) makes a model retrieve relevant passages from your document store before answering, then generate from them. It's the standard way to make an LLM "understand your documents."

With a cloud API, your documents get sent to a third party first. Some files — medical, financial, internal company data — shouldn't leave your machine at all.

**Benefits of local RAG:**

* Privacy: documents and queries never leave the machine
* Cost: nothing per call, only hardware and electricity
* Control: model, embeddings, and database are all self-managed
* Offline: works without a network

**The tradeoff:** results are usually a bit behind top cloud models, and you maintain the pipeline yourself.

***

## The three pieces

A RAG pipeline is built from three components: an **embedding model** turns text into vectors, a **vector store** does similarity search, and an **LLM** generates answers from the retrieved content. All three run locally:

| Component | Local choice |
|-----------|--------------|
| Embedding model | `nomic-embed-text` (via Ollama) |
| Vector store | Chroma (local PersistentClient) |
| LLM | `qwen2.5:7b` (via Ollama) |

***

## Install and pull the models

```bash
pip install chromadb ollama
```

```bash
ollama pull nomic-embed-text
ollama pull qwen2.5:7b
```

`nomic-embed-text` is Ollama's embedding model, about 274MB; `qwen2.5:7b` is the generator. If your machine has little memory, switch to `qwen2.5:3b` instead.

***

## Step 1: chunk your documents

The first step of RAG is splitting documents into *retrievable pieces*. Too small and you lose context; too big and retrieval precision drops. So you set a **chunk size and overlap**.

```python
def chunk_text(text: str, chunk_size: int = 500, overlap: int = 100) -> list[str]:
    words = text.split()
    chunks = []
    step = chunk_size - overlap
    for i in range(0, len(words), step):
        chunk = " ".join(words[i : i + chunk_size])
        chunks.append(chunk)
    return chunks
```

### Chunking rules of thumb

* Aim for 300-800 words per chunk (or 400-1000 tokens)
* Leave 10-20% overlap between chunks so key sentences aren't cut

***

## Step 2: embed and index

Convert each chunk into a vector with the embedding model, then store it in the vector database.

```python
import ollama
import chromadb

client = chromadb.PersistentClient(path="./notes_db")
collection = client.get_or_create_collection(name="notes")

docs = [
    "Our pricing plans are Free and Pro; Pro costs $9 per month.",
    "Refund policy: paid subscriptions can be refunded within 14 days, no questions asked.",
    # ... your document chunks
]

ids = [f"note-{i}" for i in range(len(docs))]
embeddings = [
    ollama.embed(model="nomic-embed-text", input=d)["embeddings"][0]
    for d in docs
]

collection.add(ids=ids, documents=docs, embeddings=embeddings)
```

`PersistentClient` saves data to the local `./notes_db` directory — it survives restarts.

***

## Step 3: query and answer

At query time, embed the question too, find the closest chunks in the store, then feed those chunks plus the question to the LLM.

```python
def ask(question: str) -> str:
    q_emb = ollama.embed(model="nomic-embed-text", input=question)["embeddings"][0]

    hits = collection.query(query_embeddings=[q_emb], n_results=3)
    context = "\n\n".join(hits["documents"][0])

    prompt = f"""You are a private personal assistant. Answer the question using ONLY the context below.

Context:
{context}

Question: {question}
Answer:"""

    response = ollama.chat(
        model="qwen2.5:7b",
        messages=[{"role": "user", "content": prompt}],
    )
    return response["message"]["content"]
```

### Why "using only the context"

That instruction is important. It asks the model not to answer from memory, restricting output to the retrieved content — this is the key to reducing hallucination in RAG.

***

## A complete runnable pipeline

Put the pieces together and you have a full local RAG:

```python
import ollama
import chromadb

def chunk_text(text: str, chunk_size: int = 500, overlap: int = 100):
    words = text.split()
    step = chunk_size - overlap
    return [" ".join(words[i : i + chunk_size]) for i in range(0, len(words), step)]

def build_index(file_path: str, db_path: str = "./notes_db", name: str = "notes"):
    client = chromadb.PersistentClient(path=db_path)
    collection = client.get_or_create_collection(name=name)

    raw = open(file_path, encoding="utf-8").read()
    chunks = chunk_text(raw)

    ids = [f"{file_path}-{i}" for i in range(len(chunks))]
    embeddings = [
        ollama.embed(model="nomic-embed-text", input=c)["embeddings"][0]
        for c in chunks
    ]
    collection.add(ids=ids, documents=chunks, embeddings=embeddings)
    return collection

def ask(question: str, collection) -> str:
    q_emb = ollama.embed(model="nomic-embed-text", input=question)["embeddings"][0]
    hits = collection.query(query_embeddings=[q_emb], n_results=3)
    context = "\n\n".join(hits["documents"][0])

    prompt = f"Answer the question using only the context below.\n\nContext:\n{context}\n\nQuestion: {question}"
    response = ollama.chat(
        model="qwen2.5:7b",
        messages=[{"role": "user", "content": prompt}],
    )
    return response["message"]["content"]

# Build the index once
collection = build_index("my_notes.txt")

# Query again and again
print(ask("What is our refund policy?", collection))
```

Make sure Ollama is running first (or run `ollama run` once to start it in the background).

***

## Privacy and cost: the math

| Item | Cloud RAG | Local RAG |
|------|-----------|-----------|
| Data leaves your machine | Yes | No |
| Cost per query | Per token | 0 |
| Hardware cost | 0 | One-time machine expense |
| Electricity | 0 | A little |
| Privacy risk | Depends on the vendor | Yours to control |

Local RAG has near-zero marginal cost: once the models are downloaded, ten thousand queries cost nothing extra.

***

## Ways to extend this

This pipeline can grow in several directions:

* **Better embeddings**: `bge-m3` supports many languages and handles Chinese better
* **Bigger vector stores**: switch to Qdrant or LanceDB for millions of vectors
* **More document formats**: add PDF / DOCX parsing (e.g. `pypdf`)

***

## Summary

* RAG = embedding model + vector store + LLM, all three can run locally
* Chunk → embed → retrieve → generate: four fixed steps
* The key selling points of local RAG are privacy and zero marginal cost
* Get the minimal pipeline running first, then upgrade piece by piece
