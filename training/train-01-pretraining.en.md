# Pre-training: Reading the Entire Internet

> 📅 2026-08-04 · Core Concepts
> The day a model 'learns' is really months of one task — 'guess the next word' — run over terabytes of text on thousands of GPUs. That's pre-training.

---

`llmcore-01` said an LLM is essentially "a next-word-guessing machine." So how did that machine get trained? The answer is **pre-training**: one task — "guess the next word" — run over internet-scale data on thousands of GPUs for months.

Pre-training is the starting point of every modern model. It decides how much language and world-knowledge a model has, and who can even afford to play.

## What pre-training does

In one line: **feed the "guess the next word" task a whole internet's worth of text, and drill it billions of times.**

```text
[articles, books, code, conversations…]
        ↓ tokenize
[one long sequence of tokens]
        ↓ at each position
hide the next token → model guesses → check → adjust weights
        ↓ repeat
thousands of GPUs × months × trillions of tokens
```

The per-step mechanics are in `llmcore-01` (feed a sentence, guess the next, compute the error, nudge the parameters). What defines pre-training is **scale**: data measured in hundreds of trillions of tokens, parameters in the tens-to-hundreds of billions, hardware measured in "thousands of $40k GPUs."

## Scale too big to picture

Feel the numbers with three dimensions:

| Resource | Magnitude |
|---|---|
| Training data | Tens of trillions of tokens |
| Model parameters | Tens of billions to trillions |
| Compute | Hundreds of thousands to millions of GPU-hours |
| Cost | Millions to hundreds of millions of dollars |

> Pre-training is a rich-person's game: one full pre-training run's bill exceeds most companies' entire annual budget. That's why the number of labs that can train a large model from scratch fits on one hand.

## What pre-training teaches

Drilling "guess the next word" to perfection yields three things *as a by-product*:

1. **Language** — grammar, syntax, word choice all internalized.
2. **World knowledge** — "Paris is in France", "photosynthesis", all compressed into the parameters.
3. **A seed of reasoning** — because text is full of "because/therefore" and "if/then", the model learns to continue along logic.

The key: **none of this is "taught" — it's forced out by the guessing task.** You never tell it "the capital of France is Paris", but to guess that sentence correctly, it must store that relationship in its parameters.

## Pre-training ≠ learning what you want

A pre-trained model can do one thing: **fluently continue text.** It won't obediently answer your question, won't follow "reply in JSON", won't know it's a support agent. It's like a brilliant, disobedient genius.

Two things turn it into a usable product:

* **Fine-tuning** — teach it to obey and specialize — next post `train-02-finetuning`.
* **RLHF (human-feedback alignment)** — teach it to match human preferences — `train-03-rlhf`.

## Why you don't need to pre-train yourself

You almost certainly never need to run pre-training. Practical reasons:

* **Too expensive**: one full run costs astronomical money.
* **Off the shelf exists**: open models (Llama, Mistral, Qwen…) release their pre-trained weights.
* **Fine-tuning is orders of magnitude cheaper**: fine-tuning on your own data can cost a millionth of pre-training.

So for 99% of people, **"using a model" = downloading or calling an already pre-trained model; "training" = fine-tuning on top**, not pre-training from scratch.

#### Decide the need

Out of the box (call an API)? Specialized (fine-tune)? Or (almost never) from-scratch pre-training?

#### Pick an already pre-trained model

Open models: download weights. Commercial: use the API. Don\\

#### ,&#xA;  },&#xA;  {&#xA;    title:&#x20;

,
&#x20;   content:&#x20;

#### ,&#xA;  },&#xA;  {&#xA;    title:&#x20;

,
&#x20;   content:&#x20;

> Remember pre-training's place in one sentence: it's the model's "basic education" — the most expensive step, played by few; you stand on top of it and build your product with far cheaper fine-tuning and prompting.

## The loss curve: how you know it's "getting better"

Pre-training scores itself with **perplexity (PPL)** — roughly "out of how many candidates the model hesitates, on average, at each guess." Lower PPL means it's better at locking onto the right next word, i.e. better at language.

```Concept:&#x20;lower&#x20;PPL&#x20;is&#x20;better
# perplexity ≈ exp(mean log-prob). Lower = more sure of the next word
# Good model (low PPL): sees "AI moves ____" → fast at 0.9 probability
# Poor model (high PPL): every candidate similar, like rolling dice

# During training you watch PPL fall = the model improving
#   1000 steps: PPL 8.4
#  10000 steps: PPL 4.1
# 100000 steps: PPL 2.3   ← getting better at "guessing"
```

## Why this post matters

Pre-training answers "how does a model learn to talk." It plants two seeds: **pre-training is expensive but fine-tuning is cheap** (next post), and **a model has learned to continue text, not yet to obey** (RLHF, `train-03-rlhf`).

#### Q: Why is 'pre-training a large model from scratch' not the right choice for most people?

* Because pre-trained models are lower quality

* Because pre-training is extremely costly, and off-the-shelf open models or APIs already exist

* Because pre-training can’t handle Chinese

* Because pre-training requires strong programming skills

> 💡 One full pre-training run costs millions to hundreds of millions of dollars, and open models and APIs already hand you pre-trained weights. For 99% of people, the right path is 'use an existing model + fine-tune + prompt'.
