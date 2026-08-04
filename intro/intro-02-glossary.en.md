# The Plain-English AI Glossary

> 📅 2026-08-04 · Getting Started
> Tokens, RAG, AGI, hallucination, fine-tuning — every article is packed with jargon. This post explains all the common terms in plain language, and points you to where each is covered in depth.

---

You open an AI article and the first thing you see is a wall of "token, RAG, AGI, hallucination" — if the acronyms feel overwhelming, that is not on you. The field is just addicted to jargon.

The good news: there are really only a dozen or so common terms, and each one maps to an article in this book that explains it properly. This post is that lookup table — a plain-language one-liner for each, plus where to go read the long version.

## First, the foundation

The whole book starts from one sentence: **an LLM does one thing — "guess the next word."** After reading enormous amounts of text, it learns "given the words so far, which word is most likely next." (To see why that short sentence can produce something that writes essays and code, read `llmcore-01-next-token`.)

This "guessing" is the **final piece** behind almost every term below. Hold onto that one sentence, and the rest is just elaboration.

## The "three-letter" words people mix up

`intro-01` already untangled these with a layer cake; here is the one-line essence. These five terms nest inside each other — they are not synonyms:

| Term | One-liner | In depth |
|---|---|---|
| **AI** (Artificial Intelligence) | The umbrella for getting machines to do things that need human brains | `intro-01-what-is-ai` |
| **ML** (Machine Learning) | One approach: instead of hard-coding rules, learn patterns from data | `intro-01-what-is-ai` |
| **DL** (Deep Learning) | A popular technique: learn with many-layered neural networks | `intro-01-what-is-ai` |
| **LLM** (Large Language Model) | Today’s breakout model type: specialized in language, runs on "next-word prediction" | `llmcore-01-next-token` |
| **AGI** (Artificial General Intelligence) | The hoped-for "can do anything" AI — not achieved yet | `intro-05-agi` |

> The key is containment, not equality: every LLM is DL, every DL is ML, every ML is AI — but not the reverse. AGI is a future aspiration; no current system qualifies.

The most practical way to sort these five: **"AI / ML / DL" usually points at a discipline or technique, "LLM" points at the concrete product star, and "AGI" points at a not-yet-realized vision.** Break it down that way and the acronyms in headlines stop being scary.

## The three words you meet the moment you use a product

The first things you hit in ChatGPT or Claude are these three. They decide how big, how expensive, and how trustworthy the thing can be.

| Term | One-liner |
|---|---|
| **Token** | The model does not read letters or words — it reads little chunks of text; tokens set the memory ceiling, the price, and the speed |
| **Context window** | The maximum number of tokens the model can "see at once"; anything outside the window is, to it, unseen |
| **Hallucination** | The model confidently makes things up — not because it lies, but because it learned "which word flows", never "which fact is true" |

These three are the roots of the field’s famous pitfalls. How pricier tokens are (especially in Chinese) is in `llmcore-02-tokens`; why the window equals the memory ceiling is in `llmcore-03-context`; and hallucination is the #1 thing the whole industry defends against — see `llmcore-05-hallucination`.

Flipped around, these three are "recognizable and manageable" rather than scary:

| Symptom you hit | Which term | First move |
|---|---|---|
| It seems to forget what you said three hours ago | Context window | Paste the key facts back in; do not expect it to remember |
| The bill is way higher than expected (especially with lots of Chinese) | Token | Trim the input, work in chunks |
| It gives a precise yet suspicious number or source | Hallucination | Cross-check; do not treat its tone as evidence |

#### Want to dig deeper? Expand these

“How are tokens actually split?” Common English words are usually one token each, rare words get broken into several pieces; in Chinese, each character is often 1+ tokens, so the same sentence often costs 2–3x more in Chinese than English.\n\n“What is the real difference between an 8K and 200K window?” 8K fits only a few pages of text; 200K can hold a few-hundred-page book. A bigger window is more useful — and more expensive.\n\n“Can hallucination really be prevented?” Not completely eliminated, but you can give the model evidence via RAG and ask it to cite sources, pushing the rate down to a usable level.

## The four words that make AI "useful"

A base model can only continue text. These four are what actually put it to work on your data, your tools, and your workflows:

| Term | One-liner | In depth |
|---|---|---|
| **Fine-tuning** | Take a pre-trained model and keep training it on *your* data, to teach it a specialty | `train-02-finetuning` |
| **RAG** (retrieval-augmented generation) | Before answering, pull the relevant passages from your database into the prompt, so the model answers from evidence | `rag-01-what-is-rag` |
| **Agent** | A system that does not just answer — it works step by step to finish a task on its own | `agents-01-what-is-agent` |
| **MCP** (Model Context Protocol) | An open standard that unifies "how an agent/model connects to your tools" — the "USB of the tool world" | `agents-03-mcp` |

If your goal is simply "make the model answer from my knowledge", start with RAG — it is the cheapest, easiest, and fastest way to stop frozen knowledge and hallucinations. Hold off on fine-tuning until you confirm a real need to change behavior or format.

#### 對照 / Comparison

A one-line memory: **RAG is "let it read the book, then quiz it"; fine-tuning is "train it into a habit."** The full decision on choosing between them is in `finetune-01-finetune-vs-rag`.

## When you see an unfamiliar term, drop it into three boxes

Next time you meet an acronym you have never seen, do not panic. Run it through three questions:

1. **Is it "model behavior" or "engineering practice"?** If it describes the model itself (token, window, hallucination, reasoning), look in `llmcore-*`. If it tells you how to do something (fine-tuning, RAG, Agent, MCP, quantization, prompting), look in `train-*`, `rag-*`, `agents-*`, `prompt-*`.
2. **Does it change "the model" or "the input"?** Fine-tuning changes model weights; RAG changes the input content. One question and they split cleanly.
3. **Is it "now" or "vision"?** The ones you wait for (AGI, alignment) form one class; the ones live today form another. The former live in `intro-05-agi` and the `align-*` series; the latter are the first half of this book.

Master this classification and it beats memorizing a hundred terms.

## When the acronyms show up at work

Jargon is not for exams — it is for meetings, headlines, and docs. Translate it to plain language and the scene clicks:

| A colleague / headline says | What it means |
|---|---|
| "We added RAG to this version, so it hallucinates less" | It pulls data first and answers from it; fewer fabrications |
| "Start a new session — the window is full" | The conversation got longer than the memory ceiling, so start over |
| "This model is not fine-tuned" | It was never trained on your data; it is the general-purpose version |
| "Let us build an agent to run this workflow" | Have the AI do the multi-step process by itself |
| "We connect it to our systems via MCP" | A standard protocol lets the AI operate your tools |

## Three words people most often get wrong

Let us close with three everyday misuses, because they cause the most confusion in conversation:

| Often misused | The correct take |
|---|---|
| Calling an "LLM" just "AI" | An LLM is only one kind of AI; do not mistake the part for the whole |
| Calling "hallucination" a "lie" | Hallucination has no intent — it is the inevitable result of "not fact-checking" |
| Calling "RAG" a "memory" | RAG fetches fresh material each time; it does not "remember your things" |

All three misuses share one habit: **treating the part as the whole.** Ask yourself "is this just one technique / one model?" and you will get it right half the time.

## Three lines to remember the whole table

If you do not want to study the table, these three lines are enough to hold a conversation:

```The&#x20;3-minute&#x20;cheat&#x20;sheet
One sentence: an LLM is a "guess the next word" machine.
Three pitfalls: sees little (window), reads chunks (tokens), makes things up (hallucination).
Four ways to make it useful: fine-tune changes behavior, RAG provides evidence, agents act, MCP connects tools.
```

> The one thing worth holding onto: these terms are not unrelated magic — they all orbit the same core — "predict the next word" — wrapped in two layers of data and engineering. Keep the map, then go deep term by term, and you will not get lost.

## Next

A glossary exists so you can go read the long articles. The next stop is how the whole thing evolved to where it is — `intro-03-history` (the short history of AI, 1950 → now).

#### Q: Why is “getting all the tokens, RAG, and Agents right” not the path to AGI?

* Because AGI is just another name for doing many narrow tasks at once

* Because tokens, RAG, and Agents are engineering tools, while AGI is the vision of general intelligence — and every today system is only “narrow”

* Because AGI does not exist and the word was invented

* Because tokens, RAG, and Agents must first be merged into one thing

> 💡 Tokens, RAG, and Agents (plus fine-tuning and MCP) are all engineering techniques that make current narrow AI more useful; AGI is the aspiration of general, do-anything intelligence. Stacking techniques well has not yet turned “narrow” into “general” — exactly the debate that intro-05 takes on.
