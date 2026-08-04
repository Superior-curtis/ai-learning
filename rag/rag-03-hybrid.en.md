# Hybrid Retrieval & Reranking: Vectors Are Not the Only Ranking Signal

> 📅 2026-08-04 · Deep Dive
> Vector search is great at 'semantically similar' and oddly bad at 'exactly this string'. Keyword + vector hybrid, then a cross-encoder rerank, is the most reliable retrieval recipe today.

---

Last article got the document chunked, embedded, and sitting in a vector index. Then you noticed something weird: ask "refund policy" and it finds things beautifully, but a user types an exact code — "SOP-2024-0715" — and vector search pulls back an irrelevant "2024 meeting minutes."

That is not a bug. It is the nature of vector search: it compares *meaning*, not *spelling*. And in the real world, users are often looking for exactly the literal thing. This article covers **hybrid retrieval** to fill that blind spot, then **reranking** to make the ordering match what users actually need.

## The vector blind spot: exact match

Embeddings turn "semantically similar" into "spatially close," which is great for synonyms and paraphrases. The cost:

* **Exact identifiers** (product codes, contract numbers, employee IDs) get diluted into something generic.
* **Proper nouns and acronyms** (say "CI/CD" or "LLM") get treated as ordinary words in vector space.
* **Rare but exact sentences**, when there are few semantically close candidates, all score mediocre.

`llm-02-embeddings` called embeddings "putting meaning into coordinate space" — and coordinate space has no concept of "identical string." Keyword search is the opposite: it is hypersensitive to the literal.

## BM25: the modern keyword retriever

Naive keyword search (scan the whole document for terms) is too crude. The modern baseline is **BM25**:

* For each query term, it counts how often it appears in the candidate.
* It penalizes terms that appear in *many* documents — words like "the" or "is" carry no signal, while rare terms discriminate.

```
query: "SOP-2024-0715"
BM25 scores → the chunk containing that exact string wins
Vector scores → a crowd of semantically-close-but-spelled-differently chunks tie
```

BM25 does not "understand" meaning, but it is the king of exact matching. Combining the two is literally "understanding" plus "being able to look it up."

## Hybrid: combine both signals

The intuition for hybrid retrieval is simple: **run both searches, then merge the results.** The problem is "how to merge" — vector scores and BM25 scores live on different scales and cannot be added directly.

The most common solution is **RRF (Reciprocal Rank Fusion)**: ignore the scores, look at the *ranks* instead. The better a candidate ranks in each list, the more it contributes:

```
score(doc) = sum over each list i of 1 / (k + rank_of_doc_in_list_i)   # k ≈ 60 is common
```

The #1-ranked candidate contributes the most, then decays fast. RRF's advantage: **no weight tuning, no scale mismatch** — it turns "this ranks #2 on vectors and #5 on keywords" into one comparable number.

Numbers make this concrete. Say the vector list and the BM25 list each have 4 candidates, with k = 60:

| Candidate | Vector rank | BM25 rank | RRF score (approx) |
|---|---|---|---|
| chunk A | 1 | 3 | 1/61 + 1/63 ≈ 0.0324 |
| chunk B | 2 | 5 | 1/62 + 1/65 ≈ 0.0315 |
| chunk C | 10 | 1 | 1/70 + 1/61 ≈ 0.0307 |
| chunk D | 5 | 4 | 1/65 + 1/64 ≈ 0.0310 |

chunk A ranks high on both lists, so it is safely first; chunk B stays close because it is #2 on vectors; chunk C is #1 on BM25 but drops to 10 on vectors, and its total falls below B and D. This is exactly what hybrid wants: **being good on both beats being great on one and missing on the other.**

### The other route: weighted score merging

RRF is the "rank camp." The other school is the "score camp" — normalize both sides to the same scale (say 0–1) and add them with a weight:

```
combined = α × vector_score + (1 − α) × bm25_score      # α in [0, 1]
```

The score camp's advantage is **you can tune α**: if your data is mostly exact codes, push the weight toward BM25; if it is long-form prose, push toward vectors. The cost is **one more hyperparameter to tune**, and the result drifts whenever the normalization scheme changes. For most people, RRF's "zero parameters" is the more stable starting point — get it running first, and switch to the score camp later if you truly need the fine control.

## Why hybrid beats vector-only

| Scenario | Vector-only | Hybrid (vector + BM25) |
|---|---|---|
| Paraphrase and synonym | Strong | Strong |
| Exact codes, models, numbers | Often missed | One hit |
| Acronyms, proper nouns | Diluted | Literal fallback |
| Rare but exact long phrases | Mediocre score | Can make the list |
| Slightly misspelled input | Semantic hit possible | BM25 no help; vectors carry it |

The bottom line: **hybrid is at least as good as vector-only, and it clearly wins exactly where vectors are weakest — exact matching.** The cost is one extra keyword pass, which is negligible at modern retriever speeds.

> The one thing to remember: vectors give "understanding," keywords give "findability," and RAG needs both. Hybrid retrieval pulls a bigger candidate pool on purpose — better to over-retrieve here and let the next stage do the fine selection.

## Reranking: make the ordering smart

Hybrid retrieval solved "**findability**," but the ordering is still crude — vector and BM25 each carry their own biases. Next it goes to a **reranker**, typically a **cross-encoder**:

* The early bi-encoder embeddings encoded the question and each document *separately* — fast, but the two never look at each other.
* A cross-encoder concatenates "question + each candidate" and feeds it to the model, which actually *reads* "does this passage fit the question" — slow, but accurate.

In practice it is a two-stage pipeline:

```
Stage 1 (fast): vector + BM25 hybrid, pull back top 50–100
Stage 2 (accurate): cross-encoder scores each, rerank to top 5–10
```

This is exactly the "lower K, add a reranker" fix `llm-03-rag` hinted at. Stage one is wide (do not miss), stage two is precise (keep the good stuff). Divided labor gives you both speed and quality.

```text
question ──→ vector search (top 50) ──┐
│                                  ├─→ merge (RRF) → top 50~100
└──→ BM25 search (top 50) ─────────┘              │
↓
cross-encoder rerank → top 5~10
│
↓
question + those passages → LLM generation
```

## Practical pitfalls

A few common hybrid mistakes, worth knowing early:

* **Tune only one side**: shrinking K, or adding keywords without keeping vectors, quietly loses the other signal's coverage. Both sides need a big-enough K so the rerank has good material.
* **Forgetting metadata filters**: hybrid is better at *retrieving*; precision still comes from filtering. Filter first, then hybrid, then rerank — a sequence, not a menu.
* **Reranker tradeoff**: a cross-encoder is slower than a bi-encoder, so it must run *after* the pool is small. With a giant candidate pool, reranking becomes the bottleneck.

## When you can skip reranking

Reranking is powerful but it is not mandatory. Whether to add it depends on how good the first stage already is:

| Situation | Need reranking? | Why |
|---|---|---|
| Small corpus, single query type | Often no | Vector + BM25 stage one already looks clean |
| Top-k returns a lot of noise | Yes | One precision pass washes out the junk |
| Very high accuracy bar, costly to be wrong | Yes | It is the best tool to turn "close" into "most correct" |
| Tight latency budget | Trade-off | A cross-encoder is the slowest stage in the whole pipeline |

The deciding question is not "is it available" but "**how much of the stage-one top-k is junk.**" Lots of junk → worth confirming with `rag-04-evaluating` that reranking earns its keep; barely above your tolerance → do not pay the latency for it.

## Next up

At this point the retrieval side has grown from "finds things" to "finds things and orders them well." But how do you *know* it got better? "Feels smoother" is not evidence — you need to **measure retrieval and generation separately**, with a test set you can re-run. That is the most practical article in this series: `rag-04-evaluating`.

#### Q: Why is hybrid retrieval more robust than vector-only search?

* Because hybrid shrinks the candidate pool, making reranking faster

* Because vectors handle meaning and BM25 handles exact literal match; they complement each other, hitting codes and identifiers reliably

* Because BM25 fully replaces vector search

* Because hybrid makes metadata filtering unnecessary

> 💡 Vector-only is strong on semantics but weak on exact matching; BM25 is the reverse. Hybrid keeps both signals (usually merged via RRF) and clearly strengthens exactly the cases vectors miss.
