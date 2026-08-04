# RLHF: Making Models Match Human Preferences

> 📅 2026-08-04 · Core Concepts
> A pre-trained model is brilliant but disobedient: it lies, it's lazy, it backfires. RLHF uses 'human scores' to derive a reward model, then teaches the model to 'behave'.

---

By now the model speaks well and reasons well — but it's still not a "good" assistant. It might:

* Confidently fabricate answers (hallucination, `llmcore-05`).
* When it doesn't know, squeeze out nonsense instead of saying "I don't know."
* Sycophantically agree with you — you say black, it says black.
* Refuse to help, or backfire.

The process that turns a model from "brilliant but hard to work with" into "genuinely useful" is called **RLHF (Reinforcement Learning from Human Feedback)**. It's the step behind ChatGPT-class products' original wow.

## The full pipeline: pre-training → SFT → RLHF

The standard three steps for a modern "assistant" model:

```text
Step 1  Pre-training      learn language & knowledge (guess the next word)
Step 2  SFT               learn the format & behavior of "answering questions"
Step 3  RLHF              learn "to match human preferences" — helpful, honest, harmless
```

`Step 2` (SFT) is, at heart, fine-tuning (`train-02`): take examples of "how humans answer", and reshape the model's "continue text" into "answer the question." But it only mimics human examples — **human examples are finite and can't cover everything.** RLHF goes further: it teaches the model *which behaviors are better*.

## A stew of three ingredients

RLHF needs three roles: **human feedback**, a **reward model**, and the **policy model being trained.**

**1. Collect human preference data (the expensive part)**

Hire annotators, show them two model outputs, ask "which is better." Tens of thousands of "preference pairs" accumulate into a map of what humans prefer.

```
Output A: Sure, I'll follow up on your project progress……        ← useful, specific
Output B: OK 😊👍                                                 ← perfunctory but harmless
Human: A is better → record a "A beats B" pair
```

**2. Train a reward model**

From those "A beats B" pairs, train a small "scorer": given an output, it returns a score — **predicting how much a human would like this answer.** It learns "which kind of talk humans prefer."

**3. Use the reward model to teach the policy model (PPO)**

Let the policy model keep generating; score each answer with the reward model; reinforce high-score directions, suppress low-score ones. Over millions of steps of generation, the model is shaped by the "reward signal" into something more aligned with human preferences.

```text
humans score (A beats B)
↓
reward model: learns to "score"
↓
policy: generate → reward model scores → reinforce high / suppress low
↓              (repeat for millions of steps)
a model that's "more useful, more honest, more harmless"
```

## How this relates to "alignment" (Part 5)

You might think this is what `alignment` is about. Let's clarify:

* **RLHF is the "method"**: a concrete technique in the training phase (make the model match human preferences).
* **Alignment is the "goal / field"**: making AI's goals match human intent — far broader, covering safety, ethics, AGI risk, etc. (see `align-01-what-is-alignment`).

RLHF is **one concrete step of alignment inside the training pipeline** — not all of alignment. Alignment also covers things "scoring" can't solve (e.g., a model being too sycophantic, or fundamental safety boundaries).

## You can't (and don't need to) run RLHF yourself

Honestly: **RLHF isn't for you to do.** Reasons:

* **Massive human scoring**: tens of thousands of pairs need a professional labeling team and money.
* **RL infrastructure**: PPO training is extremely engineering- and compute-heavy.
* **Off the shelf exists**: the models you get via API already went through full RLHF — you're using "the finished product."

So for 99% of people, RLHF is a "understand it, be grateful, use it directly" section. If you genuinely want to control behavior, prompting or open, already-aligned models is more practical.

> RLHF's place in one sentence: it's the finishing step that turns a "brilliant but disobedient genius" into a "reliable assistant" — done by the oligarch labs; you're the one using the product.

## The newcomer: DPO (no reward model)

RLHF depends on "preference pairs" + reinforcement learning, which is heavy. A lighter approach, **DPO (Direct Preference Optimization)**, has taken off: it also uses "A beats B" data, but **skips the reward model and RL** — it directly changes the loss function to make the model prefer the chosen option.

```
RLHF: preference data → reward model → RL (PPO) → alignment
DPO : preference data → change the loss directly → alignment (no reward model / no RL)
```

It's cheaper, more stable, and easier to reproduce. Much recent open-source alignment uses DPO-style methods. The core idea is the same: **use "human preference" as a signal to teach the model how to be better to humans.**

## One-sentence recap

A model's life is taught like this:

1. **Pre-training** — a genius is born (speaks, but doesn't obey).
2. **SFT** — learns the format of answering questions.
3. **RLHF / DPO** — learns to be "helpful, honest, harmless", becoming an assistant you can use.

Next, meet the **kinds** of models: open vs. closed, and what distinguishes the families — `models-01-open-vs-closed`.

#### Q: In RLHF, what is the main job of the 'reward model'?

* Generate more training data for the model

* Learn from human comparisons, then give any output a score for how much a human would like it

* Convert model output to JSON

* Write a better prompt for humans

> 💡 The reward model learns from 'A beats B' human comparisons, becoming a 'preference signal' scorer: the policy model is shaped by reinforcing its high scores and suppressing low ones.
