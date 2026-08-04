# The Player Map

> 📅 2026-08-04 · Core Concepts
> Chips, clouds, labs, open weights — at every layer a handful of names trade blows. See who is on the same side and who is a direct rival, and the AI politics come into focus.

---

The last article split the industry into a value chain; this one puts names on it. Good news: **at every layer the list of players is short enough to memorize.** The chip layer is nearly a one-company show, the cloud layer is three giants, and the model layer is a handful of labs plus an open-weight crowd.

Understanding "who sits in the same box" matters more than tracking individual headlines — because competition here is rarely one-on-one; it is mostly **camp against camp.**

## The birds-eye map first

This table is the whole chessboard. Read it horizontally as layers, vertically as the main names in each slot:

| Layer | Main players | Open / closed | Who they fight |
|---|---|---|---|
| **Chips / GPUs** | NVIDIA (near monopoly), AMD, Cerebras | Closed | The war on "alternative compute" (custom silicon, cloud in-house design) |
| **Cloud / compute** | AWS, Azure, GCP | Closed | Each other, plus each integrating into the model layer |
| **Flagship labs** | OpenAI, Anthropic, Google, DeepSeek | Closed (DeepSeek mixed) | Each other head-on, plus alliances with clouds |
| **Open-weight** | Meta (Llama), Qwen (Alibaba), Mistral, DeepSeek | Open | The fight against "closed-API dependency," plus leaderboard rivalries |
| **Apps / platforms** | assorted SaaS, agent platforms, office suites | Mixed | Whoever can turn a model into a product people pay for |

## The chip layer: one company feasts

NVIDIA holds the rarest position in this industry — **certain demand, technical lead, ecosystem lock-in.**

* It is the "shovel king": from training to inference, the world's compute demand passes through it first.
* Its moat is not just the chip but **CUDA**: the whole AI software stack is built on its ecosystem, so switching platforms is expensive.
* Challengers come in two flavors: **hardware** (AMD, Cerebras, Google's custom TPUs) and **alternatives** (clouds designing their own silicon, or software that compresses models onto smaller hardware — the compute bar from `train-01-pretraining`).

For now NVIDIA feasts, but the "irreplaceable" halo is slowly being chipped at.

## The cloud layer: three giants play integration

AWS, Azure, GCP nominally sell "GPU-hours," but the real game is **"who has the most complete bundle."**

* The three use capital to assemble clusters of tens of thousands of GPUs, with network, storage, security, and databases — a single chip purchase cannot sustain that scale.
* They all integrate into the model layer: running their own labs (GCP → Google Gemini), investing in others (Azure → OpenAI, AWS → Anthropic), bundling "cloud + model + API" into one invoice.
* The cloud war shifted from "CPU compute" to "who is tied to the strongest model."

> The one thing to remember: clouds are not just "compute landlords"; they are integrators. Whoever holds the strongest model and the cheapest inference can carry off the entire enterprise cloud bill in one bowl.

## The model layer: lab against lab

OpenAI, Anthropic, Google, and DeepSeek fight head-on. Unlike the chip layer's one-company dominance, this one **swaps the leader every few months:**

* **OpenAI**: first-mover halo, the most complete ecosystem and productization (ChatGPT is the biggest app).
* **Anthropic**: safety-first branding and engineering quality; Claude is strong at coding and long documents.
* **Google**: vertical territory of "model + cloud + endpoints (Search, Android)"; Gemini lives inside its own ecosystem.
* **DeepSeek**: forges near-flagship models at radically low cost, spearheading the open-weight wave and challenging the "burn money to train" bill.

The labs' core contest is **"how much must the next generation of models cost to keep up"** — which leads straight to `econ-01-inference-cost`.

## The open-weight camp: a separate front

Meta (Llama), Alibaba (Qwen), Mistral, and DeepSeek run a different play: **instead of API lock-in, hand you the weights.** For the deep dive on open vs. closed, go back to `models-01-open-vs-closed`.

* They compete with each other on leaderboards, but their bigger opponent is the idea of "closed-API dependency" itself.
* Open weights lower the barrier to "having a model": anyone can download, self-host, and sidestep per-token pricing (the home turf of `local-ai`).
* This force also pressures the labs: either use open models as traffic funnels into their APIs, or widen the gap with stronger flagships and better products.

## Who stands on the same side

The real AI politics is **camp versus camp:**

```text
Closed-integration camp:  clouds (the three) ally with flagship labs (OpenAI/Anthropic/Google)
selling "turnkey + strongest capability + complete suite"
Open-self-hosting camp:   Meta/Qwen/Mistral/DeepSeek + developer community
selling "downloadable weights + privacy + predictable cost"
Compute upstream:   NVIDIA sells to both camps but holds the strongest bargaining power
```

## What each player is betting on

Names are not enough; you need **"what is each one betting on."** That predicts its next move and how you should expect it to behave:

| Player | Main bet | If the bet fails |
|---|---|---|
| **NVIDIA** | AI compute demand keeps exploding and CUDA is unshakable | custom silicon plus software compression loosens the "irreplaceable" hold |
| **AWS / Azure / GCP** | tie to a flagship model → the whole enterprise cloud bill | model quality lags and customers move to another cloud |
| **OpenAI** | a moat of scale from first-mover plus productization | open models close in and the differentiation thins |
| **Anthropic** | quality and safety reputation, deep developer and enterprise roots | its applications get cloned directly by another model |
| **Google** | its own ecosystem (Search, Android) fully baked with models | ecosystem advantages cannot cover the model gap |
| **Meta / Qwen / Mistral / DeepSeek** | open weights plus low cost, bypassing API lock-in | fall behind on leaderboards, losing community and commercial users |

## Dependency: who stands on whose shoulders

Mapping "who needs whom" reveals more than "who fights whom":

```text
NVIDIA  ←──── everyone (clouds, labs) needs its chips
  ↑               ↑
AWS ──→ OpenAI   GCP ──→ Google Gemini
  ↑               ↑
Azure ──→ OpenAI   AWS invests in Anthropic
  ↑
model labs ──→ need cloud compute and distribution
  ↑
app layer ──→ needs API quality and model capability
```

Notice two asymmetries:

* **Chips are one-way dependence:** everyone depends on NVIDIA, but NVIDIA depends on no single customer — so it holds the biggest bargaining power.
* **Labs and clouds are mutual dependence:** labs need the clouds' capital and compute; clouds need the labs' models to attract customers. This "mutual need" is exactly why alliance headlines never stop.

## Why the "DeepSeek phenomenon" deserves its own look

DeepSeek is not just "one more lab." It stands for two things:

1. **The cost assumption of training is rewritten:** it approaches flagship quality at a fraction of the cost, questioning the axiom that you must burn hundreds of millions to keep up.
2. **An accelerator of open weights:** its arrival turns "self-host frontier-ish capability" from theory into daily practice, forcing closed labs to re-prove that the API premium is worth it.

We will run the numbers in `econ-01-inference-cost` — but the seed of that article lives with DeepSeek.

## The four relationship types between players

Put two players on the same board and the relationship is not only "competition." There are four, often coexisting at once:

| Relationship | Example | Key signal |
|---|---|---|
| **Competition** | OpenAI vs Anthropic, AWS vs Azure | leaderboards, customer battles, price cuts |
| **Customer / supplier** | labs rent GPUs from clouds, apps pay APIs | where the bills flow, contract length |
| **Investment / alliance** | Azure in OpenAI, AWS in Anthropic | equity headlines, exclusive distribution |
| **Mutual dependence** | Google is both a model maker and a cloud | own-ecosystem priority, bundled sales |

Read the headlines through these four lenses and they stop being isolated events and start reading as chess moves. "AWS invests in Anthropic" is not just investment news — it is simultaneously a move in "AWS countering Azure's OpenAI tie-up."

## How to use this map

* **Find your upstream:** who sets your costs? On the cloud layer, price-shop among the three; on the inference layer, run the long-term math of API vs. self-hosting.
* **Find your rivals:** if you package models into products, your competitors are not just similar SaaS — they are also the labs integrating downward.
* **Watch the camp moves:** one headline about a cloud investing in a lab tells you more about the future than ten model releases.

## Next

All the players are on the board. Now the practical question: **how do they actually make money** — pricing, productization, and whether the math works — `industry-03-business-models`.

#### Q: Why are clouds called 'integrators' rather than just compute landlords?

* Because clouds do not make models

* Because clouds bundle model, cloud, and API into a single invoice, carrying off the whole enterprise cloud spend

* Because clouds do not sell storage

* Because clouds do not charge money

> 💡 The three big clouds run or invest in labs and bundle 'cloud + model + API' as one offering, so the customer's entire cloud bill moves to them. That makes them integrators, not merely compute landlords.
