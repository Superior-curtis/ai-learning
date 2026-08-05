# Where Bias Comes From: A Model Is the Shadow of Its Data

> 📅 2026-08-04 · Core Concepts
> Models reproduce society's biases not because they hold opinions but because all they learned is the statistical shape of their training data. This post traces the four paths bias takes into a model, explains why there is no clean fix, and what teams can actually do.

---

Ask an assistant to filter engineering resumes, and a model under the hood will quietly rank the ones with women's names lower. It is not "anti-women". It simply read a lot of "engineering context + male pronoun" pairings online, and it treats that correlation as the most likely next step.

Same story in image generation: type "CEO" and you get a row of men in suits; type "nurse" and you get smiling women. The model is not being deliberately stereotypical — it is faithfully replaying the statistics of the data you fed it.

This is not a flaw; it is the nature of the machine. A next-token predictor can only reproduce the patterns that recur in its data. **Bias is not a question of what a model thinks; it is a question of what the data looks like.** This post breaks down the paths bias takes to get inside a model, why there is no clean fix, and what engineering teams can actually do.

## Bias is statistics, not a bug

A model has no values. It has a set of parameters that encode "in the text it saw, which words tend to follow which words." As `train-04-corpus` put it, a model's capabilities and tendencies are almost entirely decided by what it ate.

So when the web text pairs "doctor" with "he" and "nurse" with "she", the model learns the pairing — not because it endorses it, but because "she" is statistically the more likely next word. **To a model, a bias is the same kind of thing as "cat" following "meow": a recurring pattern in the data.** The only difference is that a human reading the pattern pauses to ask whether it should exist; the model does not.

## Bias and hallucination: two symptoms of one engine

"Bias" and "hallucination" (`llmcore-05-hallucination`) sound like separate problems, but they are two symptoms of the same engine. Hallucination is a model that cannot tell the difference between "most fluent" and "most true"; bias is a model that cannot tell the difference between "most common" and "most fair".

Seeing them together has a practical payoff: **you stop hoping that editing the output can cure either one, because both live in the statistics behind the model, not in the sentence in front of you.** For hallucination you verify and cite; for bias you make it visible, measurable, and manageable — which is exactly what the rest of this post is about.

## Four paths bias flows in

Bias does not enter at a single step. At least four doors are open:

```text
statistics of training data → model parameters → product prompts → output users see
↑                              ↑
annotator values               interface and feedback loops
```

| Path | What it is | Example |
|---|---|---|
| **Training data** | Web text already carries society's biases | engineering contexts read male; one group appears only in negative stories |
| **Labeling and feedback** | Annotator judgment, guidelines, sampling | RLHF preference data writes annotator values into the model |
| **Product usage** | How prompts are written, how the UI is shaped, who the users are | default prompts surface stereotyped answers; autocomplete amplifies them |
| **Measurement and deployment** | What you evaluate on, who the test set covers | eval sets that miss dialects or age groups make those groups invisible |

The four doors are not independent — they stack. A data-level bias gets amplified by annotation, then turned by the product's prompts into the daily experience of real users.

## Over-representation and under-representation

The "data gravity" from `train-04-corpus` is especially visible here:

* **Over-represented groups** become the model's default world: responses to them are the most fluent, the most accurate, the most "default".
* **Under-represented groups** sit at the edge of the data: the model knows little about them and answers wrong more often. Worse, if the scraps of data that do exist are mostly stereotypes, the model treats the stereotype as fact and amplifies it.

Take a corpus dominated by the English-speaking web: the model's output quality for English users is far higher than for speakers of small languages — and if the few texts that do mention those small languages happen to be travel brochures or disaster coverage, the model's picture of that whole region gets locked inside those few genres.

So "more data is better" does not hold for bias — **distribution matters more than volume.** A corpus written entirely by one group and a balanced corpus produce completely different models.

## The evaluation itself can be biased

The last easy-to-miss entry point: **the very benchmark we use to judge "is this model good" can be biased too.** If the test set treats one group's phrasing and cultural background as the "correct answer", then a model doing well for that group is just "normal" — while its low scores for other people get hidden by the average.

A classic trap is looking only at "overall accuracy" — it is a mean over all groups, and one group's high scores can perfectly mask another group's low ones. The lesson of `train-04-corpus` applies here in reverse: **the distribution of data decides what a model can do, and also what an eval can and cannot see.** So bias-focused evals are not "add a few more test cases"; they make "score each group separately" the default way of evaluating.

## Concrete cases

Bring it down to three phenomena you have probably seen:

* **Hiring assistant**: the model learns "the decisions that were made historically", and historical decisions already encode past discrimination. It does not invent discrimination; it inherits it.
* **Image generation**: "CEO" and "judge" skew white and male; "nurse" and "teacher" skew female — because that is how those occupations pair up in the visual data.
* **Translation**: when rendering a language without grammatical gender into English, the model has to pick one, and it picks the more common one — engineers become "he", nurses become "she".

The common thread: **the model is faithfully replaying correlations that were already in the input data.** The problem is not that it learned wrong; it is that the data itself is lopsided. Blaming the model is like blaming the mirror for reflecting your face.

## Bias reinforces itself: the feedback loop

The harder part is that bias is not "one-time" — it self-reinforces. The model gives a stereotyped answer, the user upvotes or keeps using it, the interaction is recorded as feedback, and the model becomes more likely to give the same answer next time.

The clearest example is search and autocomplete. What the user sees is a suggestion "raised on the bias of past users", and clicking those suggestions feeds the next round of suggestions. **Bias grows out of the data and gets fed back in through usage.** This is also why post-launch monitoring is not an extra task for the safety team but one of the main battlefields of bias work — `ethics-02-responsible-deployment` turns this loop into an actual process.

## Why "debiasing" has no clean fix

If bias is just a data problem, why not clean the data and be done? No such clean fix exists, for four reasons:

1. **Bias is entangled with capability.** Every correlation a model learns is a mix of useful and harmful. Pull out a skewed correlation and you often damage normal abilities with it — removing "doctor pairs with he" may weaken the model's grasp of "doctor" itself.
2. **There is no neutral baseline.** "Neutral" is itself a choice. Which group should be the default? Is fairness "treat everyone the same" or "compensate for difference"? People with different stakes answer differently.
3. **Fix one, pop up another.** Repairing the skew against group A often introduces new skew against group B — and models optimized for average accuracy are already bad for minorities.
4. **Alignment and filtering only touch the surface.** `llmcore-05-hallucination` showed that model output is probabilistic; likewise, any filter only lowers the probability of a biased answer. It never reaches zero, and it never changes the underlying statistics.

In one sentence: **bias is not a screw you can remove; it is part of the model.** So debiasing is not a single surgery — it is ongoing engineering.

## What teams can actually do

Not "delete the bias", but "make bias visible, measurable, and manageable":

* **Audit the data**: inventory the representation of the training corpus (`train-04-corpus`) — who is in it, who is not — and document it openly.
* **Bias-focused evals**: build eval sets that probe for bias — run the same question with different names, dialects, and genders, and compare.
* **Red-teaming**: deliberately test with prompts likely to trigger stereotypes (the testing spirit of `security-06-guardrails`), and turn every finding into a permanent test case.
* **Monitor after launch**: track per-group success-rate gaps instead of only the average.
* **Process and diversity**: clear annotation guidelines, a diverse evaluation team, documented decisions — much bias slips in where nobody is watching.

Here is the seed of a bias-focused eval: one question, several names, compare the outputs.

```python
# a simple bias probe: same question, different names
def probe_bias(model, question):
  names = ["Wei Chen", "Mei Chen", "Jun Wang", "Li Wang"]
  for name in names:
      prompt = f"{question} My name is {name}."
      reply = model.complete(prompt)
      print(name, "->", reply[:40])
```

One probe run tells you little; what matters is **running the same probe before and after launch** — that is how drift in bias becomes visible:

#### 對照 / Comparison

> Keep this one line: bias is not a question of what a model thinks; it is a question of what the data looks like — the model is just the shadow of its data. To change the shadow you change the light source: data, evals, prompts, and process. No debiasing script replaces that work.

## Next

Bias is the first ethics question of what a model brings into the world. Once you know where it comes from, the next question is: even knowing that, how do you ship a feature responsibly? From risk assessment, guardrails, and monitoring to sunsetting a feature — that is a whole engineering habit. `ethics-02-responsible-deployment`.

#### Q: Why is removing biased examples from the training data not a clean fix for bias?

* Because a model is a statistical system where bias and capability are entangled; removing one correlation can damage normal abilities

* Because bias is really the model’s own value system, so editing data cannot help

* Because training is too fast for the data to be changed

* Because bias only exists in image models, not text models

> 💡 Bias is part of the model, not a removable screw: it shares the same statistics as capability, and there is no neutral baseline to fall back on. So debiasing is not a one-time surgery but an ongoing effort of data auditing, targeted evals, and post-launch monitoring.
