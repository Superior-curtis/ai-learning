# Embeddings and Vector Search: Putting Semantics into Coordinates

> 📅 2026-08-01 · Core Mechanics
> What embeddings are, how cosine similarity works, what vector databases do, and when retrieval succeeds and fails.

---

In the previous post we saw that every word inside a model is a vector. This post answers: **where do those vectors come from, and what are they good for?** The answer is embeddings — turning text into coordinates so that "similar meaning" becomes "close in space," and vector search is simply finding nearest neighbors in that space.

## What an embedding is

An embedding maps a token, sentence, or document into a fixed-dimension vector of real numbers. With the `sentence-transformers` library, a sentence becomes a 384- or 1024-dimensional vector:

```python
from sentence_transformers import SentenceTransformer

model = SentenceTransformer("BAAI/bge-small-en-v1.5")

vec_a = model.encode("The cat sat on the mat")
vec_b = model.encode("A dog lay on the floor")
vec_c = model.encode("Interest rates rose this quarter")

print(vec_a.shape)  # (384,)
```

The important thing is not the dimension — it is the **position**. In space, `vec_a` and `vec_b` are close together (both animals on floors), while `vec_c` is far away (it's finance news).

### From a "dictionary" to a "semantic map"

Early methods (one-hot, TF-IDF) treated words as independent identifiers: "cat" and "dog" shared nothing because they live in separate columns of a vocabulary.

The key breakthrough of embeddings: **turn meaning into continuous coordinates.** In the learned vector space, similar words cluster together, and differences between vectors carry directional meaning — the classic example:

```
vector("king") - vector("man") + vector("woman") ≈ vector("queen")
```

Embeddings capture not just "dictionary definitions" but the relational structure between words. Modern sentence embeddings follow the same philosophy, upgraded from "words" to "whole sentences."

## How to compare "closeness"

Given vectors, the remaining question is how to measure similarity. The most common answer is **cosine similarity**, which looks only at the direction of two vectors, not their magnitude — ideal for text, where sentence length shouldn't affect the meaning judgment.

```
cos_sim(A, B) = (A · B) / (|A| · |B|) = Σᵢ AᵢBᵢ / (√Σᵢ Aᵢ² · √Σᵢ Bᵢ²)
```

It ranges from -1 to 1: 1 means the same direction (very similar), 0 means orthogonal (unrelated), -1 means opposite.

```python
import numpy as np

def cosine_similarity(a, b):
    a = np.asarray(a, dtype=np.float32)
    b = np.asarray(b, dtype=np.float32)
    return float((a @ b) / (np.linalg.norm(a) * np.linalg.norm(b)))

print(cosine_similarity(vec_a, vec_b))  # high, e.g. 0.62
print(cosine_similarity(vec_a, vec_c))  # low, e.g. 0.05
```

In practice you usually normalize vectors first (`a / ||a||`), which turns cosine similarity into a plain dot product and speeds up the computation.

## What vector databases do

"Find the K nearest vectors" sounds simple, but with 10 million documents, computing cosine similarity one by one is far too slow. Vector databases solve exactly this problem:

* **Index structures:** approximate nearest neighbor (ANN) algorithms such as HNSW (hierarchical navigable small world) or IVF (inverted file) bring search down to milliseconds.
* **Dynamic updates:** add new vectors and delete old ones on the fly.
* **Hybrid filtering:** filter by metadata (tags, dates, permissions) before the vector search runs.

Common options: FAISS (Facebook's library), Chroma (lightweight, good for small projects), Qdrant, Pinecone, Weaviate, and Milvus.

### Minimal working example: FAISS

```python
import faiss
import numpy as np

# assume all vectors are normalized (unit length)
embeddings = np.random.rand(10_000, 384).astype("float32")
faiss.normalize_L2(embeddings)

index = faiss.IndexFlatIP(384)   # dot product == cosine after normalization
index.add(embeddings)            # build the index

query = np.random.rand(1, 384).astype("float32")
faiss.normalize_L2(query)

scores, ids = index.search(query, k=5)  # top-5 nearest neighbors
print(ids, scores)
```

For larger scales, swap in `IndexHNSWFlat` and search stays in the milliseconds.

## When retrieval works

* **Semantic paraphrase:** asking "how to stop a dog from barking" can recall "behavioral correction for barking" — even with no shared keywords.
* **Cross-lingual matching:** good multilingual embeddings place sentences with the same meaning close together across languages.
* **Fuzzy queries:** when users can't remember the exact wording, vector search still finds relevant content.

## When retrieval fails

Vector search is not a silver bullet. Common failure modes:

| Problem | Why it happens | What to do |
|---------|----------------|------------|
| Exact-match fails | searching "API v2.3" returns "API v3.1" | hybrid search with keyword / exact matching |
| Multiple conditions required | cosine is a soft ranking, no boolean logic | filter by metadata first, then rank |
| Embeddings don't know rare tokens | proper nouns, code snippets are rare in training data | use a bigger model, or add synonym expansion |
| Semantics are too subtle | titles vs. bodies can't be distinguished by one vector | try multi-field / multi-vector retrieval |
| The data itself is too similar | thousands of insurance clauses, top-k is all near-duplicates | use a reranker to break the tie |

There is also a frequently ignored trap: **embedding model and retrieval task mismatch.** Using an embedding trained for classification for retrieval usually works poorly. Choose a dedicated retrieval embedding (BGE, E5, GTE) and, ideally, fine-tune it on your own data.

## Evaluating retrieval quality

Measure retrieval with three simple metrics:

* **Recall@k:** does the correct document appear in the top k?
* **Precision@k:** of the top k, how many are actually correct?
* **MRR (Mean Reciprocal Rank):** how early does the first correct document rank?

Once you have a benchmark (query → expected document), a single evaluation run tells you whether switching the embedding model, changing chunk size, or adding metadata filtering actually helps or hurts.

## Summary

| Concept | In one sentence |
|---------|-----------------|
| Embedding | map text to vectors so similar meaning = close in space |
| Cosine similarity | compare vectors by direction; well suited to text |
| Vector database | ANN index for millisecond search over millions of vectors |
| Success factors | retrieval-tuned embeddings + good chunks + metadata filtering |
| Common failures | exact match, compound conditions, rare tokens, overly similar data |

Embeddings turn "understanding meaning" into "computing distance" — and that is the first building block of RAG. Next, we'll assemble embeddings, retrieval, and generation into a full RAG pipeline.
