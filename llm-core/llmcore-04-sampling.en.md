# Sampling & Temperature: How the “Guessing” Can Be Tuned

> 📅 2026-08-04 · Core Concepts
> Why does the same question get different answers twice? Because the model draws, it doesn't pick. Temperature, top-p, and top-k are the three knobs that decide whether it's conservative or unhinged.

---

Recall from `llmcore-01`: the model computes a probability distribution over every candidate word, then "draws" one. This post unpacks that "draw" — because **how you draw from the distribution is tunable**, and three knobs (temperature, top-p, top-k) decide whether an AI's answer is conservative and stable, or creative and off-the-rails.

## Drawing vs. picking

Let's set up the contrast. Given a probability distribution, there are two extreme strategies:

| Strategy | Behavior | Consequence |
|---|---|---|
| **Greedy** | Always take the highest-probability token | Stable, repeatable, but rigid and prone to loops |
| **Sampling** | Draw by probability | Diverse, natural, but different each time |

Real products sit in between: **they sample, but use knobs to control how wild the draw gets.** That's why asking the same question twice gives different answers — not a bug, just different lottery tickets.

## Temperature: sharpening or flattening the distribution

Temperature is the most famous knob. It doesn't change the ranking, only the gaps:

* **Low temperature (near 0)**: widens the gap between high- and low-probability tokens → almost equals picking the highest → conservative, certain, like reading from a script.
* **High temperature (e.g. 1.5+)**: flattens the gaps, so low-probability tokens can win → creative, offbeat, occasionally unhinged.
* **Temperature = 1**: raw draw, no adjustment.

Intuitively, temperature applies a root to the distribution: roots shrink big numbers and grow small ones (relatively).

```text
original          temperature low (0.2)   temperature high (1.5)
fast 42%          fast 88%                fast 30%
quickly 31%       quickly 11%             quickly 27%
forward 11%       forward 1%              forward 19%
other 16%         other 0%                other 24%
(fairly flat)     (nearly locked on fast)  (anyone can win)
```

> The spirit: temperature controls "predictive gambling". Want facts, code, definite answers → low temp; want stories, ideas, a relaxed tone → high temp. Many products use 0–1, and above 1 things start to "drift".

## top-p and top-k: letting only high-scoring words compete

Temperature lets low-probability words occasionally win, but that's sometimes too dangerous (e.g., a mistyped API name). Two other knobs **filter out low-scoring words directly**:

* **top-k**: keep only the top-k highest-probability words, drop the rest. top-k=10 → only the top 10 can be drawn.
* **top-p (nucleus sampling)**: start from the highest-probability word and keep adding until the cumulative probability passes p (e.g. 0.9). In other words, "keep the group that accounts for 90% of probability" and cut the long tail.

You can combine both: top-k cuts to 10 candidates, then top-p trims the tail of those 10.

```A&#x20;conceptual&#x20;sampling&#x20;implementation
import random

logits = {"fast": 2.1, "quickly": 1.7, "forward": 0.9}
# 1) softmax → probabilities
probs = {"fast": 0.42, "quickly": 0.31, "forward": 0.27}

# 2) temperature: score ÷ temperature (lower temp → bigger gaps)
# 3) top-k: keep only the top-k highest-probability words
# 4) draw
picked = random.choices(list(probs), weights=probs.values(), k=1)[0]

# Commercial APIs take the params directly:
#  temperature=0.2, top_p=0.9, top_k=40
```

## Combining the three knobs: a decision table

| Use case | Temperature | top-p / top-k |
|---|---|---|
| Translation, code, math | Low (0–0.3) | On (conservative, fear typos) |
| General chat, summarization | Medium (0.7) | Off or high (0.9) |
| Writing, brainstorming | High (0.9–1.3) | Off (let low-probability words in) |

> Rule of thumb: decide "should this task be certain or creative", then set the temperature. Letting code generation run on high-temperature sampling is a fast track to inspiration — and to bugs that are a pain to find.

## Why products look "stable"

You might ask: then why does ChatGPT seem to give about the same answer every time?

Because products do two things:

1. **Set a low temperature** — conservative by default.
2. **Cage the model with a system prompt** — instructions that fix the format and tone.

So "stable products" are the result of "low temperature + strong prompt", not an inherently stable model. Want more control? Lower the temperature and write an explicit output spec.

## Back to the original question

"Why does the same question give different answers twice?"

* Because products sample (not greedy).
* Higher temperature → bigger differences; temperature near 0 → differences near zero.
* If you must have identical output, set temperature to 0 (many APIs support it).

Now that you understand the "draw" knobs, the natural next step is its dark side: when the model **draws confidently but wrong** — that's hallucination (`llmcore-05-hallucination`).

#### Q: To make code generation 'as stable and correct as possible', how should you tune?

* Raise temperature for more creative output

* Lower temperature and consider top-k / top-p to drop low-probability words

* Set top-p as low as possible so more words make the list

* Turn off sampling and go greedy; the params don’t matter

> 💡 Code favors correctness and repeatability: low temperature magnifies gaps (approaching greedy), and top-k/top-p then cut low-probability words — a double clamp against errors. High temperature only invites random typos and imagined APIs.
