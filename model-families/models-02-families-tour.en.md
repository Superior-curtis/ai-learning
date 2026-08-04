# A Tour of the Major Model Families

> 📅 2026-08-04 · Core Concepts
> GPT, Claude, Llama, Gemini, Mistral, Qwen — six families' origins, openness, and strengths, in one tour.

---

The last post split models into "open vs. closed" (`models-01-open-vs-closed`). Now let's visit the "stores": who made these models, and what each excels at.

First, the caveat: **this post is a snapshot, not eternal truth.** The AI world can swap its champion in six months. Read it by treating "families" as mental categories and "specific models" as data you update frequently.

## The six families at a glance

| Family | Company | Openness | One-line impression |
|---|---|---|---|
| **GPT** | OpenAI | Closed (API) | The pioneer; biggest ecosystem, most tools |
| **Claude** | Anthropic | Closed (API) | Safety & alignment focus; strong long-text & writing |
| **Gemini** | Google | Mostly closed | Deep Google integration; multimodal from day one |
| **Llama** | Meta | Open-weight | The foundation of the open-source community |
| **Mistral** | Mistral AI (France) | Open + closed | Efficient small models, European style |
| **Qwen** | Alibaba | Open-weight | Very strong Chinese; all-rounder |

Also frequently mentioned: **DeepSeek** (open-weight, extremely low cost), **Grok** (xAI, integrated with X), **Gemma** (Google's open small model). The point isn't memorizing a list — it's understanding that each has its strengths.

## GPT: the pioneer, king of the ecosystem

* **Origin**: ChatGPT in late 2022 brought LLMs to the masses; "GPT" nearly became synonymous with "AI assistant."
* **Strengths**: the most mature ecosystem — plugins, function calling, multimodal, the Codex agent; most users, richest feedback data.
* **Character**: comprehensive, steady, the "default option" that works out of the box.

## Claude: sells "alignment"

* **Origin**: Anthropic, founded by former OpenAI leaders, with safety alignment as its brand core (see `train-03-rlhf`).
* **Strengths**: famous for polish on "helpful, honest, harmless"; long-text comprehension, writing, and code are all top tier.
* **Character**: cautious, measured speech, fits workflows that "need it to do the job properly." Claude Code is its terminal agent (the star of the whole `claude-code` series).

## Gemini: Google's integration machine

* **Origin**: Google DeepMind; from day one emphasized **multimodality** (native image, video, audio understanding).
* **Strengths**: deeply bound to Google products (Search, Workspace, Android); multimodal is a native advantage.
* **Character**: you "meet it inside Google's family," rather than deliberately choosing it.

## Llama: the foundation of open source

* **Origin**: Meta released the weights, making self-hosting possible.
* **Strengths**: not "strongest," but "most modified" — the community fine-tunes, distills, and localizes it into countless derivatives.
* **Character**: it's more the "master mold for Lego bricks" than a finished product.

> Remember the pattern: closed families (GPT / Claude / Gemini) compete on "product completeness"; open families (Llama / Qwen / Mistral) compete on "malleability." One lets you worry less; the other lets you do more.

## Mistral and Qwen: two "open" routes

* **Mistral** (France): famous for "small model, high efficiency" — reaching near-large-model performance with fewer parameters; popular in Europe and self-hosted circles.
* **Qwen** (Alibaba): especially strong in Chinese; sizes from tiny to flagship; a mainstay of the Chinese open-source scene.

## How to think about "who's strongest"

Leaderboards (`models-06-evaluation`) change daily. Three more durable ways to judge:

1. **Match your task**: long-form writing? Claude is strong. Code? GPT/Claude both strong. Chinese? Qwen is solid. Multimodal? Gemini is native.
2. **Test, don't trust marketing**: build your own eval set (dozens of your real tasks), run each family, compare — more accurate than any leaderboard.
3. **Accept taste**: model "personality" affects your workflow (cautious vs. bold); pick the one you vibe with.

```A&#x20;starting&#x20;point&#x20;for&#x20;running&#x20;your&#x20;own&#x20;comparison
# Don't trust leaderboards alone — run your real 20 tasks across families
tasks = ["Explain calculus to a high-schooler in three sentences",
       "Rewrite this contract in plainer language",
       "Write a Python function that cleans data"]

for provider in ["openai", "anthropic", "google"]:
  for task in tasks:
      # call each vendor's SDK, collect the reply
      ...
# Then (the key) grade by hand: which one fits YOUR needs
```

## One-sentence summary

There's no "best model" — only "the model that fits your task and situation." Treat families as a mental map and models as changing data, and you'll never be held hostage by a leaderboard.

Next, **multimodality** — when models no longer read only text but also see images and hear audio, what the world becomes — `models-03-multimodal`.

#### Q: Why is comparing models on your own 20 tasks more reliable than looking at a leaderboard?

* Because leaderboards are often manipulated by marketing

* Because leaderboards test generic tasks; your real tasks are the real test for the model

* Because leaderboards don’t update

* Because leaderboards only test English

> 💡 Leaderboards evaluate 'generic benchmarks' aligned to average contexts. Your tasks (language, format, style) are the only real exam — your own 20-question test is closer to reality than any third-party list.
