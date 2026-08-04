# Vector DBs with RAG: Turning Your Store Into an Answering Machine

> 📅 2026-08-04 · Getting Started
> Tool picked, now wire embedding, indexing, retrieval, and generation into one end-to-end pipeline: feed documents, build an index, retrieve, feed the LLM — then win back precision with metadata filtering and hybrid retrieval.

---

You picked your vector database (`vectordb-02-comparison`). But what you actually want is not "a database" — it is a **thing that answers user questions**. In this post we embed that piece properly into RAG: from feeding in documents, all the way to a model producing an answer backed by the right passages.

I will walk one **complete end-to-end pipeline**, and demonstrate the two things that move it from "it runs" to "it works": metadata filtering, plus vector-and-keyword hybrid retrieval (the working version of `rag-03-hybrid`).

## The whole RAG in four beats

Stringing together the parts from `rag-02-chunking` through `rag-03-hybrid` gives a loop that is really just four beats:

```
Beat ① Embed  document → chunk → turn each chunk into a vector
Beat ② Index  vectors → store in the vector DB, with metadata
Beat ③ Retrieve  query → embed → filter + hybrid → return top-K
Beat ④ Generate  feed "query + retrieved chunks" to the LLM
```

The vector store owns beats ② and ③. Earlier, `rag-02-chunking` decided "what can be retrieved"; here we decide "how fast and how accurately." Walk all four beats; none can be skipped.

## A minimal end-to-end pipeline

For anyone still distracted by tooling, here is the smallest complete version — plain Python you can actually run (conceptual; use the API of whatever library version you choose):

```Conceptual:&#x20;end-to-end&#x20;RAG&#x20;(Qdrant&#x20;example)
# Conceptual; use the API of whatever library version you choose
from sentence_transformers import SentenceTransformer
from qdrant_client import QdrantClient
from qdrant_client.models import Distance, VectorParams

embedder = SentenceTransformer("BAAI/bge-small-en-v1.5")
client = QdrantClient(":memory:")          # in-memory for demo; use a real service in prod

# Beat 1 embed: docs is a list of chunks, each with metadata
docs = [
  {"text": "Refund policy: full refund within 30 days", "meta": {"dept": "cs", "y": 2026}},
  {"text": "Leave rules: 10 days per year, may carry over", "meta": {"dept": "hr", "y": 2026}},
]
client.create_collection(
  "handbook",
  vectors_config=VectorParams(size=384, distance=Distance.COSINE),
)

# Beat 2 index: write vector and metadata together
for i, d in enumerate(docs):
  vec = embedder.encode(d["text"]).tolist()
  client.upsert(collection_name="handbook", points=[
      {"id": i, "vector": vec, "payload": d["meta"]}
  ])

# Beat 3 retrieve: filter first (customer service only), then nearest
q = "Can I get a refund"
qvec = embedder.encode(q).tolist()
hits = client.query_points(
  collection_name="handbook",
  query=qvec,
  query_filter={"must": [{"key": "dept", "match": {"value": "cs"}}]},
  limit=2,
).points

# Beat 4 generate: feed only the pulled passages to the LLM
context = "
".join(p.payload.get("_text", "") for p in hits)
# in production: send context + question to the model (omitted here)
print(context)
```

Note the comment at Beat 4: **you feed only the retrieved passages to the model — not the whole vector store.** That is the essence of RAG. Retrieval trims first, so generation is neither flooded nor left to make things up.

## Metadata filtering: the cheapest source of precision

`rag-02-chunking` wanted every chunk to carry metadata. That is not for decoration — it enables "**filter first, then compute distance**," usually the most efficient single precision win:

```
filter (cheap, narrows the scope first)
  └→ e.g. dept = customer service → search only those documents
vector distance (costlier, ranks within the narrowed scope)
  └→ among service docs, find the most similar by meaning
```

Why is it cheaper than adding a reranker? Because **filtering is Boolean and deterministic** — trivially cheap and reliably effective — whereas reranking turns "similar" into "right" and is expensive, something you can add later. The order is always: **filter first, then hybrid, then rerank if you need it.** Skip filtering and you are using an expensive index to do coarse work it was never meant for.

> The one thing to remember: filtering is cheaper and more effective than ranking — narrow the scope to what you should be looking at with metadata, then let vectors find the nearest within that narrowed scope. That is the dividing line between RAG that works and RAG that merely responds.

## Hybrid retrieval: when vectors cannot carry it alone

Pure vector search (the craft from `llm-02-embeddings`) is strong at "similar meaning" and oddly weak at "this exact string." Real users are endlessly searching for things like `SOP-2024-0715` — something literal. Whole article `rag-03-hybrid` is devoted to solving exactly this.

In practice, vector stores usually let you hold both vector and keyword signals in **the same query**:

```
score(chunk) = RRF fusion of (vector rank + BM25 rank)   # the fine print in `rag-03-hybrid`
order: metadata filter -> hybrid -> rerank if needed
```

For a RAG that "over-fetches and lets the next stage refine," hybrid is almost always at least as good as pure vector, and clearly better on exact strings. `rag-03-hybrid` shows how to fuse the two scores and when a reranker is worth it.

## Three pitfalls in practice

Once the pipeline runs, the real crashes happen in the details. Knowing these three traps saves you a lot of late-night debugging:

* **Documents get updated, vectors do not.** You edit a document, but the old chunk vector still sits in the index, and users cannot find the new content. Fix: upsert with a stable `doc_id` (overwrite, not append), and delete by metadata — do not let ghost vectors quietly accumulate.
* **top-k too small or too large.** Too small → not enough context and the model guesses; too large → the window floods and tokens burn. Start at 3–10, then adjust against the reranking results from `rag-03-hybrid`; do not stuff a hundred passages into the prompt.
* **Filter returns zero candidates.** A metadata condition that is too strict (say, a department with no documents) empties the pool and retrieval comes back empty-handed. Before shipping, design the fallback for "filtered-to-nothing" — falling back to unfiltered beats a blank page.

## Before shipping, walk this order

Do not chase perfection on day one. RAG improves by increments — thicken it in this order:

#### Get the minimal pipeline running

Feed a few documents, index, retrieve, feed the LLM, and confirm it answers. This step is about getting it to work, not about perfection.

#### Add metadata and filtering

Give every chunk a dept, timestamp, and source, and filter before retrieval. Often the single highest-yield improvement.

#### Add hybrid retrieval

Bring in keyword signal for queries about exact codes and strings, fused with the RRF from `rag-03-hybrid`.

#### Measure it, then decide on reranking

Build a small eval set and run it a few rounds; let the numbers decide whether a cross-encoder reranker is worth it. Do not go by feel.

## Next up: the variable you cannot feel your way through

Chunking, embedding, retrieval, filtering, hybridization — all in place. Congratulations, you have a system that genuinely answers. But with a full toolbox, one variable remains: **how do you know it got better?**

You retuned the embedder or tweaked a filter — did it improve or regress? "It feels smoother" is not an answer. To turn that into comparable numbers you need an eval set you can run repeatedly — the last and most valuable piece of the RAG series. Re-read the closings of `rag-03-hybrid` and `rag-02-chunking` and you will see each step was already paving the way toward "being measurable." Add measurement, and your vector store becomes a real answering machine.

#### Q: Why is 'filter by metadata first, then compute vector distance' usually the most worthwhile way to improve precision?

* Filtering requires an expensive model but is more accurate

* Filtering is a deterministic Boolean condition with low cost; it narrows the scope first so vectors rank only where they should, cheaper and more effective than ranking-then-reranking

* Filtering can fully replace vector search

* Filtering only matters for very large databases

> 💡 Filtering is a cheap Boolean operation that narrows the scope (e.g. only customer-service docs), then vector distance ranks within that narrowed space — far more efficient than the expensive reranking step that turns similar into right. The order is filter, then hybrid, then rerank only when needed.
