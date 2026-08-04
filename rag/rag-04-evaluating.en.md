# Evaluating RAG: Turning 'Seems Fine' into Numbers

> 📅 2026-08-04 · Deep Dive
> RAG has two independent things to measure: whether retrieval found the right chunks, and whether generation stayed faithful to sources. Skip retrieval metrics and you cannot tell if a change helped; skip faithfulness and hallucinations hide inside fluency.

---

By now your RAG runs end to end: chunk, embed, hybrid retrieve, rerank, generate — the whole pipeline. Then someone asks: "**How good is it?**" And all you can say is, uh, seems fine.

"Seems fine" is not a metric, and it will not tell you what to change next. The most expensive part of a RAG system is not compute — it is shipping a new version and having no idea whether it got better or worse. This article shows you how to trade "feeling" for "numbers" — split into two independent things that must be measured separately.

## Two problems, measure them apart

The quality of a RAG answer is the product of two stacked failures. If you do not separate them, you will never know which stage to blame:

```
retrieval: the database has the answer -> did we pull it back?   <- retrieval quality
generation: the retrieved passage -> did the model cite it faithfully?  <- generation quality
```

If retrieval is perfect but the model refuses to follow it, the answer is still fabricated. If the model is faithful but retrieval grabbed the wrong passage, the answer is wrong too. So you need two sets of metrics.

## Retrieval quality: recall and precision

The two most useful numbers for retrieval come from classic information retrieval:

| Metric | It asks | Roughly computed as |
|---|---|---|
| **Recall** | Of what should have been found, how much did we find | relevant chunks hit ÷ all relevant chunks |
| **Precision** | Of what came back, how much was actually relevant | relevant chunks hit ÷ all chunks returned |

In the words of `rag-02-chunking`: low Recall = **the answer was not carved out or not hit** (chunking/retrieval's fault); low Precision = **a pile of noise came back** (filtering, reranking, or K's fault). Which number drops tells you which stage to attack.

## Generation quality: faithfulness

Once retrieval is right, measure "**did the model stay true to the source?**" That concept is **faithfulness**:

* **High faithfulness**: every claim in the answer is supported by something in the retrieved passages.
* **Low faithfulness**: the model writes fluently but mixes in its own "memory" — the `llmcore-05-hallucination` scenario, with the source now being "what it assumes it knows."

A practical method: for each answer, have a human (or a grading model) split the answer into claim-by-claim statements and check each one against the context. The fraction of statements that are supported is your faithfulness score.

> The one thing to remember: retrieval is measured by recall/precision ("can we find it"), generation by faithfulness ("does it stay honest"). The two multiply, they do not add — either one being zero makes the whole answer wrong. Fix the weaker one first, not both evenly.

```Conceptual:&#x20;compute&#x20;recall&#x20;and&#x20;faithfulness&#x20;over&#x20;an&#x20;eval&#x20;set
# Conceptual; use the API of whatever library version you choose

# With only 5 questions, one table is enough:
evals = [
  {"qid": 1, "relevant": ["c3", "c4"], "retrieved": ["c3"],        "claims_grounded": 2, "claims_total": 2},
  {"qid": 2, "relevant": ["c7"],        "retrieved": ["c2", "c7"],  "claims_grounded": 1, "claims_total": 3},
  {"qid": 3, "relevant": ["c1"],        "retrieved": [],            "claims_grounded": 0, "claims_total": 0},
  {"qid": 4, "relevant": ["c5"],        "retrieved": ["c5", "c9"],  "claims_grounded": 2, "claims_total": 2},
  {"qid": 5, "relevant": ["c8"],        "retrieved": ["c3"],        "claims_grounded": 1, "claims_total": 1},
]

def recall(e):    return len(set(e["relevant"]) & set(e["retrieved"])) / len(e["relevant"])
def precision(e): return len(set(e["relevant"]) & set(e["retrieved"])) / max(1, len(e["retrieved"]))

print("avg recall   =", sum(recall(e)    for e in evals) / len(evals))
print("avg precision=", sum(precision(e) for e in evals) / len(evals))
print("faithfulness =", sum(e["claims_grounded"] for e in evals) / max(1, sum(e["claims_total"] for e in evals)))
```

Those three lines of output are your RAG "dashboard." The small number of questions is fine — the numbers tell you whether the bottleneck is retrieval or generation: low recall means "could not find it," low faithfulness means "it is making things up."

## Build a small evaluation set

You do not need a thousand questions to start. Begin with a small, **stable** set you can re-run, and that is enough. The principle, in four steps:

#### Collect real questions

Pull 30–50 of the questions users actually ask from logs. No logs? Have a business team "role-play a user" and write some. Real distribution beats questions you invented.&#x20;

#### Label ground-truth passages

For each question, a human marks which passage in the document holds the correct answer. This label is the denominator for recall — without it, recall cannot be computed.

#### Write a "golden answer"

No need for verbatim prose — a list of the points a complete answer should cover. It becomes the baseline for faithfulness and answer quality.

#### Freeze it and re-run

Store the set as a test. Every time you change chunking, retrieval, or the prompt, run it and record the scores. Make improvement decisions from here.

Thirty to fifty questions looks small, but "few and stable, can re-run" beats "many and messy, run once and forgotten" every time. The value of an eval set is **reproducibility**, not volume.

## Offline vs. online: evaluation comes in two kinds

What we built above is "**offline evaluation**" — score a fixed set before deploy, to make decisions. Production also needs "**online monitoring**" — watch whether quality drifts during real use:

| Dimension | Offline evaluation | Online monitoring |
|---|---|---|
| When | Before and after a change | Continuously after launch |
| Data | Fixed test set | Real user traffic |
| Measures | Recall / Precision / faithfulness | Failure rate, fallback rate, satisfaction proxies |
| Question it answers | "Can this version ship?" | "Did it get worse after shipping?" |

Practical advice: **get the offline set first, then talk about online monitoring.** Without an offline set, online failures give you no way to localize which stage's change caused them. Data and query distributions drift — re-sample a fresh test set every quarter, and do not let your numbers sit at half a year old.

## Common failure modes

Once you start measuring, you will keep hitting the same patterns. Recognize them and debugging gets fast:

| Symptom | Root cause | Fix |
|---|---|---|
| Low retrieval recall; answer in store but not found | Answer cut at a chunk boundary, or semantically off the beaten path | Add overlap, cut along structure, contextual chunk |
| Returns a pile of noise (low precision) | Too little filtering, K too large, no rerank | Metadata filter, tune K, add a reranker (see `rag-03-hybrid`) |
| Retrieved chunk relevant but holds no answer | Got the "neighbor" not the exact passage | Check chunk boundaries, raise recall, confirm the answer is carve-able |
| Retrieval fine, generation fabricates | Prompt does not force citation, or model assumes it knows | "Use only the context" + allow "I do not know," lower temperature |
| Keeps missing the same class of question | Data itself is missing a section, or a source is stale | Fix the data, not the model |

Notice the last column: **RAG fails honestly — where it goes wrong usually names the responsible stage.** Retrieval-type failures and generation-type failures need completely different fixes, so the first move is always to localize.

## Close the loop on improvement

With these numbers, RAG improvement shifts from "inspiration-driven" to "experiment-driven":

```
change (chunking / retrieval / prompt) -> re-run eval set -> watch recall, precision, faithfulness
         ^                                                          |
         └──────────────── keep what improved, roll back what regressed ──┘
```

Change one thing at a time, compare on the same set, and you own the dashboard of your RAG system. This matters so much in production that it deserves its own treatment — and it is the only objective basis for `finetune-01-finetune-vs-rag` deciding "fine-tune or fix the RAG": the numbers tell you where the bottleneck is, then you decide where the money goes.

## The whole series in review

You now hold the complete RAG playbook: the **why** (`rag-01-what-is-rag`), the **chunking** (`rag-02-chunking`), the **retrieval that finds things** (`rag-03-hybrid`), and **knowing whether it is good** (this one). Next, when the RAG numbers hit target but you wish the model itself sounded more like your company, that is the door into the fine-tuning world.

#### Q: Retrieval recall keeps coming in low, but faithfulness is great. What is the most likely problem?

* The model refuses to answer from context and needs a new prompt

* The answer is in the data but not being pulled back — typically a chunking or retrieval issue, like an answer sliced at a boundary or an off-path phrase

* The reranker pushed the correct chunk too far down

* No fix needed; low recall does not affect usage

> 💡 Recall measures 'did we find what we should have'; faithfulness measures 'did the model cite what it found.' Low recall with high faithfulness means retrieval is dropping the right passages — a retrieval-side issue, not a generation-side one.
