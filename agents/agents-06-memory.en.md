# Agent Memory: Working Memory vs Long-Term Memory

> 📅 2026-08-04 · Core Mechanics
> How does an agent remember what it did? From the working memory of the context window to external stores and RAG-backed long-term memory — a complete look at memory layers.

---

## The scenario: your agent repeats the same mistake

The agent you built in `agents-04-first-agent`, when asked "do I need an umbrella tomorrow?", checks the weather, jots a note, and answers nicely. Close the program and reopen it, and it has forgotten everything — even the note you told it about not scheduling outdoors in the rain.

That is not a bug; that is a **memory design** problem. Humans distinguish working memory from long-term memory, and agents do too. Understand these two layers and your agent stops living forever in "today."

***

## Two layers of memory, two homes

| Aspect | Working memory (short-term) | Long-term memory (persistent) |
|--------|---------------------------|-------------------------------|
| Lives where | The model's context window | External store: database, files, vector store |
| Lifespan | Gone when the conversation ends | Persists across conversations and time |
| How it is read | Carried whole on every request | Retrieved on demand, then put into context |
| Capacity | Bounded by the window limit | Theoretically unlimited |
| Cost | Paid per token | Paid only when retrieved |

> One sentence to remember: working memory is "what you can hold in mind now", long-term memory is "what you can find later". All of an agent's thinking happens in working memory; long-term memory is just its external hard drive.

The physical ceiling of working memory is the context window — as `llmcore-03-context` explains, a model can only "see" so many tokens at once. That means everything an agent thinks about must first fit into this window; anything outside it might as well not exist for the model.

***

## Why you cannot cram all memory into context

The obvious approach is to dump the entire history into the window. Three reasons it fails:

### 1. Cost: tokens are expensive

If every conversation carries 100k characters of history, every call pays for those 100k — even when most of it is never used. Longer conversations, higher cost.

### 2. Diluted attention

Another point from `llmcore-03-context`: the longer the context, the thinner the model's attention on any single fact. Stuff it with noise and real information gets missed.

### 3. Repeated round trips

Remembering one thing long-term means re-reading "everything" every time. Often all you need is a single sentence, yet the whole life story gets retransmitted.

So the goal of memory design is: **keep what is most needed in working memory, store the rest externally, and retrieve precisely when required.**

***

## Four ways to implement long-term memory

What does external storage look like? Ordered from least to most structured:

### 1. Session history archive

Store the `messages` as JSON and reload them next time — the approach in `agents-04-first-agent`. Simplest, but only suits short conversations.

### 2. Note-style memory

Write important facts as notes, e.g. "the user prefers short answers." Use a `save_note` tool to write and a `search_notes` tool to read. This turns memory into a tool, echoing `agents-05-tools-and-actions`.

### 3. Summarized memory

Every N turns, compress the old conversation into a summary and continue with only the summary. Great for saving tokens, but detail gets lost.

### 4. RAG memory

Split facts into chunks, embed them, store them in a vector database, and on demand retrieve the most relevant slices by semantic search. This is memory combined with retrieval — see `rag-01-what-is-rag`.

***

## Two strategies for reading and writing memory

Long-term memory is not just "where to store" but "how to read and write." Two dominant strategies:

### In-context: write memory into the prompt

Put all relevant history directly into the context. Simple and direct, but bounded by the window, and everything gets carried along each time.

### RAG-backed: retrieve on demand

At query time, first use embeddings to retrieve the most relevant slices, then place them into context. Large capacity, cheap on tokens, but adds a retrieval step and depends on retrieval quality.

| Strategy | Good for | Bad for |
|----------|----------|---------|
| In-context | Short chats, small memory | Long-term, large memory |
| RAG-backed | Long-term, large knowledge | When recall must be exact |

> The two are not mutually exclusive: mature agents often use both layers — recent conversation goes in-context, old knowledge goes through RAG. Keep what is fresh visible and what is old findable.

***

## The full journey of a memory

From "something happens" to "recalled later," a memory passes through four stages:

```text
event happens → write (store externally) → index (how to find) → retrieve (back into context)
↑______________________________________ next time it is needed ____________↓
```

Each stage can go wrong:

* **Write failure**: never stored, so it never happened.
* **Index failure**: stored but unfindable, so it might as well not exist.
* **Retrieve failure**: found but irrelevant, so it becomes noise.
* **Staleness**: retrieving outdated information is more dangerous than having none.

***

## Three hands-on principles for memory design

### 1. Choose what you write

Not everything is worth remembering. When writing a note, ask: "will I need this next time?" If not, do not write it — a cluttered memory is harder to search.

### 2. Be deliberate about retrieval

Attach **timestamps** and **sources** to what you retrieve, so the model can judge freshness and trustworthiness. A 2026 memory must not masquerade as a fact from today.

### 3. Remember to forget

Long-term memory needs cleanup: delete stale facts, merge duplicates. An agent that forgets is more reliable than one that remembers everything.

***

## Next stop

Memory lets an agent know what it did, but the next question is: **does it know what to do first?** The next article, `agents-07-planning`, covers planning — decomposing a big task into steps, deciding the order, and when to spin up sub-agents for parallel work. However good the memory, without a plan the agent is just spinning in place.

***

## Key takeaways

* Working memory = the context window, gone when the conversation ends; long-term memory = external storage, persists across conversations.
* Cramming context has three costs: expensive, diluted attention, repeated round trips.
* Four long-term implementations: history archive, notes, summaries, RAG.
* In-context suits small memory, RAG-suited large knowledge; the two can coexist.
* All four memory stages can fail: write, index, retrieve, freshness.
* Three principles: choose what you write, timestamp what you retrieve, remember to forget.

#### Q: Why can you not just cram all history into the context window?

* Because the model cannot read long text

* Because token cost is high, attention gets diluted, and the whole thing is retransmitted every time

* Because external storage is faster

* Because long conversations always contain syntax errors

> 💡 The context window is finite and expensive: carrying all history makes every call costly, overlong context dilutes attention, and most history is resent each time — so memory needs to be layered.
