# The Training Corpus: What the Model Ate

> 📅 2026-08-04 · Deep Dive
> A model learns exactly what it's trained on. Every crawled page, book, and snippet of code decides what it knows — and plants its biases and the copyright debate.

---

Pre-training at scale (`train-01`) runs on massive **training data**. And what a model "knows, can do, and is biased toward" is almost entirely decided by "what data it ate."

This post is about the data itself: what the model actually eats, how it's "washed", and why "garbage in, garbage out" is especially honest in AI.

## One line

> A model's "knowledge" = the statistical footprint of its training data. **No data, no capability.**

A model that never saw Chinese has zero Chinese ability; a model that only saw code can't write poetry. The ceiling of capability is drawn by the data first.

## Data composition: a dish of what

Modern large-model pre-training data looks roughly like this (ratios vary by model):

| Source | Examples | Provides |
|---|---|---|
| **Crawled web / Common Crawl** | pages from millions of sites | lots of uneven language & commonsense |
| **Books** | public domain, licensed | refined, long-form, narrative |
| **Academic papers / Wikipedia** | arXiv, Wikipedia | knowledge density, structure |
| **Code** | GitHub (licensed) | programming ability |
| **Conversations** | forums, Q\&A | interactivity, tone |
| **Synthetic data** | produced by other models | volume, guidance |

**Common Crawl** (a public dataset that periodically crawls the whole web) is often the biggest slice — but it's uneven quality, so it requires "cleaning."

## Cleaning: separating the scraps

Raw crawled pages can't be trained on directly: there are ads, garbage, duplicates, malicious content. So pre-training includes a whole "cooking" pipeline:

* **Filtering**: strip porn, violence, low quality, machine-generated junk pages.
* **Dedup**: the web is full of copy-paste; that "fattens" the model without helping it learn.
* **Quality screening**: use proxies like "how similar to Wikipedia prose is this" to keep more formal passages.
* **PII removal**: delete IDs, phone numbers, etc., so the model doesn't memorize them.

#### Raw crawl

Common Crawl and others pull millions of pages.

#### Dedup

Remove near-duplicate blocks across pages/sites (URL dedup / MinHash near-dedup).

#### Quality filter

Keyword-stuffed ads and low-quality content get dropped.

#### Curated blend

Mix in books, Wikipedia, code to hit a desired "recipe".

#### Into pre-training

The clean token stream feeds the train-01 pipeline.

> The spirit: pre-training is the most honest case of "garbage in, garbage out". Data quality and representativeness directly set model quality and bias. Cleaning isn't an optional nice-to-have — it's the necessary step to avoid trouble.

## The trap in the data: toxicity and bias

Why do models have biases (`ethics-01-bias`)? Because **the internet has biases.**

* The web describes women and men differently; the model learns to implicitly reproduce that asymmetry.
* Some groups appear too rarely in the data, or only as stereotypes; the model magnifies it.
* Filtering "too clean" (deleting all controversial topics) leaves the model ignorant or overly evasive on certain subjects.

This isn't the model's "original sin" — it's the **gravity of the data**. Understanding it is why "de-biasing" can't just be a matter of deleting a few lines.

## The copyright war: is this legal

The hottest topic in training data: **can a model eat others' books, articles, and code without permission?**

Still fiercely debated, with positions:

| Stance | Claim |
|---|---|
| **Rights-holders / publishers** | Training on works without authorization = infringement; should pay or ask (several active lawsuits) |
| **Model companies** | Training is "transformative use," covered by fair use; or they used licensed data |
| **Open-source community** | Emphasize "the model doesn't reproduce the text verbatim"; distinguish learning vs copying |

> This is a legal fight still in progress, not yet decided. The useful takeaway for readers: when you choose "which model", its data licensing is part of its cost and its stance. Some spend heavily on licensed data; others lean heavily on crawling. That shapes quality, price, and controversy risk.

## Data recipe → model strengths

Different data recipes raise different model leanings:

* Heavy code → strong at coding, but possibly stiff prose.
* Balanced multilingual → stable across languages, but maybe not top-tier in any single one.
* High academic/English share → high knowledge density, but weak on small languages.

So "choosing a model" isn't just comparing parameters — it's **comparing data recipes** — which is part of why, as `models-02-families-tour` will discuss, labs publish their data strategy as a selling point.

## One-sentence close

A model's capability, bias, and even legal risk are all written in "what it ate." **Want to predict what a model will be like? Look at its data recipe first.**

The remaining puzzle is "why does scaling up model and data together suddenly make models much stronger" — scaling laws, `train-05-scaling-laws`.

#### Q: Why can't 'a debiasing program at the backend' fully fix a model's reproduction of internet bias against a group?

* Because the bias comes from the statistics of the training data itself, not a code bug

* Because model companies don’t want to fix it

* Because a debiasing program would slow the model down

* Because bias only exists in Chinese models

> 💡 Bias is the 'gravity' of data statistics: the asymmetries a model learns from data get reproduced. Backend filters can soften the surface, but the root is data composition — you must intervene at data collection and cleaning.
