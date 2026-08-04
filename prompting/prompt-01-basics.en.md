# Prompting Basics: Say What You Mean

> 📅 2026-08-04 · Getting Started
> Writing prompts is not magic. Once you accept that an LLM is just a next-word-guessing machine, your job becomes narrowing its choices toward the right answer — with clear instructions, context, and constraints.

---

A lot of people treat prompt writing like a secret craft — as if someone found a magic phrase that makes AI obey. There is no such thing.

Go back to the mental model from `llmcore-01-next-token`: **an LLM is a machine that guesses the next word.** You give it some text, and it keeps writing. So prompting is not "giving orders to an assistant who understands you". It is **steering a prediction machine with words.**

Once that clicks, everything falls into place. Your prompt tells the machine: here is the situation, and here is roughly the kind of "next sentence" I want to hear. This post is about saying that clearly.

## Why clarity matters

Imagine walking into a restaurant and telling the server: "Bring me food." They would freeze — Chinese or Western? Spicy or not? Budget? How many people?

But if you say: "Two bowls of the signature beef noodles, one with no cilantro, one extra spicy, to go." — no guessing required.

An LLM is the same. It is a server eager to speak. If you are vague, it guesses — and what it guesses is usually the most generic, safest answer. That is rarely what you wanted.

| What you give it | How it responds |
|---|---|
| Just a question | A generic, safe answer |
| Context and constraints | Converges toward what you actually want |
| An explicit output format | Something you can use directly |

The good news: you do not need to be technical to write good prompts. You need to **finish your sentences.**

## The three ingredients

Almost every good prompt breaks into three parts. You do not always need all three, but when you add the missing ones, the difference is immediate.

**One: a clear instruction.**

This is the only strictly required part — what you want the model to do. State it as one clear verb.

* Vague: "I want you to improve this text."
* Clear: "Rewrite this text as a formal letter. Do not use contractions."

**Two: context.**

This is the "why" and the "in what situation". It decides which drawer of knowledge the model reaches into. The same sentence gets a very different answer depending on context.

* "This is for a weekly team report" → professional, terse.
* "This is a note to a friend" → relaxed, casual.

**Three: constraints.**

The fences: length, format, tone, forbidden moves. The more explicit the constraints, the less the model drifts — and the easier it is to verify the output.

## Good example vs bad example

Put the three ingredients together and the difference is visible in seconds.

#### 對照 / Comparison

Same task. The bad prompt forces the model to guess; the good one paves the road.

> The golden rule in one sentence: tell the model who it is, what situation it is in, and what "next sentence" you hope to hear. The more thinking you do up front, the less the model has to do.

## A reusable prompt template

You do not need to memorize a format. Use this skeleton:

```prompt-template
You are 【role】. (Context: here is my situation.)
Please 【describe the task clearly, starting with one verb】.

【Background: why, for whom, in what setting】
【Constraints: length / format / tone / what not to do】

—
Here is the text to work on:
【paste your content】
```

Swap out the 【brackets】 for your own words. Run it a few times and you will notice a pattern: **whenever you leave a bracket empty, the model starts guessing.** That is the exact spot to fill in.

## Putting "guess the next word" to work

Back to the mental model. Writing a prompt is really doing one thing: **narrowing the probability distribution.**

If you say nothing, the model's distribution over "what comes next" is flat, so the answer lands in the generic middle. As you add role, context, and constraints, you push probability away from the wrong options and toward what you want. You cannot control the random draw itself (`llmcore-04-sampling` covers that) — but you can control **what options are on the ticket.**

This is also why one-line prompts fail so often: they hand all the choice back to a machine that is in a hurry to speak.

## Three common myths

* "Longer prompts are better" → No. Length dilutes focus. Good prompts are short where it is easy and detailed where it matters.
* "Prompts must be very formal" → No. Plain, concrete language is usually easier for the model to follow.
* "Wording controls everything" → No. Prompts shrink the model's choice space, but they do not control the random draw underneath.

## Writing a prompt is a loop

Do not expect the first attempt to be perfect. Writing a prompt is more like tuning than drafting — and the iteration is almost mechanical:

1. **Run it once**, and look at where the model "guessed wrong".
2. **Find the missing bracket**: was it context, or a constraint?
3. **Add it, run again.**

| Common failure | What is missing | Fix |
|---|---|---|
| Answer too generic | context | Add "for whom, why, what setting" |
| Wrong tone | role | Add "you are a…" |
| Runs too long | constraint | Add a length limit |
| Messy format | output format | See `prompt-02-structure` |

Prompts drift, bloat, and need maintenance — treat them like small programs rather than one-off requests. Do not be afraid to edit; editing is half the craft.

## Next step

This post was about saying things clearly. But clarity is only half of it — **getting the model to reliably produce the role and format you want** is the topic of `prompt-02-structure`.

#### Q: Why does a one-line prompt often produce a generic, middle-of-the-road answer?

* The model dislikes short inputs

* With too little information the next-word distribution is too flat, so the model settles on the safest generic answer

* The model memory is too small to store long prompts

* Short prompts trigger the moderation filter

> 💡 An LLM predicts the next word from context. Less input means a wider range of plausible continuations, so the output falls toward the most common, safest option. Instructions, context, and constraints shrink that range.
