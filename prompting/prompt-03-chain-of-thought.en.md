# Chain-of-Thought: Reason Step by Step

> 📅 2026-08-04 · Core Concepts
> A simple 'reason step by step' often takes a model from often-wrong to clearly-reasoned and far more accurate. But it has costs and traps — this post covers when to use it and when it backfires.

---

The previous two posts were about saying things clearly and giving the model structure. This one targets the model's "thinking" directly: **getting it to reason step by step.**

Start with a counterintuitive fact. Ask a reasoning-heavy question — arithmetic, logic, a decision — and the model often answers with something confident and badly wrong. Add one sentence and accuracy jumps. The sentence is:

> Think step by step, then give your conclusion.

That is **chain-of-thought** (CoT for short).

## Why step-by-step helps

Back to the `llmcore-01-next-token` mental model: an LLM predicts one word at a time. If it rushes straight to the answer, every word in between must land in one shot — and one-shot tends to land on whatever is statistically smoothest, not logically correct.

"Think step by step" forces it to **emit more intermediate text.** Each extra round gives it a chance to:

1. Break the problem into pieces ("first compute…", "now consider the second condition…").
2. Let each step carry its own predecessor forward, forming one continuous path.
3. At the final word, draw from a distribution shaped by the reasoning — not by first instinct.

In other words, you are using text to build it a **longer bridge to walk across.**

## A concrete comparison

Ask it: "A restaurant has 12 tables that seat 4. Tonight there are 57 guests. How many tables are needed?"

**Direct:**

> 57 / 4 = 14.25, so 15 tables.

That one got lucky. Ask it to go step by step:

```chain-of-thought
Reason step by step: A restaurant has 12 tables that seat 4. Tonight there are 57 guests. How many tables are needed?

Step 1: Compute the total capacity. 12 tables x 4 seats = 48 people.
Step 2: Tonight there are 57 guests. 57 - 48 = 9 people with no seat.
Step 3: Those 9 need 9 / 4 = 2.25, so we need 3 more tables (4 seats each).
Step 4: Total tables needed: 12 + 3 = 15.
Conclusion: 15 tables.
```

Look at the difference: **the second approach forces the model to first establish the intermediate fact that capacity is only 48.** That fact is what makes it notice, at step 2, that there is not enough room — and then follow the correct path. Asked directly, the model often skips that step and slides into the statistical instinct of "57 / 4".

## When to use it

CoT is not glue for every crack. It shines on **multi-step, decomposable** tasks:

| Task type | Direct | With CoT |
|---|---|---|
| Math / arithmetic | error-prone | much better |
| Multi-condition logic | misses conditions | clearly better |
| Code tracing / debugging | guesses a lot | better |
| Summarization, translation, simple Q\&A | already good | barely any gain, wasted tokens |

One-line rule: **use CoT when the answer needs to be computed, not looked up.** When the task is just re-presenting known information, CoT adds nothing.

## When it backfires

CoT has two famous traps.

**First, it can reason off a cliff — confidently.** The step-by-step reasoning is not guaranteed correct at each step. Once an intermediate step is wrong, the model follows the error forward with total confidence. That is error propagation. More steps do not mean more correct.

**Second, it amplifies sycophancy.** If your prompt says "I trust your abilities, you can definitely solve this", the model may want to please you — and CoT gives it room to stage a full, plausible-looking performance that ends at the answer you expect rather than the correct one.

In other words: **CoT makes the model think more, but it also gives it more room to think for show.**

## Optional-verbatim CoT vs hidden CoT

Since the reasoning text itself has both value (accuracy) and risk (error propagation, leaking the reasoning), practice splits into two camps:

* **Optional, verbatim CoT**: "reason step by step" becomes a switch you can turn on and off. Turn it on when you need a justification or a verifiable trail; turn it off when you only want the answer. Many models now expose a `thinking` parameter in their API — the same idea, made first-class.
* **Hidden CoT**: the model reasons internally, then outputs only the conclusion. Cleaner output, and the reasoning is not copied or misused — but you cannot audit whether the reasoning was sound. That is a real trade-off in regulated settings (medical, legal, compliance).

> One sentence to remember: CoT trades extra intermediate text for accuracy — it thinks more, but not necessarily in the right direction. Treat it as an option, not a belief: turn it on when you need a reason, turn it off when you want a fast answer.

## Practical rules of thumb

Condensing all of the above:

1. **For multi-step problems, explicitly ask for step-by-step reasoning** and ask it to write each step down.
2. **Do not use it for trivial tasks** — it only gets slower and pricier.
3. **Drop the flattery** ("I know you can do this") — it feeds false confidence.
4. **Verify the intermediate steps**, especially the final conclusion. CoT reduces errors; it does not remove them.
5. If you need auditable reasons, or you do not want the reasoning copied, consider **outputting only the conclusion.**

## CoT is not free

"Step by step" is not zero-cost. The extra intermediate text means:

* **More expensive**: every step burns tokens, and per-token APIs show it on the bill.
* **Slower**: longer output, higher latency.
* **More noise**: when you only want the answer, a trail of reasoning is noise.

The practical move is to **turn it on and off per scenario**: interactive, fast, cheap → off; justified, auditable, verifiable → on. Treat CoT as a parameter, not a default.

## What about "reasoning models"

You may have heard of "reasoning models" — the ones that shine at math, logic, and competitive programming. Roughly, many of them have chain-of-thought built in: even if you do not say "think step by step", they reason through a few rounds internally before answering.

Two implications for you:

* **A reasoning model saves you the trouble of writing CoT yourself** — but the cost is speed and spend, because it genuinely does extra work.
* **You can still ask for CoT explicitly**, especially when you want the reasoning written out for a human to read. That is separate from whether the model does it internally.

The deciding question remains the task itself: if it needs computing, add it; if it only needs to re-present known information, skip it. `models-04-reasoning` goes deeper.

## Next step

CoT makes the model think deeper, which also makes it easier to mislead — especially when someone else's instructions are hiding in the input. Next we turn from "how to write well" to "how to write so it holds up": system prompts, style guides, and injection-resistant robustness in `prompt-04-robustness`.

#### Q: Why does asking the model to 'think step by step' improve accuracy on multi-step reasoning tasks?

* It makes the model call additional compute resources

* The extra intermediate text forces it to establish intermediate facts first, then carry them to the conclusion

* It makes the model copy a standard answer from its training data

* It increases the number of random draws

> 💡 Generation is word-by-word. Step-by-step reasoning forces the model to first write down intermediate facts it can build on (like 'capacity is only 48'), then predict the conclusion from those facts, drastically cutting the chance of sliding into a statistical first instinct.
