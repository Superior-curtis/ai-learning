# Attention: How Models Decide Who Listens to Whom

> 📅 2026-08-01 · Core Concepts
> A plain-English walkthrough of self-attention: how Q, K, V let every token consult the whole sentence for context.

---

Attention is the heart of modern large language models (LLMs). From GPT to Claude, almost every model treats attention as the basic unit of language processing. This post unpacks it in plain English: first the problem it solves, then a step-by-step walk through the Q / K / V math, and finally a runnable snippet that ties the whole flow together.

## What attention solves

### A sentence is more than its words

Consider "The cat sat on the mat." On its own, "sat" is just a verb; in the sentence, you know its subject is "cat" and the place is "mat." Meaning emerges from the relationships *between* words.

Recurrent neural networks (RNNs) treated a sentence as an assembly line: process one word at a time and "carry" earlier information to the end. The longer the sentence, the more the early words get diluted — the classic long-distance dependency problem.

Attention works differently. It lets each word, at the moment it is processed, **reach back and consult every other word in the sentence**, deciding for itself who matters and who does not.

### Fixed window vs. dynamic window

* **Fixed window (n-gram / convolution):** only look at nearby words; the window size is hard-coded.
* **Dynamic window (attention):** each word decides whom to look at and how much; there is no distance limit.

That is the key advantage: **every word "sees" the whole sentence at once**, and *how much attention to pay* is something the model can learn.

## The plain-English version: a meeting

Imagine a sentence as a meeting, where each word is an attendee. Attention is the room's "communication rules":

### Step 1: everyone prepares three cards

Each word uses three learned weight matrices to turn itself into three roles:

* **Q (Query):** *Who am I looking for?* — what I am interested in.
* **K (Key):** *Who am I?* — a label others can use to recognize me.
* **V (Value):** *What do I have to offer?* — the actual information I carry.

### Step 2: everyone asks everyone

Compare "my Q" against "everyone's K" to compute a similarity score. The higher the score, the more relevant that token is to me, and the more I should listen to it.

### Step 3: softmax turns scores into weights

Scores can be positive, negative, and wildly different in scale — unusable as weights directly. `softmax` squeezes them into values between 0 and 1 that sum to exactly 1: a proper "attention budget."

### Step 4: weighted sum

Add up each token's V, weighted by the attention weights. High-weight tokens contribute more. The result is a new vector for that position: "this word, after absorbing the whole sentence's context."

## What Q / K / V actually are

From a matrix point of view it is all very clean. Let the input be $X$ with shape `(seq_len, d_model)` (number of tokens × vector dimension):

```python
import numpy as np

def softmax(x):
    x = x - x.max(axis=-1, keepdims=True)  # numerical stability
    e = np.exp(x)
    return e / e.sum(axis=-1, keepdims=True)

def self_attention(X, Wq, Wk, Wv, d_k):
    # X: (seq_len, d_model)
    Q = X @ Wq           # (seq_len, d_k) queries
    K = X @ Wk           # (seq_len, d_k) keys
    V = X @ Wv           # (seq_len, d_k) values

    scores = Q @ K.T     # (seq_len, seq_len) similarity
    scores = scores / np.sqrt(d_k)  # scale for stable gradients
    weights = softmax(scores)       # each row sums to 1
    out = weights @ V    # (seq_len, d_k) weighted sum
    return out
```

`Wq`, `Wk`, `Wv` are weight matrices learned during training. Every input token gets projected into "query," "key," and "value" vectors by these three matrices.

### A picture is worth a thousand words

For three tokens, the attention weight matrix looks roughly like this (each row is one token's attention budget):

```
        "the"   "cat"   "sat"
"the"  [ 0.98   0.01    0.01 ]
"cat"  [ 0.02   0.90    0.08 ]
"sat"  [ 0.01   0.15    0.84 ]
```

How to read it: look at the "sat" row — it gives 15% of its attention to "cat" and 84% to itself. "Cat" is its subject, so it mainly consults "cat." Every row sums to 1.

## Matrix shapes at a glance

| Tensor | Shape | Meaning |
|--------|-------|---------|
| X | (seq\_len, d\_model) | input token vectors |
| Q, K | (seq\_len, d\_k) | queries and keys |
| V | (seq\_len, d\_v) | value vectors |
| scores | (seq\_len, seq\_len) | pairwise similarity |
| weights | (seq\_len, seq\_len) | attention distribution after softmax |
| output | (seq\_len, d\_v) | context-aware representations |

In real models `d_model` is typically `768` to `12288`, with `d_k = d_v = d_model / num_heads`.

## Why divide by √d\_k

`Q @ K.T` is a dot product over d\_k numbers. The higher the dimension, the larger the dot products, and softmax quickly saturates toward a near one-hot distribution — gradients vanish. Dividing by `√d_k` keeps scores at a sane scale; it is a key detail that makes attention trainable.

## From single-head to multi-head attention

Real models almost always use **multi-head attention**: split Q / K / V into several groups, run attention on each group independently, then concatenate the results.

It is like running several breakout sessions at once, each noticing a different aspect:

* one head learns syntactic dependencies ("sat" → "cat")
* one head learns coreference ("it" → "the cat")
* one head tracks positional or special-token patterns

Multi-head lets the model capture many kinds of relationships simultaneously instead of committing to a single way of allocating attention.

## Why this matters for LLMs

* **Bidirectional understanding:** every token can consult tokens to its left and right, so meaning is judged more accurately.
* **Long context:** attention has no fixed distance cap — in principle any position in a 100k-token context can consult any other (the cost: compute grows quadratically with length).
* **Stackable:** transformers stack dozens of attention layers; each layer "re-interprets" the previous representation, which is where model depth comes from.
* **Generation is just attention again:** when predicting the next token, the model re-runs attention over all existing tokens, then predicts the most likely continuation.

## Summary

| Concept | In one sentence |
|---------|-----------------|
| Attention | each token consults all others based on relevance |
| Query | "who am I looking for" |
| Key | "who I am, so I can be found" |
| Value | "the content I actually provide" |
| Softmax | turn scores into weights that sum to 1 |
| Multi-head | parallel breakout sessions, each noticing a different aspect |

Attention is not magic — it is a very concrete set of matrix operations: **query × key → scores → softmax → weights → weighted sum of values**. Understand those five steps and you understand the heart of the transformer.

Next, we'll look at where those vectors come from in the first place — the world of embeddings.
