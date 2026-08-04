# Red Teaming in Practice: Finding Weaknesses Before Attackers Do

> 📅 2026-08-04 · Deep Dive
> Red teaming is less about 'hacking your own system' and more a disciplined method: threat modeling, controlled test environments, responsible disclosure, and acting only within authorized scope.

---

You shipped your system and everything looked fine. Then one day a stranger you have never met finds a hole somewhere you never looked. Red teaming is about getting your own people to play that stranger **first** — to find the weakness before a real attacker does.

The earlier pieces covered `security-01-prompt-injection`, `security-02-jailbreaks`, and `security-03-evals` — what attacks look like and how to measure them. This piece is about turning that into **a disciplined process**: red teaming in practice.

A quick framing: recall the `llmcore-01-next-token` mental model. A model is a next-token predictor, and instructions and data share the same token stream. So red teaming an AI system is rarely "find a bug in one line of code". It is more often "probe where this probability distribution gets steered by a given input". That makes AI red teaming closer to behavioral science than to pure code inspection.

## What red teaming actually does

Red teaming is simply: **a trusted group systematically hunting for a system's weaknesses from an attacker's point of view.** The goal is not "prove we are strong". It is "we find the problem before someone else does".

For an AI system, that usually spans several kinds of targets:

* **Behavioral weaknesses**: under which inputs does the model produce disallowed content, leak its system prompt, or break policy.
* **Tool abuse**: can the model be talked into calling tools it should not call (see `security-04-secure-apps`).
* **Data leakage**: can private information, system instructions, or secrets flow out through the output.
* **Backend weaknesses**: classic issues like injection and unauthorized access — AI does not solve everything, but it must not route around them.

One mindset matters above all: **red teaming is not a "caught one, we win" competition. It is a hunt for coverage.** The more systematically you think through the attacker paths, the more likely you are to close holes early.

## Start by drawing the threat model

Red teaming does not begin with wild testing. It begins by answering "what are we protecting, from whom, and what is the worst case." This is the same threat model from `security-04-secure-apps`; here it is the starting line:

```threat-model
Assets:   user data, system prompt, tool permissions, API keys
Attackers: curious testers, rivals after data, account stealers after support agents
Attack surface: user input, RAG documents, web content, model output, tool calls
Worst case: data leak, unauthorized action, reputational harm, legal liability
```

With those four columns you know where this red-team round attacks. An inbox assistant does not need testing for "impersonate a president". It needs testing for "can it be tricked into forwarding mail."

## A controlled test environment

Red teaming is often misunderstood as "fool around on production." A proper red team runs in a **controlled test environment**:

* **Use a replica or sandbox**: test on your own staging setup or an isolated model instance, not on a system facing real users.
* **Synthetic data**: use dummy "bait" users and data, and never touch real personal data.
* **Minimal blast radius**: if you must get close to production, scope the impact down to reversible actions.
* **Separate test accounts**: test accounts and permissions are kept fully apart from real users.

The point of a controlled environment is that you can **test boldly** without disturbing real users or touching real data. A red team can fail and emit "dangerous" output — as long as it happens inside a safe sandbox.

## Reporting and disclosure: finding is only the start

Once you find a weakness, the real value of red teaming begins. **A weakness that is not reported well is equivalent to not finding it.** A proper flow includes:

1. **Triaging**: score each finding by severity (Critical / High / Medium / Low) with reproduction steps and impact.
2. **Handing it to the fix team**: describe the issue and the recommendation clearly, not just "it broke."
3. **Tracking to closure**: every finding gets a lifecycle — opened, assigned, fixed, verified, closed.
4. **Regression**: after the fix, re-run the original cases to confirm the hole is really closed.

That echoes `security-03-evals`: **a technique that caused an incident should become a permanent automated test case**, so it does not silently come back later.

> The heart of red teaming is responsible disclosure, not "finding it counts as winning." A weakness found, reported, and fixed in a controlled environment is a totally different path from one exploited by an attacker in production. Authorization, controlled testing, and reporting — all three are required. Missing any one and you are no longer red teaming.

## Only test what you are authorized to test

It needs to be said plainly: **a red team only acts within authorized scope.** Unauthorized testing, even with good intentions, can break laws, violate a platform's terms of service, and damage other people's systems. A few firm rules:

* **Get authorization first**: a written document stating scope, timing, permitted techniques, and an emergency contact.
* **Respect the scope**: test only what was authorized. If you stumble onto something out of scope, stop and talk — do not barge through.
* **Only test systems you own**: never test third-party systems, someone else's servers, or services without permission.
* **Know the platform rules**: for cloud services or APIs, check what behavior counts as abuse (it can get your account banned or trigger legal action).

In one line: **half of red teaming is craft, half is discipline.** Real red teamers ask about their authorization boundary before they start typing.

## Make it a routine

Red teaming should not be a once-a-year burst. It is a **process you replay after every significant change.** Rather than staging one giant drill, the more practical move is to shrink it and embed it into your workflow:

#### Build the test set

Turn the threat model into a reproducible set of test cases, saved in the library.\n\n→ see `security-03-evals` for how to build this collection.

#### Set up a safe environment

Prepare an isolated sandbox and synthetic data, so testing can safely "misbehave".

#### Replay after every change

Re-run the same test set whenever the model, system prompt, RAG data, or tools change.

#### Track and fix

Triage, assign, fix, and regress — close out every finding.

Treat red teaming as part of your CI, and it stops being "the occasional adventure" and becomes **a shift that guards your system every day.**

| Term | Focus | Relation to red teaming |
|---|---|---|
| Red team | Find weaknesses from an attacker view | The star; about coverage and method |
| Penetration test | A one-off, deep technical check | One way to run a red team |
| Eval | Automate the measurement of behavior | Turns red-team findings into repeatable tests |
| Blue team | Build and maintain defenses | Red team's partner; receives the findings |

## A miniature walk-through: red teaming a support agent

Let us apply all of that to one example. Suppose you own a support agent with two tools: "reply to message" and "escalate ticket". One red-team round can look like this:

1. **Threat model**: the assets are support conversations and the system prompt; the attacker is an account thief who wants to talk the agent into favors.
2. **Test list**: can it be tricked into revealing another customer's order status? Can it escalate an unrelated ticket? Can the system prompt be coaxed out?
3. **Controlled environment**: replay every item in a sandbox with synthetic order data and synthetic users.
4. **Triage results**: order leakage is high risk, ticket mis-escalation is medium, prompt extraction is medium — each with reproduction steps.
5. **Fix and regress**: patch against the permission design in `security-04-secure-apps`, then drop all three items into the automated test set.

Note there is no "take the whole system down" moment in this example. Red teams often find **small behavioral holes** first — and those are exactly the ones that hurt most when exploited.

## Red teaming and alignment: do not forget the source

Red teaming is not a creation of the LLM era. In `train-03-rlhf` we saw that a large share of alignment training data comes from human red teams: people fire harmful prompts at the model and feed "refusing" back as the desired behavior. In other words, **red-team findings were once the nourishment that made models safer.**

That has two practical implications:

* **Separate application layer from model layer**: application red teaming tests "will my system prompt, guardrails, and tool permissions be bypassed"; model red teaming tests "will the model itself falter on harmful requests." Both are needed, but the settings differ.
* **Findings can be fed back**: with responsible disclosure, your application-layer findings can go back to the model vendor or the open-source community, so the fix is not limited to your application.

## What a good red-team report looks like

The report is the deliverable of red teaming, and the only artifact that gets re-read. A good report has four requirements:

1. **Reproducible**: anyone following the steps gets the same result — not "I think I hit it once."
2. **Evidence-based**: include full input and output records, proving "this is not a hallucination."
3. **Impact-focused**: say what a finding can actually cause, so the fix team knows how to prioritize it.
4. **Actionable**: not just "there is a hole here", but "change it to this" — defenders dread a hole with no direction.

When the report is done, remember the spirit of `security-03-evals`: **turn this technique into an automated test case so it re-runs itself in the future.**

## Next step

Red teaming finds the weaknesses. The next move is turning "blocking them" into a disciplined routine — guardrails and content filtering. `security-06-guardrails` walks through putting a model-independent check at the input, the system prompt, content isolation, and the output, so untrusted content rarely steers the model.

## One-sentence summary

> Red teaming is not "find a hole to prove you are weak". It is **finding weaknesses early, fixing them, and turning them into permanent tests — authorized, controlled, and responsibly disclosed — so your system gets repaired by your own people before real attackers arrive.**

#### Q: Why does red teaming run in a 'controlled test environment' instead of straight on production?

* Because production models are slower and tests would stall

* A controlled environment uses synthetic data and a sandbox, letting testing boldly probe bad behavior without affecting real users or real data

* Because a controlled environment is cheaper and lowers the testing cost

* Because red teams only test code, never behavior

> 💡 The core of a controlled environment is 'you can safely misbehave': with bait data and an isolated sandbox, testing can freely produce dangerous output and make suspicious requests without touching real users or personal data. That lets a red team probe the attack surface far more completely.
