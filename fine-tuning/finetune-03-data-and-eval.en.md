# Data & Evaluation: What Decides a Fine-tune

> 📅 2026-08-04 · Deep Dive
> LoRA makes fine-tuning cheap, but data and evaluation decide the outcome. How to prepare data, split validation, prove it's really better, and dodge overfitting and forgetting.

---

`finetune-02-lora` showed you how to make fine-tuning cheap. Cheap is a condition, not a result — run dirty data through even the smoothest tooling and you still get a dirty model. This post covers the real deciding factors: **data preparation** and **evaluation.** Clean data and honest measurement tell you whether the fine-tune actually got better, instead of just feeling better.

## What good data looks like

Back to the spirit from `train-02-finetuning`: fine-tuning pushes a model from "general" toward "specialized" using your data. So the data is **demonstration** — what you want the model to reply like is what each example should look like. Three keys:

**1. Quality over quantity.** One crisp example that matches your target format beats ten vague ones. Every row should be a response you'd happily copy verbatim.

**2. Deduplicate.** Repeated rows make the model "memorize" those few harder, crowding out learning other patterns. Run a cheap approximate dedup (n-gram or embedding similarity) and drop near-identical examples.

**3. Consistent format.** The easiest thing to overlook: your input/output format must match what you'll *actually use in production*. If you ship with a system prompt, the training data should look like that too. If you want JSON replies, every demonstration must be valid JSON.

### A "good demonstration" vs. a "bad one"

The same task, difference at a glance. Say you're teaching a support model to answer in bullets:

| Bad demonstration | Good demonstration |
|---|---|
| Input: "Return policy?"<br />Output: "Sorry, you can return items. Check the site for details." | Input: "Return policy?"<br />Output: "Sorry about the hassle! On returns:<br />• Requests within 7 days of delivery<br />• Original packaging required<br />• Refunds land in 3–5 business days<br />Anything else I can check?" |

The bad one is loose, structureless, and punts with "check the site" — it teaches the model to be lazy. The good one teaches "apologize first, bullets, closing question" all at once. **Every row is teaching material, and bad material gets learned.**

> Rule of thumb: fine-tuning data is demonstration, not a fact database. Every row teaches "given this input, reply like this" — so inputs should mirror reality, outputs should be worth copying, and formats should match the production environment.

## The train/validation split: stop being optimistic

The classic fine-tuning mistake: you're thrilled after training, then discover in production it only memorized the training set. The cure is old-school: **hold out a validation set that never sees training.**

* **Train set**: the ~90% you actually run fine-tuning on.
* **Validation set**: the ~10% kept "hidden," never used in training, measured at the end to judge generalization.

If train accuracy keeps climbing while validation stalls or dips, that's the classic overfitting signal — the model has started to *memorize* instead of *learn*.

#### Split before you start

Cut the data 90/10 train/validation, and the 10% never enters training afterwards.

#### Watch the curves

Train keeps climbing while validation stalls or dips → overfitting signal, prepare early stop.

#### Score only on validation

Once the model is chosen, give it a final score on the never-seen validation set.

## Measuring "actually better"

Evaluate a fine-tune on two tracks: **behavior tests** and **regression tests.**

**Behavior tests: did the target behavior appear?** Build a **golden set** — twenty to fifty prompts that really occur in your product, with human-marked "ideal replies." Run it before and after fine-tuning, and compare:

* Is the format right? (opening, closing, fields, tone)
* Is the content right? (facts, keywords, structure)
* How far from the ideal reply?

**Regression tests: did existing abilities survive?** Fine-tuning specializes, and specialization often brings *forgetting* — the model may start forcing out-of-domain questions into your format. So keep a "general ability" test set (a few summarization, translation, and common-sense items) and confirm you didn't wreck what was already there.

### What counts as passing

Behavior-test scores don't need to be perfect, but they must be **repeatable and comparable**:

| Check | A passing look |
|---|---|
| Format compliance | Every reply matches the target structure (all JSON, all bulleted) |
| Content hits | Keyword/fact hit rate clears the threshold you set |
| Consistent tone | Spot-checked by humans; no longer looks like the default model |
| Regression clear | General-ability scores haven't visibly dropped |

Don't invent the bar out of thin air — **measure the "untuned" baseline first**, then compare every round against it. Progress and regression both get judged against the baseline, so you can't fool yourself.

```evaluation&#x20;sketch
# Split: train vs validation (validation never trains)
train, val = split(dataset, ratio=0.9)

# Behavior tests: fixed golden prompt set
for prompt, ideal in golden_set:
  out = model.generate(prompt)
  score += check_format(out, ideal)   # format + content

# Regression tests: confirm old abilities survived
assert general_ability(model, baseline) >= threshold
```

## Overfitting and catastrophic forgetting: two enemies to watch

* **Overfitting**: memorizing the training data so validation degrades. Defense: little and clean data, sane rank (see `finetune-02-lora`), a held-out validation set, and early stopping.
* **Forgetting (catastrophic forgetting)**: learning the new wipes out the old. Defense: mix a small slice of "general ability" examples into the training data, and keep an eye on it with the regression set.

One line: **"learns new, remembers well, keeps old" pull against each other**, and evaluation scores all three at once.

> Both enemies share one weapon: keep a set of data that never entered training, and let it serve as the yardstick for both validation and regression. Overfitting shows up in validation; forgetting shows up in regression. You need both readings.

## A minimal evaluation flow

You don't need academic rigor. This is enough in practice:

1. **Measure the baseline first**: run the *untuned* base model on the golden set and record the score.
2. **Measure again after**: same golden set, same regression set.
3. **Compare three things**: did the target behavior improve? Did general ability regress? Is that regression acceptable?
4. Not satisfied? Fix data or rank and run another loop — evaluation isn't a one-time final exam, it's the dashboard for every round of tuning.

One thing people forget: **keep the evaluation data cleanly separate from the training data.** If golden prompts leak into training, your pretty scores are an illusion. It's the same discipline as the validation split — and it means the eval prompts themselves shouldn't be copied out of the training set.

This flow is the same shape as the bigger question of "how strong is a model," which `models-06-evaluation` will scale up to the benchmark level. For now, make it run on a single fine-tuning task.

## Closing: data and evaluation are the real line

LoRA lowers the barrier; what actually decides whether this tuned model ships is whether the data is clean and the evaluation is honest. Adopt the habit — split validation first, measure a baseline first, compare every round — and you won't be fooled by training curves, and you won't ship a model that merely memorized its homework.

#### Q: After fine-tuning, training accuracy is high, but the held-out validation set starts dropping. Most likely cause?

* The validation set is too small to be accurate

* The model is overfitting — memorizing training data instead of generalizing

* Rank is too low and it under-learned

* The data format is inconsistent

> 💡 Train loss keeps falling while validation stalls or drops is the classic overfitting signal: the model memorizes rather than generalizes. The fixes are little and clean data, sane rank, a held-out validation set, and early stopping.
