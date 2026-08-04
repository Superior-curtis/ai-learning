# Vector Database Showdown: From PostgreSQL to Dedicated Services

> 📅 2026-08-04 · Tools
> pgvector, Qdrant, Milvus, Pinecone, Weaviate each have their own personality. The first decision is never 'which is strongest' but 'hosted or self-hosted, in my existing database or a dedicated service'.

---

The previous post covered what a vector store does and when you need one. Now you actually have to pick one, so you search around and find pgvector, Qdrant, Milvus, Pinecone, Weaviate, Chroma, FAISS… more options than you can shake a stick at.

Good news: **choosing between these tools is like choosing between models — not "which is strongest" but "where you are and how far you plan to go."** This post puts the five mainstream options on the same table, with an honest comparison and a decision guide you can just follow.

## Get the boundary conditions right before comparing tools

Before comparing vendors, three questions will already eliminate half the options:

1. **Where does your data already live?** If you are already on Postgres, putting vectors into pgvector is usually far cheaper than standing up a separate vector service. Existing infrastructure is often the deciding factor.
2. **Self-host or have a service carry the load?** This decides whether you spend your ops skill or your money.
3. **What is the data volume and query pattern?** Tens of thousands with a simple filter is fine anywhere. Millions, high concurrency, and hybrid retrieval (`rag-03-hybrid`) are what actually force you into a "real" vector store.

Answer those three and the remaining choice space is narrow.

## The five contenders at a glance

| Option | Form | Positioning in one line | Embedded or standalone |
|---|---|---|---|
| **pgvector** | Postgres extension | Put vector search back into the database you already run | Embedded |
| **Qdrant** | Standalone service | Lightweight, focused on vectors, friendly API | Standalone |
| **Milvus** | Standalone service | Built for massive scale, richest ecosystem | Standalone |
| **Pinecone** | Fully managed | No ops, call and use | Standalone (hosted) |
| **Weaviate** | Standalone service | Built-in multimodal and GraphQL, handy hybrid search | Standalone |

Chroma and FAISS are also good, but they are a different animal: FAISS is a *library*, not a database (no persistence, filtering, or concurrency), and Chroma leans toward lightweight local setups. This post focuses on the five genuinely database-grade options.

## pgvector: keep everything in one place

pgvector is not a new database — it is an **extension to Postgres**. The data still lives in your existing tables; you just add a vector column:

* **The biggest win is "one less thing."** Documents, metadata, vectors, and user data all live in a single database. No syncing two systems.
* Supports both `IVFFlat` and `HNSW` indexes; rock solid up to hundreds of thousands of vectors. It only starts to strain at millions.
* Transactions (ACID), backups, and permissions all come along for free from Postgres — a trust bonus for any team.

The trade-off: **it will not fight for the last drop of vector performance.** At millions of vectors with extreme latency demands, a dedicated engine will beat it. But for "my data is already in Postgres," it is the default answer.

## Qdrant: lightweight and focused

Qdrant is a **standalone vector search service written in Rust**. Its personality is "focus":

* It does the vector-centric job and little else; the API is clean and easy to reason about, and deployment is light.
* A single Docker container starts it locally; there is also a managed Qdrant Cloud.
* Native payload (metadata) filtering with payload indexes, and hybrid retrieval works well.
* For mid-size projects, it is often the friendliest starting point among the standalone services.

## Milvus: built for enormous scale

Milvus **splits vectors, metadata, and storage into separate modules**, designed from the ground up for "hundreds of millions of vectors, distributed deployment":

* The most complete feature set: sharding, replication, fault tolerance, GPU acceleration — the works.
* The price is **the most complex architecture and deployment**. For most people, Milvus is the "come back when you are really big" option.
* Ops cost is high; it is a poor first vector database.

## Pinecone: outsource the entire operation

Pinecone is a **fully managed** vector service — you never touch a server:

* Works out of the box, one API call; scaling, backups, and high availability are all its problem.
* For teams that want to touch zero infrastructure and focus on the product, it is the smoothest experience.
* The cost is **money and coupling**: usage-based billing, and your data lives in someone else's house — moving out is painful.
* For prototypes and startups that cannot hire an infra team, it often wins instantly.

## Weaviate: multimodal and hybrid built in

Weaviate is distinctive in treating search as a first-class citizen: it manages **objects plus vectors**, not just vectors:

* Native support for **hybrid search (BM25 + vectors)** and the reranking mentioned in `rag-03-hybrid`; wiring them together is straightforward.
* Built-in multimodal modules (text, images, audio) and a GraphQL API.
* Available self-hosted or as a managed Weaviate Cloud.

## MMR and hybrid retrieval: who does it natively

`rag-03-hybrid` will tell you retrieval cannot ride on vectors alone. Here is how far each option goes out of the box:

| Option | Hybrid (vector + BM25) | MMR diversity | Built-in rerank |
|---|---|---|---|
| **pgvector** | Wire it yourself | Implement it yourself | None (roll your own in SQL) |
| **Qdrant** | Native Query API | Native | Experimental |
| **Milvus** | Native | Native | Native reranker |
| **Pinecone** | Native Hybrid Index | Native | Pair with third party |
| **Weaviate** | Native BM25 + vector | Native | Native |

**MMR (Maximum Marginal Relevance)** fixes a common RAG ailment: pure vector search often returns top-k results that are near-duplicate restatements of the same thing — say five passages all about "how to get a refund" and none about "the refund deadline." MMR adds a *diversity* bonus on top of *relevance*, so the returned passages stop repeating each other.

For RAG, **MMR is practically free quality**: it needs no new model, just a different ranking rule, yet it makes the passages fed to the LLM cover more ground. Unless your query natively wants a single "one answer" passage, enabling it is usually a pure win.

## Hosted vs. self-hosted: the real fork in the road

Everything above collapses into one crucial either-or:

| Dimension | Self-host (pgvector / Qdrant / Milvus) | Hosted (Pinecone / cloud editions) |
|---|---|---|
| You need | Ability to run servers and deploy | Just call an API |
| Data sovereignty | Fully yours | In the vendor data center |
| Upfront cost | Low (your own hardware) | Floats with usage |
| Flexibility | High, deeply tunable | Low, bounded by the API |
| Privacy / compliance sensitive | Home turf | May need a workaround |

Teams with sensitive data or a need for full control will almost always self-host; teams that want speed and no ops will go hosted. **Answer this one first, then "which vendor."**

> The one thing to remember: decide "hosted or self-hosted, in an existing database or a standalone service" first — then pick a brand. pgvector is the default when your data is already in Postgres; Qdrant is a light, good standalone start; Milvus is for massive scale; Pinecone is for those who never want to touch ops.

## Decision guide: walk through it once

Run your situation through a small flow:

```
① Data already lives in Postgres?   yes → pgvector (unless millions and latency-critical)
② Truly enormous (hundreds of millions)?  yes → Milvus
③ Do not want to touch ops?         yes → Pinecone (or each vendor's cloud edition)
④ Self-host, light standalone?      yes → Qdrant (multimodal/hybrid → Weaviate)
⑤ None of the above?                → Chroma / FAISS is fine, do not over-engineer
```

## How to read cost

"Free" and "expensive" are easily misread. The thing to compare is **total cost of ownership**:

* **Self-hosted**: hardware and power are fixed; more queries cost nothing extra — good for large, predictable traffic.
* **Hosted**: billed by storage and queries; cheap to start, floats up as volume grows — good for early-stage, spiky traffic.
* **pgvector**: because it shares your Postgres, it is almost "free" — but remember the extension still needs maintaining.

A pragmatic tip for beginners: **start with Qdrant (self-hosted) or Pinecone (hosted), get it running and measure, then think about upgrading to a heavier stack.** Do not shoulder a Milvus before you have a million vectors.

## Next up

You have a tool now. The next step is where all the tools work together: wiring embedding, indexing, retrieval, and generation into one end-to-end RAG — plus metadata filtering and hybrid retrieval. That is `vectordb-03-with-rag`.

#### Q: Someone says 'it is free anyway.' Which true advantage is being misread?

* Pinecone usage-based billing actually gets cheaper at large volume

* Self-hosted hardware costs are fixed and do not rise with queries — cheap in a total-cost-of-ownership sense

* pgvector needs no resources whatsoever

* Qdrant is free but cannot be used in production

> 💡 Self-hosted machines and power are fixed and do not grow with query volume (cheap in total cost); hosted billing floats up as volume grows. Read 'free/cheap' through total cost, not the unit price.
