# Roles, Examples, and Output Formats: Structuring the Prompt

> 📅 2026-08-04 · Getting Started
> Saying what you want is only step one. For reliable output you need three structural tools: a system role, a few good examples, and an explicit output format.

---

`prompt-01-basics` covered saying what you mean. This post covers something just as important: **giving the model structure.**

Ask it to "make this text sound more professional" and you can either let it improvise, or tell it "you are a senior editor, write three short paragraphs, at most four lines each, and avoid adjectives." The second one is far more likely to give you what you want — and it **reproduces**. Today, tomorrow, on a different model, you get roughly the same result.

Stable and reproducible. That is why engineers fall for prompt engineering.

## Roles: tell it who it is

"You are a…" is the most common and most effective prompt opening there is. Why does it work? Back to `llmcore-01-next-token`: the model infers from context what kind of "next sentence" is most likely. Give it a role, and you are saying: here is the situation, now continue in the language that kind of person would use.

* "You are a senior editor" → the prose gets disciplined, fond of cutting filler.
* "You are a physics teacher with ten years of experience" → explanations start leaning on intuition and analogy.
* "You are a Python performance engineer" → suggestions start talking about profiling and complexity.

A role is not a spell. It helps the model **pick which language-style drawer to open.** Pick the right drawer and the tone, vocabulary, and priorities all line up.

At the API level the role usually lives in the `system` message; in a chat it is the first line. Same effect either way.

## Few-shot: demonstrate, don't describe

There is an old saying worth adopting: **one good example beats ten descriptions.**

Rather than telling the model "make the tone calm and professional", just give it two input-to-output pairs. It infers the rule on its own.

```few-shot
Rewrite the following emotional lines into calm product feedback, one item per line.

Original: "This app is total garbage, every update makes it worse, does the team even listen?!"
Rewrite: "User reports the recent update made operation more complex and suggests restoring the previous flow."

Original: "Finally, the new feature is here, waited forever, thumbs up!"
Rewrite: "User appreciates the newly added feature and looks forward to further improvements."

Original: "This bug is driving me insane, I have restarted a hundred times and it still crashes."
Rewrite:
```

Watch what the model learns from the examples: keep the objective facts, drop the emotion words, keep the tone neutral. You did not write ten rules — a couple of examples demonstrated the rules.

This is **few-shot prompting**. It shines on style-transfer tasks: translation, rewriting, summarization, customer-service tone. A few examples get you close; three to five is usually plenty, and more can just make the model imitate the noise in the examples.

## Output format: make it usable

The last item is the one people skip — and it matters most for productivity: **state the output format explicitly.**

* In chat: "list the points, one sentence each."
* In code: "output only JSON, no explanatory text."
* In documents: "use Markdown headings for the three sections."

Why does this work so well? Models are good at following formats because they were trained on enormous amounts of fill-in-the-blank, heavily formatted text. Specify a format and the model's attention shifts from "what should I write" to "how do I fill this shape" — and the output feeds straight into your next tool or script.

> An explicit output format is the step that turns a chat toy into a programmable tool. Telling the model what shape to return beats telling it to try harder.

## A quick before / after

Say you often need to turn meeting notes into a to-do list.

**Before:** "Please turn these meeting notes into a to-do list."

The model replies with a long, rambly, inconsistently formatted paragraph. You clean it up by hand.

**After:**

```structured-prompt
You are a project assistant. Turn the meeting notes below into a to-do list. Output only JSON:

[
{ "task": "what to do", "owner": "who", "due": "due date" }
]

Rules:
1. One task per item, under 20 characters.
2. Use the owner names mentioned in the notes; if none, use "unassigned".
3. If no due date, use "not scheduled".
4. Output only the JSON array, no preamble or closing line.

Meeting notes:
"...Ming says he will finish the login page UI by next week, Hua owns the backend API, and nobody has claimed the data pipeline yet..."
```

Same raw material. Before gives you a paragraph to clean up; After gives you a JSON you can feed straight into a tool. The difference is those three tools: **role, examples, format.**

## Why this is more stable

All three do the same thing: **shrink the model's choice space** (echoing `prompt-01-basics`). The role shrinks tone and vocabulary, the examples shrink approach and structure, the format shrinks the shape. Layer all three and there is almost no room left for the model to improvise — so the result is stable, reproducible, and easy to verify.

That matters later: `prompt-04-robustness`, about making prompts survive weird input, is built on the idea that you first tie the prompt down tightly.

## When the model ignores the format

Every once in a while you specify a format and the model improvises anyway. Three common causes:

1. **Conflicting instructions**: system prompt and user content argue, and the model leans toward whichever came later.
2. **Format vs content conflict**: "output only JSON" versus "explain why" — the model usually chooses to be clear.
3. **Overwhelmed prompt**: the format rule is buried in fifty lines, so the model sees it but does not weight it.

The fix always points the same direction: **put the format rule last, make it the most visible line, and make sure nothing contradicts it.** This also echoes the "pin down the look" idea in `prompt-04-robustness`.

## Turn a good prompt into a convention

When you find a prompt that works every time, do not leave it in the chat history — turn it into a convention. Save the winning "role + examples + format" and reuse it next time.

* Write it down as a document: "this is our fixed customer-service prompt."
* Put it in project config (a `CLAUDE.md`, for example) so every model call in the team uses the same structure.
* Version it: keep a record of every change to role, examples, and format, so you can answer "why was it better before".

This upgrades a one-off successful prompt into a reusable asset. It is a different game from `train-02-finetuning` — fine-tuning changes the model, while here you pin down behavior with a prompt, no retraining needed.

| Level | Practice | Cost |
|---|---|---|
| Personal | Keep it in your own notes | almost zero |
| Project | Write it into a `CLAUDE.md` / config | low |
| System | Version it, add validation and retry | medium (see `prompt-04-robustness`) |

## Next step

You can now say things clearly and you can hand over role and format. The next technique is the most powerful — and the one that needs the most care: **getting the model to reason step by step**, why it helps, and when it backfires. See `prompt-03-chain-of-thought`.

#### Q: Why are few-shot examples often more effective than describing a rule in words?

* Examples let the model look up the original source in its training data

* A few paired examples directly demonstrate the pattern, and the model infers style and structure from them

* Examples make the model finish faster

* The model cannot understand abstract written rules

> 💡 An LLM is trained to continue text, so it is especially good at following examples. A clean input-to-output pair shows the pattern directly, easier to infer and reproduce than an abstract description.
