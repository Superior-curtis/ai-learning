# Tokens: Language Isn't Processed as Words

> 📅 2026-08-04 · Core Concepts
> Models don't read characters or words — they read tokens, little chunks of text. The same Chinese sentence can cost three times as many tokens as its English version.

---

The previous post said an LLM's job is "guess the next word". But there's a hidden question: **what does the model actually see as a "word"?**

You'd think it sees characters or words. In reality it sees units called **tokens**. Tokens are the most basic — and most overlooked — currency in the AI world, because **they determine the model's memory limit, its billing, and its speed.**

## What is a token

A token is a "chunk of text". The model doesn't know letters or words — it only knows tokens, produced by a piece of software called a **tokenizer** that splits input text into a sequence of tokens.

The split varies by language and is delightfully irregular:

```English:&#x20;common&#x20;words&#x20;are&#x20;usually&#x20;one&#x20;token&#x20;each
Hello, world!    →  [Hello] [,] [world] [!]        (4 tokens)
The quick brown fox →  [The] [quick] [brown] [fox]   (4 tokens)
unbelievable       →  [un] [believ] [able]           (3 tokens, split into word-parts)
```

* Common words: one token each (`the`, `is`, `hello`).
* Rare or compound words get split into several tokens (`un-believ-able`).

```Chinese:&#x20;each&#x20;character&#x20;is&#x20;often&#x20;one&#x20;or&#x20;more&#x20;tokens
你好，世界！   →  [你] [好] [，] [世] [界] [！]     (6 tokens)
人工智慧         →  [人工] [智慧]                     (2 tokens, common phrase)
颶風             →  1-2 tokens per character; rare characters cost more
```

Key takeaway: **Chinese is usually "more expensive" than English.** The same sentence can cost 2–3× as many tokens in Chinese. That's not discrimination — English simply has far more training data, so the tokenizer splits it more economically.

## Why split into tokens at all

Wouldn't storing whole sentences be simpler? The problem is vocabulary size:

* If the model used characters, English's 26 letters would be enough — but language isn't built from letters; meaningful building blocks like `un`, `believ`, `able` are.
* If it used whole words, English has ~500k common words — a huge dictionary, and you'd still never see new words (names, coined terms).
* **Tokens are the compromise**: a few tens of thousands of common "sub-word" blocks cover almost all text, with more meaning per unit than raw characters.

Think of English as reusable Lego bricks: `un` + `believ` + `able` gives you `unbelievable`, `believable`, `unbelievably`… without needing a new brick for every combination.

## BPE: how the tokenizer decides where to split

The most common algorithm is **BPE (Byte Pair Encoding)**. The idea is dead simple: **start with characters, merge the two symbols that appear next to each other most often, and repeat until your budget runs out.**

1. Split all text into single characters.
2. Count which two characters appear adjacent most often.
3. Merge them into one new token.
4. Repeat until the token count reaches a target (say 50k).

```text
"low low low low"  →  [l] [o] [w] [l] [o] [w] …
most frequent pair l+o →  [lo] [w] [lo] [w] …
then lo+w             →  [low] [low] …
→  eventually "low" is 1 token
```

So a tokenizer's splits aren't "grammatically correct" — they're **"the most economical split, statistically"**. That's why the same word can be split differently in different contexts, and why new words or names often get shredded into pieces.

## Why tokens matter: three reasons

**1. They set the memory ceiling (context)**

The amount of text a model can see at once is measured in tokens. An 8K context is 8K *tokens*, not 8K characters. This is especially felt by Chinese users: 8K tokens of context fits only ~3–4k characters of Chinese. See `llmcore-03-context`.

**2. They determine billing**

APIs are priced almost entirely in tokens: input tokens and output tokens, itemized. The same content in Chinese uses 2–3× the tokens of English, so **cost scales by 2–3× too.**

> Remember tokens' business meaning in one sentence: tokens are AI's billing unit — and Chinese costs more tokens than English, so the same Chinese sentence costs more.

**3. They determine speed**

During generation, the model outputs one token per "draw". Fewer output tokens → faster replies. That's why "answer concisely" is a genuine engineering lever.

## A practical rule of thumb

Mental math for token counts:

* **English**: ~1 token ≈ 0.75 words (100 words ≈ 133 tokens).
* **Chinese**: ~1 token ≈ 0.5–1 character (100 chars ≈ 100–200 tokens).

```Confirm&#x20;with&#x20;API&#x20;tooling&#x20;(conceptual)
# Every SDK reports token usage back to you
# Anthropic: response usage.input_tokens / output_tokens
# OpenAI:    response.usage.prompt_tokens / completion_tokens

response = client.messages.create(
  model="claude-sonnet-4-5",
  max_tokens=1000,
  messages=[{"role": "user", "content": "explain tokens in English"},
            {"role": "user", "content": "再解釋一次，用中文"}],
)
print(response.usage.input_tokens)   # you can see both languages side by side
```

## Next time you see "token"

You now know: tokens are the Lego bricks of text, the currency of model billing, and the ceiling on context and speed. They're one of the few concepts in AI that literally decide the number on your invoice.

Next, let's see how many tokens fit in the box: the context window (`llmcore-03-context`).

#### Q: Why does the same content usually cost more tokens in Chinese than English?

* Chinese is Unicode, so it takes more storage

* The tokenizer splits statistically, and Chinese common phrases are fewer, so more tokens per sentence

* Models favor English and deliberately under-count it

* The Chinese character set is larger, so each character is pricier

> 💡 A BPE-style tokenizer finds the statistically most economical split. English has enormous training data and rich word-part bricks, so it splits cheaply; Chinese is less often grouped into multi-character tokens, so the same content uses more tokens and costs more.
