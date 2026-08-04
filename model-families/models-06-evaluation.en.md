# Evaluation and Benchmarks: Measuring How Strong a Model Is

> 📅 2026-08-04 · Deep Dive
> MMLU, coding evals, math benchmarks, leaderboards — how we quantify 'how strong'; its three big holes (saturation, cheating, contamination); and why your own task set is the most honest measure.

---

Every news article you open says "the O-model family tops the leaderboard again" or "the new model breaks 90% on MMLU." But you and I both know that **how usable a model is, and whether a leaderboard number truly corresponds to that, are two different things.**

The previous five posts (`models-01` → `models-05`) took a thorough look at models. This is the final stop, addressing the most basic and most easily overlooked question: **how do we fairly measure "how strong a model is"?** The answer turns out to be less scientific than you think.

## First, understand benchmarks: slicing "strong" into individual problems

To measure "how strong," gut feel like "it chats smoothly" isn't enough. So academia and industry invented **benchmarks**: a fixed set of problems with standard answers; have the model answer them, then score it. Common categories:

| Benchmark | What it measures |
|---|---|
| **MMLU** | Multiple-subject choice questions — breadth of human knowledge |
| **Coding evals (HumanEval / MBPP…)** | Given a function requirement, write code that passes tests |
| **Math evals (GSM8K / MATH…)** | Multi-step computation and reasoning, one unique answer |
| **Long-text comprehension / summarization** | Understanding and organizing after reading long documents |

The higher the score, the stronger it is **on that category of problems**. Note the phrase "that category" — a high MMLU score doesn't mean it writes code well, nor that it understands your company's business.

## Why you can't look at just one benchmark

A common mistake: assuming there's only one kind of "strong." But a model's ability is **multi-dimensional** — it can be erudite yet bad at code, or a coding star yet average at long-text comprehension. A single benchmark is like "measuring height and ignoring weight" — you'll never see the full picture.

The mature approach is to look at a **profile across a set of benchmarks**: a couple for general knowledge, code, math, long text, and reasoning. `models-04-reasoning` noted that reasoning models are great at math yet worse at creative problems — a single total score washes that away; only a profile can see it.

## Leaderboards: blending a pile of benchmarks into one rank

A **leaderboard** takes the scores of several benchmarks, averages them with weights into one total, and lines them up. Convenient for summarization, but it hides two traps:

* **The weights are subjective**: who decides MMLU gets 30% or 50%? Different weightings crown different champions.
* **The total dilutes information**: a mediocre model that scores 85 everywhere can beat a lopsided expert at "math 95, conversation 60" — yet the latter may be far more useful to you.

So leaderboards are good for "a quick scan of who leads the pack," not as a bible for "who you should use" — exactly the proof behind `models-02-families-tour`'s "don't be held hostage by a leaderboard."

## The three big holes: benchmarks aren't as honest as they seem

Even though benchmarks start with good intentions, in practice they have three hard flaws:

1. **Saturation**: model after model gets stronger, and many benchmark scores approach a ceiling (90%+); the problems have become "too easy" for front-runners and no longer separate them. When a field is too easy, the ruler loses its tick marks.
2. **Cheating (gaming / overfitting)**: some teams **tune parameters specifically to a benchmark** — making the model gain MMLU points while its real-world ability doesn't improve. The score becomes a "trained-up illusion."
3. **Test contamination**: benchmark questions frequently **already ended up in the training data**. The model may have "memorized" these questions, so what you measure is memory, not reasoning. That's the other face of `llmcore-05-hallucination` — answer by rote if memorized, fabricate if not.

## Why "strong" keeps changing definition

At the end of the day, "strong" is a relative, drifting standard. Three forces keep moving it:

1. **The opponents get stronger**: last year 90 was the ceiling; this year 95 is routine — the number stays the same, its meaning changes.
2. **The tasks change**: two years ago nobody benchmarked chart understanding in multimodal models; now it's a core capability. Benchmarks must keep adding questions to keep up with the times.
3. **Models grow "along the ruler"**: this differs from cheating — vendors genuinely optimize toward what the leaderboard rewards. Short term it looks like gaming; long term it's the evolution of general capability.

Understanding that "strong" is dynamic saves you frustration: **a leaderboard from last year is worth far less than one from this year.** It's the same reminder as `models-02-families-tour`'s "specific models are data you update frequently."

## So what should we trust: your own task set

Precisely because benchmarks have these holes, the most honest evaluation in practice is **a set of your own, real tasks (an eval set)**.

The method is plain to the point of boredom — yet closer to your needs than any leaderboard:

#### Collect

Gather inputs you actually meet — the way your users actually ask, the data your product actually receives.

#### Define answers

Write the ideal answer or a workable scoring rubric for each problem.

#### Score

Have candidate models answer each, graded by your criteria.

#### Compare

Run the same task set across several models, or across versions of the same model.

This is the formal version of `models-02-families-tour`'s "test each family on your own problems," and the same logic `security-03-evals` uses in the security domain. It is also the pattern that turns a model from "something a vendor claims" into "something you have verified."

## How many problems is enough

You don't need a hundred on day one. What people overrate is "count"; what they underrate is "closeness." The point isn't the number of problems — it's that **they're the ones your work actually encounters.**

| Dimension | Example direction |
|---|---|
| Correctness | What is the standard answer to this problem |
| Format | Did it output in the format you asked for |
| Style | Tone, length, whether it sounds like your brand |
| Edge cases | How it responds when input is weird, long, or vague |
| Stability | Ask the same question three times — is it consistent |

A useful habit: whenever a real user complains about an answer, add that case to the eval set. Over time the task set becomes a living record of exactly what your product keeps getting wrong — which is a far better improvement guide than any leaderboard gap.

## From hand-grading to "model judges"

Once your task set grows, hand-grading every answer gets exhausting. The industry started using **"judge models"** — a (usually closed, stronger) model that grades the candidate model against the rubric you wrote. It's fast and consistent, but it has risks: the judge can misjudge too, or inherit biases. So for high-stakes questions, keep a human in the loop at the end.

> Remember one sentence: benchmarks and leaderboards are "someone else's eyes"; your task set is "your eyes." Benchmarks can saturate, can be gamed, can be contaminated — only your own few dozen real tasks measure "how strong it is for you," and that is the only number you should trust.

## A minimal start to your own eval

Don't wait for "perfect" — just start. The most bare-bones beginning:

```python&#x20;—&#x20;a&#x20;minimal&#x20;eval&#x20;skeleton
# pseudocode: only three actions, collect, run, score
qas = [                       # your real tasks
{"q": "Explain multimodality in three sentences", "ideal": "...rubric..."},
...
]
for model in ["openai", "anthropic"]:
  for qa in qas:
      out = model(qa["q"])
      score = judge(qa["ideal"], out)   # human or model judge
```

Starting with **fifteen** real tasks is already effective. What matters is that they're yours, not copied from someone else.

## Turn evaluation into your regression test

The greatest value of a task set isn't using it once at "model-buying time," but **running it long-term as a regression test.** Every time you change your prompt, swap the RAG vector store, or update the model version, run the task set and watch for unexpected score drops.

| Change | The eval to run |
|---|---|
| Prompt change | Confirm previously-correct questions aren't broken |
| Model / version change | Compare scores to decide whether to upgrade |
| RAG or chunking change | Confirm retrieval quality hasn't regressed |
| New feature | Extend the task set and verify the new capability |

This is `security-03-evals` and `rag-04-evaluating` applying the same philosophy: **every change needs a ruler to measure it.** Without regression tests, a system breaks one change at a time — often quietly.

## One-sentence summary

"How strong a model is" has no single answer — only "strong for whom" and "strong on what class of problems." Benchmarks and leaderboards offer the convenient "someone else's eyes," but the three mountains of saturation, cheating, and contamination make them less reliable. What's truly trustworthy is **a ruler you built yourself: a real task set.**

With that, the `models-` series (`models-01` → `models-06`) is complete: open vs. closed, the family tour, multimodality, reasoning, small models, and evaluation. You now have a full map of what a model is, how to choose one, and how to measure it.

Next, we shift focus from "the model itself" to "how to command it" — **prompt engineering**, the real lever for making models useful (`prompt-01-basics`).

And as you start commanding models — editing prompts and RAG — carry the final lesson of this series with you: **run your own task set after every change.** That's the insurance that keeps a system getting better instead of drifting.

#### Q: You are a product manager deciding which new model to adopt. Which evaluation is most honest for YOUR product?

* Look at the latest leaderboard champion

* Run 15 of your product real inputs and score them with your own criteria

* Only trust the vendor claimed MMLU score

* Just look at how many parameters the model has

> 💡 Benchmarks and leaderboards can saturate, be gamed, and be contaminated, and vendor scores are just marketing. The honest evaluation is to test with your product real inputs using your own scoring criteria — that is the answer to how strong it is for your product.
