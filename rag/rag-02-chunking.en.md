# Chunking & Indexing: Turning Documents into Grab-able Units

> 📅 2026-08-04 · Getting Started
> RAG retrieves chunks, not whole documents. How big, whether to overlap, whether you keep headings and metadata — these decide whether retrieval actually finds the key answer.

---

The first step of RAG is chunking — splitting a document into small units that can be "picked up." It is the least flashy and yet the most decisive stage: **retrieval can only pick up the chunks you created.** Get this wrong and every later stage (reranking, evaluation) is just patching up a problem you could have avoided up front.

This article gives you the chunking habits that avoid the traps, how to embed and index afterward, and why "store only the vectors" is the mistake most beginners make.

## Why chunk at all

You cannot make "the whole 200-page manual" a single retrievable unit, for two reasons:

1. **Semantics get diluted.** A manual covers too many topics. Squashing it into one vector mixes "dessert recipes" and "refund policy" into the same coordinate, so neither searches well.
2. **The grain is too coarse.** Even if it is hit, you would have to push the entire 200 pages into the prompt — the window blows up instantly, and the "feed only what is needed" idea from `llm-03-rag` collapses.

So you split the document so that **each chunk carries about one answerable unit** — usually one self-contained topic or answer.

```
whole manual ──split──→ [chunk① onboarding][chunk② vacation rules][chunk③ salary bands]
                            ↑ retrieval picks these, not the whole book
```

> The one thing to remember: the chunk is the smallest unit of retrieval — you cannot retrieve an answer you did not carve out. If an answer straddles two chunks with no overlap, retrieval will likely catch neither one in full.

## Chunk size: the tradeoff

There is no single right size, but the direction is clear — too small and too big both cost you:

| Size | Pros | Cost |
|---|---|---|
| **Too small** (fewer than 100 tokens) | Precise, low noise | Too little context; semantics incomplete; loses context |
| **Mid-range** (300–800) | Balanced precision and completeness | Needs real querying to tune |
| **Too large** (>1500) | Complete context | Diluted meaning; drags in irrelevant content; blows up the window |

A good starting point is 300–800 tokens (roughly 1–3 paragraphs), then **tune it with real queries — do not guess.** The test is "is it enough to answer one question independently" — if it misses the answer it is too small; if it drags in a pile of unrelated content it is too big.

## Overlap: don't slice the answer at the boundary

The classic chunking accident: a key sentence sits exactly on the cut, its first half in chunk A and its second half in chunk B. Retrieval returns both as "relevant but unable to answer."

The fix is to let neighboring chunks **overlap** a small slice, typically 10–20%:

```
chunk① […..A paragraph…...| overlap head]
chunk②              [overlap tail| B paragraph…..]
          ↑ a sentence that fell on the boundary stays complete on both sides
```

Overlap is not "the more the better" — too much means redundant storage and wasted tokens. 10–20% is usually enough for a sentence cut at the boundary to appear intact on both sides.

## Cut along structure, not just character count

Slicing every N characters is the simplest and the best way to produce half-sentences. The better approach is to **follow the document's natural structure**: split by heading → section → paragraph first, then subdivide only as needed.

The key is **keeping structural information**. Two habits pay off especially well:

* **Including the heading**: put the section title into the chunk's content, so retrieval can judge topic by heading.
* **Contextual chunk**: when embedding, prepend "which chapter this belongs to and what came before" — retrieval quality often jumps noticeably. `llm-03-rag` mentioned this trick.

## Pick your strategy by content type

Not every document chunks the same way. Different content has its own "natural smallest unit":

| Content | Good approach | Why |
|---|---|---|
| Continuous prose (article, manual section) | 300–800 tokens, by paragraph and heading | A paragraph is often one answerable unit |
| Code | Cut by function / class / module, keep indentation | Half a function is meaningless; the unit is "one runnable piece of logic" |
| Tables | One row or a small block per unit, keep the header | The header is the semantic "anchor" for that row — do not drop it |
| Page-numbered PDFs | Extract text by layout first, then cut by page/section | Layout shreds text; cutting naively gives broken half-sentences |

The test stays the same: **"can this chunk answer one question on its own?"** For code that is "can this logic be understood in isolation," for tables "does this cell travel with its meaning."

## A rough recipe: chunk broadly, calibrate with real queries

There is no formula, but there is an order you can follow:

```
① cut along document structure (heading -> section -> paragraph)
② if a chunk is still too big, subdivide inside the paragraph by meaning
③ add 10-20% overlap so answers never sit clamped at a boundary
④ accept with your 10 hardest real queries: is each one fully carried by some chunk?
```

Step ④ is where the value is. Grab a few of the questions users are most likely to ask and most likely to get wrong, and confirm "which chunk fully carries this answer." The moment one answer is "sliced to pieces," that is the signal to revisit your chunking — and the verdict always lands on the numbers from `rag-04-evaluating`.

## Embedding + the vector index

The next step is to make each chunk semantically searchable — that is exactly the embedding from `llm-02-embeddings`:

* Convert each chunk with an embedding model into one vector.
* Store all vectors in a **vector index** (FAISS, Chroma, Qdrant, pgvector, and so on).
* At query time, embed the question and find its K nearest neighbors in the index (common top-k = 3–10).

I will not re-explain embeddings here, but one reminder matters: **chunk size must match the embedding model's sweet spot.** Most embedding models degrade on very long passages, which is another reason to keep chunks on the smaller side. For how to choose a vector store and when you even need one, see `vectordb-01-what-is`.

## Don't store only vectors: metadata

The most common beginner mistake: write the embedding to the database and call it done, keeping only the vector and the raw text. That lets you *retrieve content* but not *filter it* or *explain its source*.

Every chunk should carry a set of metadata, at minimum:

| Metadata | Use |
|---|---|
| `doc_id` | Which document this chunk came from |
| `source` / `url` | Usable as a citation |
| `heading` / `page` | For locating and filtering |
| `updated_at` | To judge freshness |
| `tags` / `department` | Business-level filters |

Filtering by metadata *before* retrieval ("only 2026","only the compliance team") cuts noise and raises precision dramatically — **filtering is cheaper and more effective than ranking.**

```Conceptual:&#x20;chunk&#x20;+&#x20;store&#x20;vector&#x20;+&#x20;store&#x20;metadata
# Conceptual; use the API of whatever library version you choose
from sentence_transformers import SentenceTransformer
import chromadb

embedder = SentenceTransformer("BAAI/bge-small-en-v1.5")
client = chromadb.PersistentClient(path="./chroma")
collection = client.get_or_create_collection("handbook")

def chunk_with_overlap(text, size=400, overlap=80):
  chunks, start = [], 0
  while start < len(text):
      chunks.append(text[start : start + size])
      start += size - overlap          # 20% overlap
  return chunks

doc = load_manual()                      # your document
for i, chunk in enumerate(chunk_with_overlap(doc)):
  vec = embedder.encode([chunk]).tolist()
  collection.add(
      ids=[f"doc1:{i}"],
      embeddings=vec,
      documents=[chunk],
      # storing only vectors is a regret -> always carry metadata
      metadatas=[{"doc_id": "doc1", "section": f"page-{i}", "updated": "2026-08-01"}],
  )
```

## Chunk quality still has to be weighed

There is no "correct at a glance" checklist for chunking. The real judge is the retrieval result — and retrieval results only count once they become **numbers** (recall, precision). Without measurement you are just guessing at the size. By the time we reach `rag-04-evaluating`, you can turn "how good is my chunking" into a number you can compare.

Next up, we upgrade retrieval from vector-only to **hybrid** — solving the single blind spot vector search struggles with: `rag-03-hybrid`.

#### Q: Why is it said that 'you cannot retrieve an answer you did not carve out'?

* Because retrieval only searches the existing chunks; if an answer is split across two with no overlap, both are relevant but incomplete

* Because the embedding model automatically fills in the uncut parts

* Because the vector index can surface omitted text

* Because the model fills missing chunks from its own memory

> 💡 The chunk is the smallest retrieval unit. If the answer straddles two chunks with no overlap, both hold only half the answer, so retrieval usually cannot return one that answers independently.
