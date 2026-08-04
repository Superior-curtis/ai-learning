# RAG from First Principles: Retrieval-Augmented Generation

> 📅 2026-08-01 · Core Mechanics
> Starting from the fact that a model's knowledge is frozen, we build the whole pipeline — chunking, indexing, retrieval, generation — and the most common pitfalls.

---

RAG (Retrieval-Augmented Generation) is the dominant way to make LLMs answer questions beyond their own knowledge. This post does not start from an API — it goes back to first principles: why models "don't know" things in the first place, then assembles the entire pipeline from scratch.

## First principles: a model's knowledge is frozen

An LLM's abilities come from the billions of examples it saw during training. The moment training ends, its knowledge of the world is "frozen":

* **It does not know what happened after its training cutoff** ("last June's news?" — nope).
* **It has never seen your private documents** (your product manual, your internal policies).
* **It confidently makes things up** — when it doesn't know an answer, there is no mechanism to say "I don't know." It tends to produce something plausible-sounding. That is a hallucination.

RAG's response is straightforward: **don't let the model answer from memory. First retrieve the relevant content from an external database, stuff it into the prompt, and let the model answer "based on the provided references."**

```
Without RAG:  question → LLM (from memory) → may fabricate
With RAG:     question → retriever (finds answer in YOUR database)
                  ↓
          question + relevant passages → LLM → cited answer
```

It's like handing the model a book it can consult, rather than demanding it have the whole book memorized.

## The pipeline: four stages

A RAG system has four stages — the first two are **offline preparation**, the last two are **online serving**:

```
Offline:  documents → chunk → embed → build index
Online:   question → embed → retrieve top-k → assemble prompt → LLM generates
```

### Offline: chunking

Documents are too long to embed as one useful vector, so they must be split into smaller pieces:

* **Chunk size:** commonly 300–800 tokens. Too small and there isn't enough context; too large and the semantics get diluted and irrelevant content sneaks in.
* **Overlap:** adjacent chunks overlap by 10–20% so a key sentence is never cut exactly at a boundary.
* **Splitting strategy:** split by structure first (headings, sections, paragraphs), then by fixed length. Keeping the heading information noticeably improves retrieval.
* **A useful trick:** prepend context (like the section title) to each chunk before embedding — "contextual chunking."

### Offline: embedding and indexing

Each chunk is converted to a vector by an embedding model and stored in a vector database (see the previous post). You also store each chunk's original text and metadata, so that after retrieval you can get the text back.

### Online: retrieval

The user's question is also embedded, and the index returns the K most similar chunks (typically K = 3–10). Good retrieval returns the key evidence the question needs; bad retrieval returns a pile of related-but-useless paragraphs.

### Online: generation

The question and the retrieved passages are assembled into a prompt, with instructions to answer only from the provided material and cite sources.

```python
system_prompt = (
    "You are a helpful assistant. Answer using ONLY the context below. "
    "If the context does not contain the answer, say you don't know. "
    "Cite the source of each claim.\n\n"
    "### CONTEXT ###\n{context}"
)

user_query = "Our refund policy: how many days, and any exceptions?"

# context = "\n\n".join(f"[doc {i}] {chunk.text}" for i, chunk in enumerate(top_k))
final_prompt = system_prompt.format(context=context) + f"\n\n### QUESTION ###\n{user_query}"
```

Three points matter in the generation stage:

* **The role instruction must force citation:** without "only from the provided data," the model will leak in its own memory.
* **Give the model room to say "I don't know":** this dramatically reduces hallucinations.
* **Track provenance:** ask the model to mark which document each claim came from, so users can verify.

## Full pipeline (conceptual)

Here is an end-to-end example using conceptual code (check the exact API for your library versions):

```python
# pip install sentence-transformers chromadb
from sentence_transformers import SentenceTransformer
import chromadb

embedder = SentenceTransformer("BAAI/bge-small-en-v1.5")
client = chromadb.PersistentClient(path="./chroma")
collection = client.get_or_create_collection("docs")

# --- Offline: chunk → embed → index ---
def chunk_document(text, size=400, overlap=80):
    chunks = []
    start = 0
    while start < len(text):
        chunks.append(text[start : start + size])
        start += size - overlap
    return chunks

for doc_id, text in documents.items():
    chunks = chunk_document(text)
    vectors = embedder.encode(chunks).tolist()
    collection.add(
        ids=[f"{doc_id}:{i}" for i in range(len(chunks))],
        embeddings=vectors,
        documents=chunks,
        metadatas=[{"doc_id": doc_id} for _ in chunks],
    )

# --- Online: retrieval ---
query = "What is the refund window?"
q_vec = embedder.encode([query]).tolist()
top_k = collection.query(query_embeddings=q_vec, n_results=5)

# --- Online: generation ---
context = "\n\n".join(top_k["documents"][0])
final_prompt = build_prompt(query, context)   # see the example above
answer = call_llm(final_prompt)               # call your model API
```

## The most common pitfalls

RAG systems are easy to get "running" and hard to get "right." Nine out of ten problems live in retrieval, not generation:

| Pitfall | Symptom | Fix |
|---------|---------|-----|
| Chunking splits the answer in half | the answer exists but is never retrieved | add overlap, split by structure, contextual chunking |
| Embedding model not suited for retrieval | retrieved results are off-topic | switch to retrieval-tuned embeddings (BGE / E5 / GTE) |
| Retrieved passages are irrelevant | context is full of noise, answer gets derailed | lower K, add metadata filtering, add a reranker |
| Data is stale or missing | no answer, and nobody knows it's a data gap | build an update mechanism and quality checks |
| Prompt doesn't force citation | the model still uses its own memory | explicitly say "use only the context" + allow "I don't know" |
| No evaluation | you can't tell if changes help | build a query→answer test set, measure Recall / accuracy |

And one conceptual trap: **treating RAG as a silver bullet that fixes everything.** If the question needs reasoning across multiple documents, or needs real-time data, a single retrieve-then-generate pass often isn't enough — you need multi-hop retrieval or tool calling.

## Summary

| Stage | What it does | Common tools |
|-------|--------------|--------------|
| Chunking | split documents into semantically complete pieces | custom code, LangChain splitters |
| Indexing | chunk → vector → store in a vector DB | sentence-transformers + FAISS / Chroma |
| Retrieval | embed the question, take top-k similar chunks | FAISS, Chroma, Qdrant |
| Generation | question + context → model answer | any LLM API |

RAG's first principle is one sentence: **the model doesn't know your data, so put the data in front of it.** Nail the four stages — above all retrieval quality — and you can get a model to answer reliably, with citations, from your private documents.

Next, we look at the opposite optimization direction: not giving the model more data, but making the model itself take up less memory — quantization.
