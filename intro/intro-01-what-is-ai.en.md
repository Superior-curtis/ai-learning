# What Is AI: A Plain-English Map

> 📅 2026-08-04 · Getting Started
> AI, machine learning, deep learning, LLMs — how are they all related? This post uses one 'layer cake' diagram to map the maze once and for all.

---

"AI" might be the most abused word in media today. The news says "AI is taking jobs", support says "our AI assistant is at your service", and your phone's photo app says "AI sorted these." But none of these "AIs" are the same thing.

This post is the front door to the whole book. No math — just a **map that lets you see the whole landscape**: what AI is, how AI / machine learning / deep learning / LLM relate to each other, and whether today's AI really "understands."

## What AI is

The plainest definition: **AI = getting machines to do things that normally need human intelligence.**

The definition is broad — chess, recognizing pictures, translation, driving, writing poems all count. But as the word is used today, it has almost collapsed into one specific approach: **use lots of data and lots of compute to let a program "learn" a capability** — instead of a human writing out every rule by hand.

In other words, nearly all of today's AI is built around a single idea: **the program learns on its own; it isn't hand-programmed.**

## The layer cake: how four words fit

The four most-confused terms are four nested Russian dolls:

```
        ┌─────────────────────────────────┐
        │               AI                │   widest: any machine doing "intelligent tasks"
        │   ┌───────────────────────────┐ │
        │   │    Machine Learning       │ │  programs that "learn" from data
        │   │   ┌─────────────────────┐ │ │
        │   │   │  Deep Learning      │ │ │  learning with many-layered neural nets
        │   │   │ ┌─────────────────┐ │ │ │
        │   │   │ │      LLM        │ │ │ │  large language models (today's star)
        │   │   │ └─────────────────┘ │ │ │
        │   │   └─────────────────────┘ │ │
        │   └───────────────────────────┘ │
        └─────────────────────────────────┘
```

```One-liner&#x20;version
AI          machines doing "intelligent tasks"
└─ ML      one approach: learning patterns from data
    └─ DL  one technique: many-layered neural networks
        └─ LLM  the hottest neural model today: specialized in language
```

> The key is containment, not equality: every LLM is DL, every DL is ML, every ML is AI — but not the reverse. An action-movie poster isn't an LLM, and a linear-regression house-price model isn't deep learning.

## Layer by layer

**AI (Artificial Intelligence)** — the outermost layer. Everything that gets machines to do intelligent tasks. A 1950s chess program is AI; today's large language model is also AI.

**ML (Machine Learning)** — the milestone: instead of hard-coding rules, **feed lots of examples and let the program learn the patterns itself.** Spam filtering, house-price prediction, recommendation systems — all ML star apps.

**DL (Deep Learning)** — a popular family of ML techniques using many-layered neural networks ("deep"). It dominates because on **complex, irregular** data (images, speech, language) it crushes traditional ML. Nearly every state-of-the-art model today is DL.

**LLM (Large Language Model)** — the breakout star of the DL family: GPT, Claude, Llama. It's "large" (hundreds of billions of parameters or more) and specialized in language. Its entire operation is the "guess the next word" trick we covered first (`llmcore-01-next-token`).

## Where "generative AI" fits

You'll also run into **Generative AI**. It isn't another layer — it's a **capability axis**: it emphasizes that the model can **generate new content** — write text (LLM), draw images (diffusion models), make music — as opposed to only classifying or predicting. Nine out of ten "AI products" you see are generative.

```
Discriminative AI:  judges — is this a cat or a dog? is this spam?
Generative AI:      creates — write an essay, draw an image, make a beat.
```

## Does today's AI "understand" — or not?

This deserves a once-and-for-all answer, because it decides how much to trust AI.

* **It does not "understand."** Inside an LLM is just math computing "which word is most likely next." No awareness, no comprehension, no "thinking."
* **But it behaves as if it understands.** Because language is compressed knowledge (`llmcore-01`), it has "stored" every pattern the world has expressed as statistical regularities.
* **So it "seems to understand" yet confidently makes things up.** It can't tell "this word flows well" from "this fact is true." More in `llmcore-05-hallucination`.

```text
An AI assistant's "thinking" is really:
[your question] → in the parameters, compute "most likely next word" → emit a word
↑
no "understanding" light bulb here — only "which word flows"
```

## You already use it every day

You think AI is far away; you use it constantly:

| What you do | The AI behind it |
|---|---|
| Phone photo app auto-tagging | Computer vision (DL) |
| Autocomplete / autocorrect while typing | A (small) language model |
| YouTube / shopping recommendations | ML recommender systems |
| Your voice assistant answering | Speech recognition + LLM |
| Translation apps | Neural machine translation |
| ChatGPT / Claude writing for you | Large language models |

## One map to hold onto

Think of this book as a search frame: understanding the AI industry is understanding how these layers stick together — from **what it is** (here) → **how it works** (Part 1) → **how to build with it** (Part 2) → **what tools** (Part 3) → **what kind of business it is** (Part 4) → **how to treat it responsibly** (Part 5).

The one thing worth remembering before everything else is this four-layer cake. Every chapter after this, you'll be standing on it.

#### Q: “Every LLM is AI, but not every AI is an LLM” — is that correct?

* Yes: an LLM is a special case of AI, not the other way around

* No: AI and LLM are the same thing

* Yes, because AI is smaller than an LLM

* No, because an LLM is bigger than AI

> 💡 Exactly: LLM is a DL, DL is an ML, ML is an AI — containment, not equality. So every LLM is AI; but plenty of AI (chess programs, rule systems) is not an LLM at all.
