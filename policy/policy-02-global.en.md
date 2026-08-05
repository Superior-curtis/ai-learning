# How the World Regulates AI: Four Routes Side by Side

> 📅 2026-08-04 · Core Concepts
> There is no single world AI law. The EU legislates by risk, the US leans on sectoral rules and voluntary commitments, China issues targeted regulations, the UK bets on pro-innovation principles — four routes, four governance philosophies. This post compares them side by side without taking sides.

---

Ask "who regulates AI" and the honest answer is **everyone — differently, and barely in sync with one another**. The last post covered the EU AI Act; this one pulls the camera back to compare the routes taken by the US, China, the UK, and the international layer underneath them all.

These routes are not a contest of right and wrong. Each one is a bet on which failure a country fears most. Once you see what each side is afraid of, the laws stop looking arbitrary.

## The EU: one law for everything (risk-based law)

Recap of `policy-01-eu-ai-act`: the EU chose **horizontal legislation** — a single AI Act covering every sector and every system, with a risk ladder deciding how heavy the duties get. It is binding, it reaches beyond EU borders, and fines scale with global turnover.

The EU mental model: **treat AI as a product-safety problem.** Like a drug or a car, an AI system must demonstrate safety before it may be sold in Europe.

## The US: sectoral rules plus voluntary commitments (the patchwork)

The US sits at the other extreme. As of this writing, **there is no comprehensive federal AI law.** Instead there is a patchwork:

* **Sectoral rules**: when AI lands inside an existing regulator's lane, existing law applies — the FDA for medical AI, the FTC for consumer deception, banking regulators for finance.
* **Voluntary commitments**: in 2023 the White House collected safety pledges from the leading labs — red-teaming, watermarking, incident reporting.
* **Executive orders**: a 2023 order asked federal agencies to build AI evaluation frameworks, but it was rescinded in 2025 — a reminder that a change of administration can swing the whole approach.
* **State laws fill the gap**: with the federal government quiet, states act. Colorado passed an AI Act in 2024, the first fairly comprehensive state-level law.
* **Soft standards**: NIST's AI Risk Management Framework is not mandatory, but it has become the industry's common language.

The US bet in one line: **"Let it run; if something breaks, use existing law."**

## China: targeted rules for each pain point (specific regulation)

China did not write one omnibus law. Instead it issues **specific rules for each perceived problem**:

* **Algorithmic recommendation rules (2022)**: transparency and explainability for recommendation algorithms, plus labels on synthetic content.
* **Deepfake and voice-cloning rules (2023)**: AI-generated faces and voices must be clearly labeled as AI-generated.
* **Generative AI measures (2023)**: services offered to the public must pass content review, and models serving the public must be filed with the authorities.
* **Foundation-model filing**: training and releasing models requires registration, and providers carry content responsibility.

The China bet in one line: **"Content and platform responsibility come first."** The core concern is less "is the product safe" and more "is the generated content lawful, and did the platform manage it properly".

## The UK: pro-innovation, principles first (principle-based route)

The UK deliberately chose the lightest touch: **cross-sectoral, principle-based, and non-statutory.**

* No new "AI authority". Existing regulators — information, communications, competition — fold AI into their existing remits.
* A set of five principles: safety, transparency, fairness, accountability, and contestability.
* The framework is currently **not legally binding**. The strategy is to encourage innovation first and only consider legislation once concrete problems actually show up.

The UK bet in one line: **"Grow the AI industry here first; regulate later if we must."**

## The international layer: a common floor beneath everyone

Beneath the countries, a slower layer of international coordination is forming. It is not law, but it shapes national law:

* **OECD AI Principles (2019)**: the closest thing to a global consensus, and the blueprint for many national laws.
* **G7 Hiroshima Process**: a voluntary international code of conduct for AI developers.
* **Council of Europe Framework Convention (2024)**: the first binding international AI treaty (a different instrument from the EU AI Act).
* **UNESCO Recommendation on the Ethics of AI**: broader, more ethics-oriented.
* **The AI Safety Summits (starting at Bletchley, 2023)**: the first time governments and leading labs sat down together about frontier AI risk, later growing into an ongoing series of international discussions.
* **The AI-trade intersection**: cross-border data flows and chip export controls are also "AI governance" — they just run through trade and security channels rather than AI-specific statutes.

## For companies: first ask where your market is

The map is nice; the real question is "what does my company actually have to obey?". Three questions locate you fast:

1. **Where is your market?** Selling only at home means mostly your own rules; going global stacks the EU, US-state, and Chinese requirements on top of each other.
2. **Does your product touch a sensitive field?** Hiring, medical, and finance are the most-watched categories everywhere — even where no AI-specific law exists, sectoral law is waiting.
3. **Are you "selling the model" or "using the model"?** In the EU, whoever develops and places a system carries the provider's heavy duties; companies that mostly *use* one have much lighter obligations.

There is no standard answer, but the questions decide how much compliance work you buy: **the same chatbot needs one honest sentence in the EU's minimal tier and a full high-risk pipeline the moment it is used for hiring.**

The three-step path most cross-border companies end up walking looks like this:

#### Map the jurisdictions you touch

Lay out your product list and check it against the EU AI Act, US sectoral law, and Chinese targeted rules, marking which products fall where.

#### Flag sensitive domains

Hiring, medical, and finance push you up a level of caution — even where no AI-specific law exists, sectoral law is waiting.

#### Build the compliance file

Get documentation, data governance, and use policies ready; the EU "prove it is safe" mindset is becoming shared language.

## Four routes, side by side

| Region | Route | Binding? | What it governs | The bet in one line |
|---|---|---|---|---|
| **EU** | horizontal law, risk tiers | yes, incl. extraterritorial | all AI systems, tiered by risk | "AI is a product-safety problem" |
| **US** | sectoral rules + voluntary | partially (health, finance, etc.) | depends on the sector; no federal catch-all | "Let it run; patch it later" |
| **China** | targeted rules, content focus | yes | generated content, recommenders, model filing | "Content and platform responsibility first" |
| **UK** | principles, non-statutory | no | enforced by existing regulators | "Innovation first, regulate if needed" |

> Do not ask "which country regulates hardest" — that is the wrong question. The right one is "which countries are regulating different things". The EU governs whether a product is safe, China governs whether content is lawful, the US governs whether an existing sectoral law covers a new harm, and the UK governs whether AI gets to survive at all. Four different questions, four different answers.

Run one AI product through four checkpoints and the questions look nothing alike:

```text
one AI product
├→ EU asks: is the product safe? (risk tiers)
├→ US asks: does any sectoral law cover it? (patch later)
├→ China asks: is the content lawful, did the platform manage it? (targeted rules)
└→ UK asks: will this scare off innovation? (principles first)
```

## Why the divergence will not disappear

Behind each route is a different fear:

* **The EU fears** unsafe, uncontrollable AI products harming citizens → so it writes product-safety law.
* **The US fears** that heavy regulation will strangle innovation and competitiveness → so it lets things run and patches after the fact.
* **China fears** runaway content, irresponsible platforms, and a disturbed information space → so it governs content and platforms.
* **The UK fears** falling behind in the AI race → so it keeps the door open.

None of this is frozen. The EU is still calibrating how the GPAI rules land, US states are filling the federal vacuum, and international treaties are trying to lay a common floor. **The directions will shift; the underlying fears behind the divergence will not disappear anytime soon.**

## Why coordination is so hard

If everyone is regulating AI, why not sit down and agree on one rulebook? The reasons are practical:

* **Definitions differ**: what counts as an "AI system" or "high risk" varies wildly across statutes; what country A regulates, country B may not even recognize.
* **Path dependence**: each country grows its new law on an old base — the EU on its product-safety tradition, the US on sectoral law, China on strong platform regulation.
* **Industrial competition**: nobody wants to fall behind in the AI race because of rules, so "strict" versus "loose" becomes a strategic choice.
* **International treaties are slow**: AI moves far faster than diplomacy; by the time a treaty lands, its content is already dated.

So the practical world is not "one global rulebook" but "national rules, plus your own patchwork job".

## The one-line recap

There is no single world AI law — there are at least four parallel routes: the EU regulates risk by legislation, the US mixes sectoral rules with voluntary commitments, China issues targeted rules about content and platforms, and the UK leans on non-binding principles to encourage innovation. Reading this map is more useful than memorizing any single statute.

Nowhere does the divergence cut sharper than at open source — where safety and control pull in opposite directions. Next: the tension between open weights and regulation, `policy-03-open-source`.

#### Q: Which statement best describes the US approach to AI regulation today?

* A comprehensive federal AI law has been in force for years

* There is no single federal catch-all law; it is a mix of sectoral rules, voluntary commitments, and state laws

* The US does not regulate any AI applications at all

* A single new Federal AI Authority coordinates everything

> 💡 The US has no comprehensive federal AI statute. Regulation happens where AI falls inside an existing agency lane (FDA, FTC, banking), through voluntary pledges, executive orders, and state-level laws such as Colorado 2024.
