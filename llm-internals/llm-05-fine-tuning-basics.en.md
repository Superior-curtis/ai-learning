# Fine-tuning Basics

> 📅 2026-08-04 · Deep Dive
> What does fine-tuning actually change inside the weights? How is LoRA technically different from full fine-tuning? What do learning rate, epochs, and rank each control? This post pops the hood.

---

Everyone nods along when the word "fine-tuning" comes up, but ask "what does it actually change inside the weights?" and most people go quiet. `train-02-finetuning` covered whether to fine-tune and what it can teach. This post takes a different camera angle, under the hood: **when you finish a fine-tuning run, the whole story is one sentence — the weights were nudged, and the shape of the language statistics was pushed a little toward your data.**

After attention, embeddings, and quantization, fine-tuning is the last piece of "internal arts" you need to understand how models get shaped into specialists. And let's start with the punchline: fine-tuning is not mysterious. It uses the *same training engine* as pre-training. The only differences are the starting point and the data — which is exactly why it is so much cheaper than the pre-training that came before.

## The same engine, a different target

Fine-tuning = **keep running "next-word prediction" training on top of weights that already know how to talk** — just with your data instead of the internet.

The loop is identical to pre-training:

1. **Forward**: the model predicts on a training example and computes a loss (how far it was from the correct next token).
2. **Backward**: backpropagation computes each weight's contribution to that error — the gradient.
3. **Update**: `W ← W − lr × gradient`, nudging the weights in the direction that reduces the loss.

The only difference from pre-training: the starting point is not random weights, but weights that have already internalized language. So fine-tuning's updates are far smaller, it needs far less data, and the whole run costs orders of magnitude less.

```python
# Concept: one parameter update during fine-tuning
# 1) forward: predict with the current weights → compute loss
# 2) backward: compute each weight's contribution to the loss (gradient)
# 3) update: W <- W - lr * gradient

lr = 1e-5   # typical for fine-tuning; an order of magnitude below pre-training (~1e-4)
epochs = 3  # how many times to see the data; small data, few passes

# W is one layer's weight matrix.
# Full fine-tuning: update every W in every layer.
# LoRA: freeze W, train only the small A and B matrices bolted on (finetune-02-lora).
```

## What shifts is mostly "behavior distribution", not a "knowledge base"

This is the biggest beginner trap. People imagine "fine-tuning = pouring new facts into the model". In practice, fine-tuning mostly learns **behavioral and distributional shifts**:

* **Behavior**: output format, tone, whether to apologize first, whether to use JSON, whether to follow instructions — these "how to respond" rules are what fine-tuning is best at.
* **Knowledge**: for the model to "remember a new fact", that fact has to be *permanently* compressed into some batch of weights. Fine-tuning can do it, but it's expensive and easily degrades into rote memorization.

Recall the mental model from `llmcore-01-next-token`: the model is really just statistics about "which word flows best", stored in its weights. Fine-tuning changes the *shape* of those statistics — making "the phrasings of your domain" more likely to win the draw. "Knowing a fact" and "preferring a certain phrasing" are the same thing at the weight level; one is just far harder than the other.

> Keep this line in mind: fine-tuning is good at changing "how it says things", not at reliably teaching "what is true". For new knowledge that keeps changing, that's context/RAG territory; fine-tuning shapes the phrasing with fixed data.

## LoRA vs full fine-tuning: the technical difference

Under the word "fine-tuning" live two very different price tags. **Full fine-tuning** updates every parameter; **LoRA** freezes all the main weights and trains only a small side path.

```text
full fine-tuning:                  LoRA:
updates every weight W             freezes W
W ← W − lr × gradient              trains only the side path A, B
output = W + A × B
```

The technical differences cut across four dimensions:

| Dimension | Full fine-tuning | LoRA |
|---|---|---|
| Parameters updated | all (7B is seven billion) | only A and B (often under 1%) |
| Memory | main weights + optimizer state, several multiples of the weights | main weights frozen, only the adapter lives |
| Expressiveness | unbounded (full-rank update) | limited by rank to a low-rank space |
| Artifact | a whole new, huge weight file | a few-MB adapter |

In one sentence: **LoRA bets that "the change you need actually lives in a very small low-rank space"** — not always true, but remarkably often in practice. If you care about memory and file size, LoRA is essentially the default; full fine-tuning becomes clearly worth it only when the change you need is so large that a low-rank space can't hold it.

## The hyperparameters: learning rate, epochs, rank

Fine-tuning quality is largely decided by three knobs:

| Hyperparameter | What it controls | Common trap |
|---|---|---|
| **Learning rate (lr)** | how far each step moves the weights | too high → divergence, losing the original ability; too low → nothing learns |
| **Epochs** | how many times the model sees the data | too many → overfitting (memorizing, not generalizing) |
| **Rank (LoRA)** | width of the side path | too big → fat adapter, diminishing returns; too small → underfits |

Practical magnitudes:

* **Learning rate**: fine-tuning usually sits around `1e-5`, an order of magnitude below pre-training's ~`1e-4`. The starting point is already good, so the step size must be small — otherwise you stomp on the language ability that was already there.
* **Epochs**: fine-tuning data is usually small; 2–3 passes is typically enough. The less data, the more you should worry about overfitting — keep a validation set that never touched training.
* **Rank**: start around `r = 8–32`, and raise only if the effect is insufficient.

## What a fine-tuning run actually looks like

Move the camera to a GPU and watch a small fine-tuning run unfold:

1. You feed in a few hundred "input → expected output" pairs.
2. Each batch, the model predicts, computes a loss, and nudges the weights in the right direction.
3. Over time, **training loss steadily drops** — the model starts to absorb the regularities in your data.
4. Meanwhile the weights drift slowly, imperceptibly. It looks like the model is "learning new things", but a truer image is a scale being tipped, bit by bit, toward your phrasing.

Which raises the natural question: when should you stop? That answer is tied to the next concept.

## Overfitting: fine-tuning's #1 enemy

Overfitting is the dividing line between "memorizing the data" and "learning the pattern":

| Signal | Healthy | Overfitting |
|---|---|---|
| Training loss | keeps dropping | drops toward zero |
| Validation loss | follows it down | **turns back up** (memorization begins, generalization collapses) |
| Output quality | stable, generalizes | perfect on training samples, falls apart on any rephrasing |

The defenses are unglamorous: **hold out a validation set, don't get greedy with epochs, do early stopping when needed, and keep the data small and clean.** `finetune-02-lora`'s practical post fleshes these out; for now, hold the direction: "training longer" is not the same as "training better".

There is one more, more fundamental defense: data quality. A few hundred clean, consistent, correctly labeled samples beat a few thousand messy ones — dirty data gets "learned" by the model, and it comes back to haunt you on launch in even weirder forms.

## Behavior vs knowledge: a concrete example

Say you want the model to imitate your support team's tone. After fine-tuning, what happens is: after phrases like "please provide your request", the model's score for "we'll take care of it right away" climbs noticeably — **the lottery has been tilted**. Meanwhile the model's world knowledge barely moves: it still has no idea whether your company has inventory today (that's a RAG question).

That's why "fine-tuning makes the model specialized" and "fine-tuning makes the model smarter" are two different claims — the former moves the shape of a distribution; the latter is far harder.

This is also why fine-tuning demos fool people so easily: run the model on your own training samples and it looks incredible — because those are exactly the samples it just memorized. Judge it on held-out examples instead.

## A table to take with you

| Concept | In one sentence |
|---|---|
| Fine-tuning | keep running next-word training on fluent weights, but on your data |
| What it learns | mostly behavior and distribution shifts (how to say things), not reliable new facts |
| Full fine-tuning | updates every parameter; expensive, highly expressive |
| LoRA | freezes the main weights, trains a low-rank side path; cheap, tiny artifact |
| Learning rate | step size; fine-tuning usually lives around 1e-5 |
| Epochs / rank | passes over the data / width of the side path; push either too far and you slide toward overfitting |

## When to get your hands dirty

This post is about principles. Whether to fine-tune, and how to decide — that's `train-02-finetuning`. Actually doing it, and keeping it cheap — that's `finetune-02-lora`. What this post gives you is a pair of eyes that see through the "fine-tuning mystery": **it is just gradient descent, pushing the shape of the language statistics a little further toward your data.** Read any fine-tuning tutorial with these eyes and you'll see exactly what each step is doing — it never edits "memory", it reshapes a distribution.

#### Q: Why does the expectation that “fine-tuning teaches the model the latest product facts” usually fall flat?

* Fine-tuning never changes the model at all

* Fine-tuning is great at changing “how it speaks” (behavior and distribution), but reliably installing new facts is costly and easily becomes memorization

* Fine-tuning only makes models dumber

* New product facts can only be learned by pre-training

> 💡 Fine-tuning reshapes which phrasings the weights favor. Behavior, format, and tone are learned quickly and well; but a new fact must be permanently compressed into the weights and tends to become rote recall — context or RAG is usually the cheaper way to deliver it.
