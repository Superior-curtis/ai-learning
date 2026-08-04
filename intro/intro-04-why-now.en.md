# Why Now: The Truth About the 2020s AI Wave

> 📅 2026-08-04 · Core Concepts
> AI has been “about to change the world” for seventy years — so why did the last five actually deliver? Not one miracle, but five things colliding at once.

---

"AI is about to change the world" — people have said it since the 1950s. Across seventy years it was said many times and mostly came to nothing. So why is **this** one real?

The most honest answer: **no single miracle appeared — five things started rolling each other up in the same window of time.** Taken alone, each one is "not enough." Taken together, they became the predictable curve in `train-05-scaling-laws`. This post unpacks the five and explains why "now" is special.

## The one-line version

> The 2020s AI wave = compute, data, algorithm, capital, and interface — five things multiplying each other within the same decade. Not one sudden "revolution," but the whole set arriving at once.

## 1. Compute: GPUs made "bigger" affordable

To make a model stronger, the most direct lever is **compute** — more GPUs to train a bigger model. And this two-decade GPU curve is steeper than Moore's law.

* 2012's AlexNet stunned the world with two GPUs; today's frontier training runs on the order of tens of thousands of GPUs.
* `train-05-scaling-laws` shows that scaling parameters, data, and compute together moves capability steadily up a predictable curve.
* That curve became "the math behind burning cash" precisely because it is **predictable** — a company knows how much stronger a model gets if it spends twice as much.

Even more decisive is the cost line: **the unit cost of a single inference (one answer) has fallen almost exponentially this decade.** Five years ago using a language model felt like a luxury; today it is cheap enough to barely notice. Falling cost is the physical precondition for "the number of people who can afford it exploding" (`econ-01-inference-cost`).

Compute is the engine, but not everything. A great engine with no fuel just idles — and the fuel is data.

## 2. Data: the internet became the training set

The model's entire "knowledge" comes from the text it read. And from 1990 to 2020, humanity digitized and posted online, at vanishingly low cost, what amounts to centuries of accumulated wisdom — an unprecedented free gathering of training material.

* Wikipedia, paper archives, open-source code, books, forums — all became training material (see `train-04-corpus`).
* Without this sea, even the scaling curve has nothing to chew on.
* But note: **the sea is finite.** Near the end of `train-05-scaling-laws` comes the warning that quality human data is being consumed — which is exactly the cost and bottleneck the `economics` series takes on.

## 3. The algorithm: Transformers made "bigger" actually "better"

With an engine and fuel, you still need a correctly designed engine. The 2017 Transformer is that engine — naturally built for parallel compute and naturally built for "guess the next word" (`llmcore-01-next-token`).

Its key contribution: **welding "scale" to "language ability."** Before it, adding more GPUs did not necessarily help; after it, "bigger = better at talking" became a replicable engineering path. In 2020, GPT-3 proved it in action — push scale to 175 billion parameters and language ability jumps with it.

Without that algorithmic breakthrough, no amount of compute and data could have moved today's curve. This is also the most overlooked part: **money and data alone, with the wrong algorithm, still stall** — most of the past seventy years of failure lost right here.

## 4. Capital: the flywheel started turning

Once the technology was ready, capital accelerated it — on a scale no earlier "spring" ever had:

| Capital phenomenon | Why it matters |
|---|---|
| Hundred-billion-dollar training budgets | Big players can "pre-buy the future" along the scaling curve |
| Cloud compute renting GPUs on demand | Startups need not build their own data centers |
| Falling inference cost | Drastically more businesses can afford to use AI (`econ-01-inference-cost`) |

Capital, compute, and product form a flywheel: **better model → more users → more revenue and data → more capital → bigger model.** This is the other side of "how expensive GPUs are, and why so few can play" — the detail lives in `econ-02-gpu-economics`.

```text
capital → compute → bigger model → better product → more users
↑                                                  │
└──────────── revenue & data flow back as capital ────┘
This flywheel is the economic core of the current spring
```

## 5. Interface: the chat box made it usable by everyone

The last piece, which many overlook, is **interface.**

* Models before 2022 were already strong, but hidden inside APIs and papers — reachable only by engineers.
* ChatGPT dropped the model into a **chat box** — no code to learn, type and it works.
* A hundred million users in two months. The technical barrier suddenly fell to zero, and the demand side swung wide open.

The technology was mature and the capital was in place — but "the whole world can use it" was the gap that interface's small step closed. **The algorithm decides "whether it can"; the interface decides "who can use it."**

## Before 2020 vs. now: a comparison

Stack the state of the five things together and the difference is obvious:

#### 對照 / Comparison

## Why "now," and not a decade ago

Put the five together and the answer is clear — **not any single one, but the timing**:

* **The 2010s**: deep learning existed, but the Transformer had not been born, and the scaling curve had not been welded to language.
* **Five years ago**: models were getting stronger, but costs were high, the interface was brittle, and capital had not poured in.
* **Now**: compute, data, algorithm, capital, interface — five cards finally in hand at once.

That also explains why so many past promises came to nothing: it was "money with no engine" or "engine with no fuel." Now is the first time the whole system has arrived together.

## Sober headwinds: this wave is not infinite

"All five cards in hand" does not mean "no problems from here." Three headwinds are pressing down on the wave:

| Headwind | What it is | Related reading |
|---|---|---|
| **Data drying up** | Quality human data is being consumed; growth may slow | `train-05-scaling-laws` |
| **Cost hitting a wall** | GPU scarcity and expensive energy leave small players behind | `econ-02-gpu-economics` |
| **Regulation arriving** | Governments are legislating and restricting, which may squeeze business | `policy-01-eu-ai-act` |

Understand the headwinds and you will not mistake "now" for "forever." It is more like a **genuine compounding growth** than a guaranteed shortcut.

## How to judge whether an "AI claim" is real

Every day you will see headlines: "another model breakthrough." Run each one through three gates to quickly decide if it is worth getting excited:

#### Gate one: capability or packaging?

Is the model itself genuinely stronger (new architecture, larger scale, new data) — or is it just “a reskin of the interface” or marketing? Only the former is worth watching.

#### Gate two: is there quantitative evidence?

Are there benchmark scores, reproducible experiments? Or only words like “stunning the world”? Numbers earn points first.

#### Gate three: place it back in the five things

Which card does this breakthrough move? Moving one card is incremental; only several resonating at once can be “era-defining”.

## The one-liner for a friend

If a friend asks you "why did AI suddenly get so popular," this one line is enough: "AI did not suddenly get smart — **compute, data, algorithm, capital, and interface** all pushed each other in the same decade and happened to line up." Then add a second line: "And the scaling laws say this is predictable, not luck." (`train-05-scaling-laws`)

If they press "so will it keep going like this," throw the "sober headwinds" section back at them: data runs out, cost hits walls, regulation arrives. **Ups and downs are the cycle; the structure is what is new.**

## What this means for you

* **Stop asking "is AI hype?"** On a seventy-year scale it has been overhyped many times; on a five-year scale the capability growth is real. Both are true.
* **To judge an AI company, ask where it sits among the five** — does it sell compute, models, tooling, or products built on models? The industry map is in `industry-01-value-chain`.
* **To pick a model or a product, check which generation the architecture is and how big it eats** — the model-family tour is `models-02-families-tour`.
* **To put AI to work in your own job, start with RAG and Agents** — the former makes it tell the truth, the latter makes it take action (`rag-01-what-is-rag`, `agents-01-what-is-agent`).

Hold the "five things" in your head and you are equipped to ask the next question: is this "now" wave actually "general intelligence" drawing closer? That is the battlefield of `intro-05-agi`.

> Let's be sober: "now" is not a miracle — it is predictable compounding growth. It can cool down and hit bottlenecks (data running out, cost hitting a wall, regulation arriving). But regardless of ups and downs, the structure — five things amplifying each other — has been built.

One last reminder: the wave continues, but **"current progress" is not "AGI progress."** Whether this wave can genuinely become "general intelligence" is exactly the fight that `intro-05-agi` picks.

#### Q: Why did the 2012 deep-learning breakthrough not immediately spark a mass wave, waiting until 2022 instead?

* Because compute was insufficient in 2012 — GPUs were only invented in 2022

* Because deep learning in 2012 was an academic event; it took the Transformer welding scale to language, plus capital and the chat interface, to become a mass event

* Because AI research completely stalled between 2012 and 2022

* Because ChatGPT used a brand-new algorithm unrelated to deep learning

> 💡 AlexNet in 2012 ignited deep learning, but only inside academia. It took the 2017 Transformer, the scaling laws landing, capital flowing in, and the 2022 chat interface — all five arriving — to go from an academic event to a mass event.
