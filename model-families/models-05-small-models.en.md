# Distillation and Small Models: Fitting an Elephant into a Phone

> 📅 2026-08-04 · Core Concepts
> How do small models 'apprentice to big ones'? What distillation is, why small models are everywhere (on-device, local, cheap), and how it connects to fine-tuning, LoRA, and quantization.

---

In 2023 everyone was bragging about "whose model is biggest." By 2025 the wind turned: people began competing over "who can cram a very strong model into a phone, run it on a laptop, and still make it blazing fast." Behind it all is an old-school knack — **distillation**.

Last post met the "thinks long" reasoning models (`models-04-reasoning`). This one looks at the power of "small": why that little model on your desk has secretly mastered seven-tenths of a giant's skill, and how it relates to fine-tuning, LoRA, and quantization.

## Distillation: a small model "apprenticed to a big one"

Imagine you can't possibly read a million books, but you find a PhD who has — and you spend your days memorizing the PhD's notes. In the end you know maybe 70-80% of what the PhD does. **Distillation is exactly this.**

The method: take an already-strong "teacher" model, feed it a pile of questions, collect its answers (possibly including its reasoning traces); then use those "teacher answers" as training data to teach a small "student" model to imitate them.

| Role | What it does |
|---|---|
| **Teacher model** | Big and strong; produces "good answers" that serve as teaching material |
| **Student model** | Small and fast; learns the teacher's answers and reasoning patterns |
| **Teaching material** | The teacher's outputs on many inputs (reasoning included) |

The whole flow breaks into four steps:

#### Prepare questions

Gather a large, varied question set covering what you want the small model to learn.

#### Teacher answers

Have the big model answer each; collect high-quality answers, ideally with reasoning.

#### Student learns

Train the small model to imitate, using the pairs of question and teacher answer.

#### Verify convergence

Confirm on held-out questions that the small model really learned, not memorized.

Key idea: **the small model learns not "the world" but "how the teacher answers."** It copies behavior and output distribution — it need not reproduce the teacher's parameter count. That is why distillation makes small models "small but strong": they stand on the teacher's shoulders.

## Why small models are everywhere

Distillation exploded because "a model that fits in your pocket" unlocks three big things:

* **On-device**: phones, laptops, and wearables can run a model directly — data stays local, works offline.
* **Local**: your own machine can run it (`local-01-ollama` is the tool in this space) — nothing uploaded to the cloud.
* **Cheap**: inference cost is low enough to call at scale; a blessing for budget-sensitive startups.

| Advantage | Why |
|---|---|
| **Privacy** | Data stays on your device |
| **Latency** | No network round trips; instant responses |
| **Cost** | Small compute needs, low unit price |
| **Reach** | Everyone can run it; AI is no longer cloud-only |

## Running a small model locally is easy

Modern tools turn this into a couple of commands. With Ollama, for instance (`local-01-ollama` has a full tutorial):

```bash&#x20;—&#x20;run&#x20;an&#x20;8B&#x20;small&#x20;model&#x20;locally
# after installing, one command downloads and runs it
ollama run qwen2.5:7b

# then you can ask it right in the terminal
>>> Explain distillation in one sentence
# Distillation trains a small model on a big model's answers,
# letting the small model inherit the big model's knowledge.
```

No cloud, no fees, no handing your data to anyone — that is the most charming thing about small models.

## How small models relate to fine-tuning and LoRA

People often mix up "distillation, fine-tuning, and LoRA." They're really **three different operations**, and they can work together:

* **Fine-tuning (`train-02-finetuning`)**: on top of an existing model, keep training with **your data** to teach your domain or style.
* **LoRA (`finetune-02-lora`)**: a "cheap fine-tuning" technique that adjusts only a small batch of extra parameters to steer the model toward a task.
* **Distillation**: **compresses and transfers "big-model ability"** into a small model.

In practice they're usually **run in relay**: distill a small model first, then fine-tune it on your data, so it's both small and knows you — the standard recipe for local AI applications. To squeeze further, **quantization (`local-06-quantization-tiers`)** lowers parameter precision and slims the file further.

## Famous distillation examples

Distillation isn't just theory; the industry already has textbook cases:

* **The Llama derivative sea**: as `models-02-families-tour` noted, Llama has been fine-tuned and distilled into countless derivatives, forming the foundation of the open-source community.
* **DeepSeek-R1 distillation**: DeepSeek publicly distilled its own reasoning model (the `models-04-reasoning` kind) into Qwen and Llama students from 1.5B to 70B, teaching small models to "think before answering."
* **Every lab's "mini / small" line**: almost every closed vendor ships a small version, using distillation to press flagship power into a cheap, small body.

What these share: **distillation frees "top capability" from being locked to the biggest, priciest model.** It is the pipeline that passes ability down.

## See it with the comparison slider

The same problem, two very different states of mind. Imagine sliding between two options:

#### 對照 / Comparison

On the left is the wise one who thinks a hard problem through; on the right, the worker who gives you an answer now. **The point isn't that one side is better — it's which side your current problem needs.**

## Practical tips for picking a small model

Before choosing one for your product, these habits save you from wasted effort:

* **Define the task, then the size**: is your job "summarization" or "domain Q\&A"? The former may need only 3B; the latter often wants 8B or more.
* **Look past parameters to distillation quality**: two 7B models are not equal; one distilled from a good teacher beats a random shrink. Read the vendor notes and benchmarks (`models-06-evaluation`).
* **Quantize before deciding**: with the same weights, 4-bit quantization can halve memory usage (`local-06-quantization-tiers`). Small model plus quantization is often what truly counts as "small enough."
* **Run it yourself**: you can run it locally (`local-01-ollama`) — test with your real data rather than trusting spec sheets.

## But small models sound too good — any catch

Of course there's a catch. Let's honestly list a few:

* **A ceiling exists**: however you distill, a small model's top usually sits below a big model's. It suits "good enough" tasks, not "strongest" tasks.
* **Reasoning and hard problems no**: last stop's reasoning models (`models-04-reasoning`) "think long before answering" — small models often **can't think that hard**.
* **Data quality is everything**: if the teacher's material is dirty, the student's answers are dirty. Distillation's quality ceiling is the teacher's quality ceiling.
* **Students inherit the teacher's bad habits**: biases and hallucination tendencies get copied along with the good parts.

So the right posture is: **big models are the "wise one"; small models are "the worker avatar."** Call the wise one when you need depth; call the worker when you need speed and privacy. A company doesn't put everyone in the boardroom — for most daily chores, a fast cheap worker is just right.

And a common misconception: many assume "small = open-source, big = closed." Actually distillation runs on **both tracks**: the open community distills Llama and Qwen, and closed vendors distill their own mini versions too — it's purely a "capability compression" technique, unrelated to openness (`models-01-open-vs-closed`).

## One-sentence summary

A small model isn't a "dumber model," it's a "**concentrated model compressed from a teacher via distillation**." It trades a bit of breadth for three boons: it fits in a phone, runs locally, and is cheap enough to use at scale — the cost is that it can't truly fight the hardest problems and can't filter out the teacher's bad habits.

But small models aren't "small = all good" either; there's one more trick that makes them even smaller: **quantization**, lowering parameter precision to slim the file down — the hands-on territory of `local-06-quantization-tiers`.

> Remember one sentence: distillation is not a spell that "makes a small model strong"; it's "using the teacher's answers as teaching material and having the student imitate." It copies behavior, not parameter count. To stack on top, fine-tune and LoRA make a small model small, fast, and attuned to you.

## How this ties back to the whole series

From `models-01-open-vs-closed` (the open-vs-closed debate), to `models-02-families-tour` (the family tour), `models-03-multimodal` (multimodality), `models-04-reasoning` (reasoning), and today's "small and beautiful" — all along we've been asking "what is a model, and what can it do?"

But "what it can do" hasn't answered a more basic question: **how do we know a model is actually strong?** Next post quantifies "strong" into numbers using benchmarks and leaderboards — `models-06-evaluation`.

#### Q: Which description most correctly explains 'distillation'?

* Training a small model to imitate the answers of a big model

* Copying a small model weights into a big model

* Retraining a big model on lots of private data

* Merging all model parameters into one

> 💡 Distillation is defined by 'learning from the teacher output': a big model produces good answers, and a small model uses those answers as training data to imitate. It copies behavior and output distribution, not parameter count.
