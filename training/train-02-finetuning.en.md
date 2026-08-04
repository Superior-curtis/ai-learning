# Fine-tuning: Teaching It a Specialty

> 📅 2026-08-04 · Core Concepts
> A pre-trained model can only continue text. Fine-tuning keeps training it on your data on top of the pre-trained weights, to teach it to obey or specialize — and it's orders of magnitude cheaper than pre-training.

---

`train-01` said pre-training is the model's "basic education" — but a pre-trained model only fluently continues text; it doesn't obey and knows nothing about your domain. **Fine-tuning** keeps training an already pre-trained model on **your own data**, to teach it a specialty — at a cost orders of magnitude below pre-training.

That said, fine-tuning is not a cure-all. Many needs are solved by prompting or RAG instead (see `finetune-01-finetune-vs-rag`). This post first makes clear that fine-tuning *is* "changing the model's weights."

## What fine-tuning is

One line: **continue the model-training flow on top of the pre-trained weights, but with your data instead of the internet.**

```text
Pre-training:  internet text ──→ a model that "speaks general language"
Fine-tuning:   your data   ────→ the same model, "speaks your domain & format"
```

The two use the same mechanics (next-token prediction, error, weight updates). The difference is the **starting point** and the **data**:

* Start: pre-training begins from random weights; fine-tuning begins from weights that already speak well.
* Data: pre-training uses massive general text; fine-tuning uses small, focused data for your need.

Because the starting point already speaks, fine-tuning only needs to *adjust*, not *re-learn* — so it needs far less data and costs far less.

## What fine-tuning usually teaches

**1. Behavior (format, style, tone)**

"Reply in Traditional Chinese, as JSON, within thirty words"; a support model that "apologizes before resolving." These are behaviors, not knowledge.

**2. Specialization (fixed skills, domain patterns)**

A legal model that knows your company's contract terminology; a code model that uses your internal libraries. These are "compressions of a specific domain."

> The spirit: fine-tuning uses your data to push a model from "general" toward "specialized". It fits fixed, predictable, high-volume, repeating patterns; it does not fit "different every time, constantly updating knowledge."

## Fine-tuning vs. prompting vs. RAG: don't rush the choice

This is the most-asked AI engineering question. The deciding factor is "are you changing behavior or knowledge?":

| Approach | Changes | When |
|---|---|---|
| **Prompting** | That turn's input | Behavior, format, one-off |
| **RAG** | Content of each input | Knowledge that can change anytime |
| **Fine-tuning** | The model's weights | Fixed skills, tone, domain patterns |

Principle: **prompt first, then RAG, fine-tune last.** Fine-tuning is the heavy hammer (weights, validation, retraining), but some fixed patterns genuinely only fine-tuning can nail.

## LoRA: the secret to cheap fine-tuning

You might think touching hundreds of billions of parameters is expensive. So the industry almost always uses **LoRA** — instead of changing all weights, **add a small set of low-rank parameters on a side branch, and train only that sliver.**

#### Freeze the base model

Keep the pre-trained weights fixed (not updated).

#### Add a "side branch"

Insert small low-rank matrices near some layers (millions of parameters, not billions).

#### Train only the branch

Run the fine-tuning flow, but update only the small branch.

#### Merge / save

When done, add the branch back into the weights (or keep it as a tiny file).

```LoRA&#x20;parameter&#x20;magnitudes
# Full fine-tuning: update all 7B parameters
# LoRA: train only a few million adapter parameters (< 1%)

# Modern frameworks (HF PEFT / Unsloth…) switch in one line:
#   from peft import get_peft_model
#   model = get_peft_model(base_model, LoraConfig(r=16, ...))
# Result: GPU needs drop to ~1/3, output is a portable little file
```

This means: **fine-tuning doesn't need enterprise compute.** With LoRA, even a consumer GPU can do small-scale fine-tuning on some models, producing a few-dozen-MB adapter file you can carry around.

## Three common fine-tuning pitfalls

**1. Data volume fantasy.** Fine-tuning isn't "more data is better." Repetitive, low-quality data is actively harmful — little and clean beats lots and dirty.

**2. Ignoring validation.** Overfitting (memorizing the training set, failing to generalize) is common. Always keep a validation set that **never saw training.**

**3. Using fine-tuning for "knowledge updates."** Your product's knowledge changes weekly? That's a RAG problem, not a fine-tuning problem. Fine-tuning learns the *fixed*; it can't learn "changes every time."

> Rule of thumb: fine-tuning is a scalpel, not a staff. Use prompting and RAG to scope the problem first; only fine-tune once you've confirmed "this really needs weight changes." Details in finetune-01-finetune-vs-rag, finetune-02-lora.

## A mental model that sticks

Think of the model as a generalist who "speaks beautifully but has never seen your world":

* **Prompting** = you recite the rules at every meeting.
* **RAG** = you hand it the relevant material when it works.
* **Fine-tuning** = before a long-term hire, you send it to your specialty bootcamp.

Next, the finishing step after fine-tuning: making the model **match human preferences** — RLHF (`train-03-rlhf`).

#### Q: Your product's knowledge 'updates to the latest version every week'. Which approach fits best?

* Fine-tuning — bake the latest knowledge into the model

* RAG — put the latest docs in retrieval, feed them as context when needed

* LoRA — open a small branch to learn each week’s new knowledge

* Pre-training — re-read the internet

> 💡 Fine-tuning learns fixed patterns; it can't learn 'changes every time'. Knowledge that changes → use RAG, which puts fresh content into each input without touching weights.
