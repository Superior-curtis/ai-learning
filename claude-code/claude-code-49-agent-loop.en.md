# Life of a Task: The Claude Code Agent Loop, Step by Step

> 📅 2026-07-28 · Core Mechanics
> An interactive, step-by-step walkthrough of the full agent loop behind a Claude Code task — from input to output, what actually happens at each step.

---

Claude Code can "do things on its own" not because the model is magical, but because it breaks a task into a repeatable loop: **decide → execute → observe → decide again**. This article uses an interactive navigator to break that loop apart, step by step.

> Pair this with the simulator page — there, every step can reveal the actual prompt and payload.

## The five stages of the loop

#### Receive input

You type a message and Claude Code receives a user message. It may be "help me fix a bug" or a large batch of requirements.

#### Assemble context

It gathers git state, CLAUDE.md, the system prompt, and conversation history into a complete context window.

#### Call the model

The assembled context is sent to the model. The model emits one of two things: plain text, or a "I want to use a tool" request (tool\_use).

#### Execute the tool

If it is a tool\_use, it really executes — reading files, running tests, searching. The tool result (tool\_result) is sent back into context.

#### Loop until done

Back to "call the model". Repeat until the task finishes or the limit is reached, then assemble the final text response for you.

## Key idea: tools are the switch for "action"

The model is great at "understanding" and "planning", but tools are what actually act. That is why the tool system takes up so much of Claude Code's architecture.

> Keep this mental model: the model thinks, the tools act. Every tool\_use is a "I want to do this" request; Claude Code executes it and feeds the result back.

## A live loop demo

Here is a simplified terminal demo simulating Claude Code running a "run the tests and fix the failing one" task:

```text
$ claude "run tests, fix the failing one"
✦ Let me check the test status first.

$ Bash: npm test
FAIL  checkout.test.js
✗ should calculate total correctly

$ Read: src/checkout.ts
…read 42 lines
→ found a tax calculation order bug

$ Edit: src/checkout.ts
Applied the change (1 line)

$ Bash: npm test
PASS  checkout.test.js
✓ all tests pass
```

Notice those middle two steps? **Read → Edit → test again**. That is the power of the loop: each step's result becomes the next step's input, until the goal is met.

## Context compression: why the loop doesn't explode

You might ask: after many iterations, doesn't the context window overflow? Yes — that is why Claude Code has a **tiered compression** mechanism that compresses history by importance when the window gets too big.

#### The compression principle

"Keep what matters most, summarise the next, drop the rest." The core of the conversation and the current task are usually preserved; older peripheral content gets summarised or discarded.

#### What you can do

You can manually /compact the conversation or /clear to start over. For long tasks, occasional manual compression keeps the model focused on what matters.

## Design trade-off: why not compute it all at once

You might wonder: why loop at all? Wouldn't one big call be faster?

#### 對照 / Comparison

```text
Single call
One shot: "fix the tests"
→ model returns the
  complete result
Pros: fast
Cons: no visibility into
  intermediate state
  no verify-as-you-go
```

```text
Agent loop
Multiple iterations
→ read → edit → test
→ adjust from results
Pros: every step verified
  handles uncertainty
Cons: slower
  burns more tokens
```

The agent loop trades "a bit more latency" for "every action gets verified". For real coding tasks, that trade is almost always worth it.

## Check your understanding

#### Q: What does a tool\_use message represent?

* The model has finished and is waiting for confirmation

* The model expresses "I want to run a tool", and Claude Code executes it

* An error message from a failed tool

* A tool quietly running in the background

> 💡 tool\_use is an action request from the model, not a result. The actual execution happens next: Claude Code calls the tool, gets a tool\_result, and feeds it back into context.

## Further reading

* Want the full loop at the source-code level? Read [QueryEngine: the core loop](/posts/claude-code-09-engine)
* Curious how context is assembled? Read [The context system](/posts/claude-code-07-context)
* Want to play with each step's payload? Visit the [simulator](/simulator)
