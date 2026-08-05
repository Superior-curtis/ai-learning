# The EU AI Act: The World's First AI Law

> 📅 2026-08-04 · Core Concepts
> In 2024 the EU passed the world's first comprehensive AI law. Its core idea is not banning AI — it is a risk ladder: the higher the risk a system poses, the heavier the obligations. This post walks through the tiers, duties, penalties, and timeline in plain language.

---

In August 2024, the European Union put the world's first comprehensive AI law into force. Not the first law to touch AI, and not the most detailed — but the first that tries to cover the whole field in one statute. The question it asks is not "should AI exist" but something sharper: **what are you going to use it for?**

The same face-recognition model, deployed at an office door and deployed for live street surveillance, gets treated completely differently under this law. Understanding why one technology carries many sets of rules is the entire key to the Act.

## A risk ladder, not a ban

The EU refused both lazy options — "regulate all AI" and "leave AI alone". Instead it sorts use cases into four risk tiers and attaches duties to each tier. The lower the risk, the lighter the load; at the top of the ladder, some uses are simply banned.

| Risk tier | Examples | Legal duty |
|---|---|---|
| **Unacceptable** (banned) | social scoring, exploiting vulnerable groups, real-time remote biometric identification in public spaces (with a narrow law-enforcement exception) | cannot be placed on the market or used |
| **High risk** | hiring, credit, medical, education scoring, critical infrastructure, justice | the heaviest full compliance package |
| **Limited risk** (transparency) | chatbots, deepfake generation, emotion recognition | tell the user: "this is not a human" or "this is AI-generated" |
| **Minimal risk** | spam filters, games, translation | effectively unregulated; voluntary codes encouraged |

Note the subtlety: **the law bans a class of uses, not a class of technology.** The same biometric model can be sold to a company for office access control, but cannot be used for live public surveillance. Green light on the technology; red light on the dangerous way of using it.

> The sentence to remember: the EU AI Act is not a law that bans AI; it is a risk ladder. The same model carries different duties in different uses. High risk does not mean "you cannot ship" — it means "prove it is safe before you ship".

## Who the Act reaches

The Act deliberately reaches far beyond Europe:

* **Providers** — whoever develops a system and puts it on the EU market, wherever their company is based.
* **Deployers** — organizations actually using such a system inside the EU.
* **Importers and distributors** — the people who bring systems across the EU border.
* **Authorized representatives** — providers outside the EU must appoint a legal representative inside it, so regulators always have someone to talk to.

In plain terms: **if your AI product sells into the EU, this law follows it.** This "extraterritorial reach" is one of the most important facts for any developer based outside Europe.

## The high-risk obligations: the heavy section

Land in the "high risk" tier and the Act asks you to build a full management system. The package reads like this:

1. **A risk management system** — identify, assess, and mitigate risk from the design stage, and keep updating it.
2. **Data governance** — show the training and validation data are relevant, complete, and free of obvious bias.
3. **Technical documentation** — write down the design, intended use, and evaluation results so regulators can review them.
4. **Automatic logging** — the system records its own operation so events can be traced afterward.
5. **Human oversight** — the design must let a person intervene and override decisions.
6. **Accuracy, robustness, cybersecurity** — prove the system holds up in the real world and is not derailed by small perturbations.
7. **Conformity assessment and CE marking** — get the system checked (self-assessment, or by a third party for some uses) before release.
8. **EU database registration** — register in the public database so the system is visible.

This is the familiar logic of pharma and aviation: **the question is never "can it be sold" but "can you prove it is safe before selling it".** For developers the real cost is often paperwork, not engineering.

## Limited and minimal risk: the duties are light

If you are not in the "high risk" tiers, the Act gets much lighter. **"Minimal risk" is effectively unregulated** — spam filters, games, and translation have no listed duties at all; the Act merely "encourages" providers to follow voluntary codes of conduct.

"Limited risk" has exactly one duty: **tell the user they are talking to AI.** For example:

* A chatbot must make clear it is a machine — you should not believe you are talking to a human.
* AI-generated deepfakes (images, audio, video) must be labeled as AI-generated where reasonable.
* Emotion recognition systems must tell people their emotions are being judged.

No review process, no pre-market check — just "say the honest sentence". For most consumer-grade AI products, that is your entire to-do list.

## GPAI: a layer added for foundation models

Late in the negotiations, the ChatGPT wave forced the EU to add a whole extra layer for **general-purpose AI models** — models too general to fit neatly into any single use-case tier. This layer runs alongside the risk ladder:

* **All GPAI models** must keep technical documentation, publish a summary of their training data, and have a copyright policy so rights holders can state whether they want their work used in training.
* **GPAI models with "systemic risk"** (roughly, trained above 10²⁵ FLOPs of compute) face extra duties — adversarial stress-testing, reporting serious incidents, and cybersecurity protections.

The 10²⁵ FLOPs figure was an attempt to capture only the top tier of models, not every small open model. Where that line lands has become one of the hottest questions in the open-source debate (`policy-03-open-source` covers it).

## Who enforces it: a multi-layered structure

The Act does not create one all-powerful AI authority. Enforcement is layered:

* **The EU AI Office**: sits inside the European Commission and focuses on GPAI models and cross-border matters.
* **National authorities**: market surveillance, compliance checks, and fines are run by each member state's own institutions.
* **Standards and codes of practice**: many details — how GPAI should operate, the technical standards for high-risk systems — are not fixed directly in the law but filled in by "codes of practice" that industry helps draft.

The upside is that this fits the existing product-safety system. The downside is obvious: **enforcement strength varies by country** — the same product can be treated differently in a strict member state versus a lax one. For cross-border companies, the cheapest strategy is usually "prepare for the strictest country".

## The fines are meant to hurt

To make the rules bite, the Act ties penalties to **global turnover** — the bigger you are, the more it costs:

| Violation | Maximum fine |
|---|---|
| Using a banned "unacceptable risk" system | €35 million, or 7% of worldwide annual turnover (whichever is higher) |
| Violating high-risk obligations | €15 million, or 3% of worldwide turnover |
| Supplying wrong information to authorities | €7.5 million, or 1.5% of worldwide turnover |

SMEs and startups get reduced caps. Seven percent of global turnover is the kind of number that makes a compliance department nervous — this is not a suggestion.

## A phased timeline

The Act is already in force, but the duties switch on gradually to give industry runway:

```text
2024-08  law enters into force
↓
2025-02  banned-practices provisions apply
↓
2025-08  GPAI obligations apply
↓
2026-08  most high-risk obligations apply
↓
2027-08  remaining high-risk (medical, vehicles) catch up
```

Read simply: **the bans arrive fast; the high-risk compliance queue takes years.** For many high-risk products the real deadline is 2026–2027 — there is still time to build compliance in.

## What the Act does not cover

Before knowing what is in scope, it helps to know what is out:

* **Military and defense uses**: largely outside the Act's scope.
* **Purely personal, non-professional use**: writing yourself a small tool is not "placing on the market".
* **Research and pre-market prototypes**: systems still in development mostly have no obligations yet (though research on banned practices still has limits).
* **Plain software**: the Act governs "AI systems"; ordinary programs and algorithms are out of scope.

The boundary is pragmatic: **"AI placed on the EU market for others to use" is the home turf.** Playing at home or prototyping in a lab will not immediately drag you into compliance.

## What it means for developers

You do not need to read the whole text. Answer three questions:

1. **Does my product reach the EU market?** No — mostly not your problem. Yes — keep reading.
2. **Which tier am I in?** Chatbots, translation, and filters are usually minimal. Hiring, credit, health, and education put you in high risk.
3. **Am I a GPAI provider?** Only if you train a very large general model and put it on the EU market.

The most common misunderstanding is picturing the Act as "a license for all AI". In practice the EU asks you to **pay by risk**: minimal-risk products carry almost no burden; the expensive compliance work concentrates in a small number of high-risk categories.

## The one-line recap

The EU AI Act is the world's first horizontal AI statute, built on a risk ladder: ban the most dangerous uses, manage high-risk systems closely, and ask only for transparency from the rest. It reaches any provider selling into the EU, fines scale with global turnover, and the obligations fully land by 2027.

With Europe as the yardstick, the natural question is: **what about everyone else?** The US, China, and the UK have each chosen very different routes — we compare them all in `policy-02-global`.

#### Q: What is the core regulatory mechanism of the EU AI Act?

* A blanket ban on all AI development and research

* Risk-based tiers: the higher the risk, the heavier the obligations — what gets banned is dangerous uses, not the technology itself

* It only covers the largest US and EU tech companies

* It has no law at all; everything relies on voluntary industry self-regulation

> 💡 The Act never bans AI technology. It sorts use cases into four tiers — unacceptable, high, limited, and minimal — and attaches obligations to each tier. What is prohibited is a class of uses, not a class of technology.
