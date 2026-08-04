# Predicting the Next Word: The Core Trick

> 📅 2026-08-04 · Core Concepts
> A large language model in one sentence: a machine that learned to guess the next word from the huge amount of text it was trained on. This post unpacks that sentence.

---

Large language models (LLMs) look magical. They write essays, write code, answer questions, reason. Peel back that magic and the core is almost embarrassingly plain — **the only thing an LLM does is predict the next word**.

Not "most of the time": **every** output, **every** token, is produced by the same trick. Look at the text so far, then guess which word is most likely to come next. This post tears that trick apart, because understanding it is understanding more than half of AI.

## The one-sentence core

> An LLM = a machine that **learned to guess the next word from a huge amount of text**.

Three keywords, three questions:

* **Huge amount of text** — what did it learn from? (training data)
* **Learned** — how did it learn? (training)
* **Guess the next word** — what is it actually doing? (inference / generation)

We'll start with the last one — "guess the next word" — and work backwards.

## What "guessing the next word" really means

Imagine a sentence half-written:

```
AI moves ______
```

A human fills the blank: "should it be fast, quickly, or forward?" — then picks one. An LLM does almost exactly that, with two differences.

**First, it doesn't pick one word. It computes a score for every word in the dictionary.**

Internally the model outputs a list: for every candidate word, a probability that it's the next word. They all add up to 1.

```
AI moves ____
  fast     42%
  quickly  31%
  forward  11%
  ...
  (every other word in a 100k+ vocabulary, a sliver each)
```

```text
input → model turns the context into "vectors"
↓
scores every word in the vocabulary (logits)
↓
turns scores into probabilities (softmax)
↓
draws one word from that distribution
↓
append it, predict the next one…
```

**Second, it doesn't really "fill in the blank" — it rolls dice.**

The model gets a probability lottery (42% fast, 31% quickly…) and **draws one word at random** according to those odds. That explains a lot:

* The same question twice can get different answers (different draws).
* The model can be made more conservative or bolder by "temperature" (covered in `llmcore-04-sampling`).

## Where those probabilities come from: trained-on "big data"

Now the earlier question: how does the model know "fast" is 42% and something odd is 9%?

The answer is humble: **it has seen the pattern "AI moves ____" and sentences like it an enormous number of times in the text it read.**

Imagine reading the entire internet — tens of billions of sentences. Your "intuition" would soak up regularities:

* See "AI moves", and your next thought is naturally fast, quickly.
* See "The capital of France is ____", and you think Paris.
* See "def main(): ____", and you reach for pass or return.

Training is taking "guess the next word" and drilling it against the internet until it's perfect:

1. Gather a huge corpus (articles, books, code, conversations).
2. Cover the next word of each sentence, ask the model to guess it.
3. When wrong, nudge the model's tens of billions of parameters to do better.
4. Repeat trillions of times until the statistics of language are internalized.

#### Feed a sentence with the next word hidden

"The cat sat on the ____". The model only sees the front, never the answer "mat".

#### The model makes a prediction

From the preceding words, it assigns a probability to every word in the dictionary.

#### Compare with the answer, compute loss

Against the true answer "mat": if it gave mat 1%, record the error.

#### Nudge the parameters

Use gradient descent to adjust the weights so the right word scores higher next time.

#### Repeat billions of times

Over the whole internet, until the statistics of language are internalized.

Here's the key: the model is **not fed knowledge directly**. It learns knowledge *as a by-product* of getting better at predicting. To guess "The capital of France is Paris", it must store the relationship between "Paris" and "France" in its parameters. So knowledge is **forced out** by the "guess the next word" task.

## A runnable, conceptual demonstration

Real inference needs model weights and a tokenizer, but the skeleton of "guess the next word" fits in a few lines. `logits` are the scores the model assigns to each candidate; `softmax` turns scores into probabilities:

```python
# Conceptual demo: not a real PyTorch model, but the exact skeleton
import math

# The model's internal "next-word scores" — higher = more likely
logits = {"fast": 2.1, "quickly": 1.7, "forward": 0.9}

def softmax(scores):
  exp = {w: math.exp(s) for w, s in scores.items()}
  total = sum(exp.values())
  return {w: e / total for w, e in exp.items()}

probs = softmax(logits)
for word, p in sorted(probs.items(), key=lambda x: -x[1]):
  print(f"{word:>8}  {p*100:.0f}%")

# →     fast  42%
# →  quickly  31%
# →  forward  27%   ← the draw could land anywhere!
```

> A real model doesn't output four words — it scores every word in a 100k+ vocabulary. The numbers above are shrunk so you can see the shape.

## Why "only guessing the next word" looks like knowing everything

This is the biggest mystery, and the answer is the prettiest: **language is compressed knowledge.**

When you ask a human to guess the next word after "Einstein proposed the theory of ____", they answer "relativity" because they actually know physics. The model doesn't "actually know". But it read an astronomical amount of text, so statistically it has absorbed that "relativity" almost always follows "Einstein proposed". **Guessing one word correctly leans on the footprints that all the world's knowledge leaves in language.**

And generation is **recursive**: the model advances one word at a time, but it feeds its own output back in and predicts the next one again. Word by word, an entire paragraph emerges from the machine "continuing itself".

## This mental model explains a lot

Once you accept "LLM = a next-word-guessing machine", much confusion evaporates:

| Phenomenon | Explanation |
|---|---|
| Why it sounds so fluent | Trained on an astronomical amount of "speaking"; the statistics of language are internalized |
| Why it confidently makes things up (hallucination) | Some next-word guesses feel natural but are factually wrong — see `llmcore-05-hallucination` |
| Why it occasionally changes its answer | Each output is a draw; ask twice, get a different ticket |
| Why it doesn't know today's stock price | It only read text up to its training cutoff; the world after that it never saw |
| Why it can write code | Code is text too — extremely regular, especially easy to "guess" |

## The most important sentence

An LLM doesn't understand. It has no awareness. It isn't thinking. It is, at bottom, **a machine that repeatedly plays "next-word lottery" across a hundred-thousand-word dictionary — where the lottery skill was honed on the entire internet.**

With that in mind, the next step is to take apart its parts: language isn't processed in units of "characters" (`llmcore-02-tokens`), its visible range is limited (`llmcore-03-context`), and that "draw" can be tuned conservative or bold (`llmcore-04-sampling`).

#### Q: When an LLM generates text, what is each output step, fundamentally?

* Copying an existing sentence from the internet

* Scoring every word in the dictionary, then drawing one from the distribution

* Searching memory for the most similar sentence

* Re-arranging the user’s words verbatim

> 💡 Generation is recursive next-word drawing: the model scores the whole dictionary, draws one word, appends it, and predicts again. Not copying, not searching.
