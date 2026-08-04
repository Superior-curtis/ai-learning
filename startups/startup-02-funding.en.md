# Funding & Valuation

> 📅 2026-08-04 · Core Concepts
> How does money flow into an AI startup, layer by layer? From seed to Series A, what actually drives the valuation — revenue, compute, or team? This article maps the funding rounds, the valuation logic, and an unromantic reality-check table. Not investment advice.

---

The last article laid out how to build the company (`startup-01-building`). But before any product can land, one question has to be answered: **where does the money come from?** This article opens up the rules of funding — not "how to raise," but how money flows in layer by layer and how a valuation actually gets negotiated. Financial products carry risk; this is educational content, not investment advice.

Set the mindset first: funding is not proof that your company is good. It is a **trade — you give up part of the company to buy growth capital** — faster growth in exchange for dilution and accountability.

## The rounds: how money comes in layers

The typical funding path climbs a staircase; each rung solves a different stage's problem. The names are convention, and in practice companies skip or overlap, but the direction looks like this:

| Round | Stage | Typical source | Main use |
|---|---|---|---|
| **Pre-seed** | idea / prototype | self, friends & family, angels | turn the idea into a demo you can show |
| **Seed** | early product, a few users | angels, early-stage VCs | find your first paying customers |
| **Series A** | paying customers and a revenue curve | venture funds | find a repeatable growth engine |
| **Series B onward** | fast growth, need to scale | large funds, corporate, PE | scale, go global, buy others |

Every round is, in essence, **"buy time with future growth, sold for today's capital."** The further along you go, the higher the bar, and the less investors want to hear vision — the more they want numbers.

## The "evidence" each round wants

"Numbers" are not one kind of number — different rounds want different evidence. Getting this backwards is the most common reason a pitch fails:

#### Pre-seed: the idea and the team

There is almost no data to show, so investors bet on "are these the people most likely to pull this off." Domain depth and execution matter.

#### Seed: usage and retention

The product works and people use it. The evidence is active users, retention, and whether growth is organic — not just download counts.

#### Series A: the revenue curve and unit economics

There are paying customers and revenue is growing, but the crux is the delivery cost behind every dollar — gross margin and retention (industry-03-business-models).

#### Series B onward: proof of scaling

The growth engine is repeatable and the only question left is "can you turn it up." It is about acquisition efficiency, channel duplication, and organizational muscle.

Aligning your "evidence" to the round beats telling a better story every time.

## Valuation is argued, not computed

The most misread point: **a valuation is not an objective number; it is the outcome of a negotiation.** Each side brings its own story, and you sign on a piece of paper. For an AI startup, the valuation argument usually turns on three pillars:

* **Revenue (the hardest pillar).** With revenue on the books, the market multiplies it — e.g., some multiple of revenue or gross margin. A company with real revenue has the most defensible story.
* **Compute and scarce resources (the AI-specific pillar).** If you hold GPU allotments or proprietary training data others cannot get, investors price "resource scarcity" into the number. This is the same logic as the "upstream sells scarcity" point in `industry-01-value-chain`.
* **Team (biggest early).** Pre-seed and seed rounds are nearly "investing in the person, not the spreadsheet" — founding track record, domain depth, and the ability to hire are the biggest variable early on.

In one line: **revenue anchors later-stage valuation, scarce resources lift expectations, and the team carries the earliest layer of trust.**

## Running the valuation logic

Saying "valuation is a negotiation" is weaker than seeing one rough assembly. Here is an illustrative walk (pure education; the numbers represent no real market):

```concept:&#x20;roughly&#x20;assembling&#x20;a&#x20;seed&#x20;valuation
# illustration: how a seed valuation might get "assembled"
# note: this is not a formula, just arguments around a table

argument A (comps): similar non-AI startups at this stage ~$8M
argument B (revenue): $0.3M annualized revenue x 20x -> $6M
argument C (scarce resource): exclusive data/compute -> +$2M -> $8M
argument D (team): founder with a prior exit -> +$2M -> $10M

investor quotes $6M-$10M -> the closing number is negotiated
```

Three wrinkles: **multiples bend with market heat, so the AI halo can inflate multiples short-term; team adds the most early and fades to almost nothing late; and the premium for scarce resources only holds if it actually converts into revenue.**

## Burn rate and runway

Two of the least-explained but most deadly words in funding: **burn rate** and **runway**.

* **Burn rate:** how much the company nets per month (revenue minus all costs). The raised money is not to "spend" — it buys you the day you hit a real milestone.
* **Runway:** cash on hand divided by monthly net burn — "how many months you can still live." The runway a healthy investor expects is far longer than you think, because raising itself takes months.

```concept:&#x20;the&#x20;runway&#x20;calculation
# runway = cash balance / monthly net burn
# e.g. $10M cash, $1M monthly net burn
runway = 1000 / 100 = 10 months

# common red flag: starting to raise at under 6 months of runway
# -> your negotiating leverage is terrible, terms get ugly
```

**Starting to raise while the runway is still long** is one of a founder's most important disciplines. A deal negotiated out of desperation is rarely a good deal.

## The term-sheet lines that actually matter

When a term sheet arrives, most people stare at the "valuation" — but the small print that changes your life is elsewhere. At minimum, understand these:

| Term | What it is really saying |
|---|---|
| **Dilution** | after the new round issues shares, how much your stake falls. Every round dilutes; it is by design, not a conspiracy. |
| **Liquidation preference** | if the company is sold, who gets paid first and at what multiple. A high preference pays investors back before you see anything. |
| **Anti-dilution** | if the next round prices lower, investors get more shares as compensation. It protects them, at the cost of your slice. |
| **Board seats** | investor voting power over major decisions. More seats means less unilateral control for you. |
| **Option pool** | shares reserved for future hires. It dilutes every shareholder and is usually the cost of attracting a team. |

You do not need to become a lawyer, but **having someone who genuinely understands funding read it with you before you sign** is the cheapest insurance there is.

## Grants vs. VC: you do not have to sell equity

The other funding path is **grants** — money that does not require giving up equity. It is not that one is "better"; it is about which fits where you are:

| | **Grant** | **VC** |
|---|---|---|
| What you give | nearly nothing (at most milestone reports) | part ownership of the company |
| Amount | usually smaller, with usage restrictions | can be large, but chases high multiples |
| Pace | slow, long applications, judged on the plan | fast, judged on the growth curve |
| Who it suits | research, deep tech, public value | commercial companies scaling quickly |

The practical view: **grants suit teams that burn slowly and whose value is hard to measure in revenue; VC suits teams that burn fast and need capital to buy growth.** Many companies do both — a grant to survive the earliest phase, then VC to accelerate.

## What investors actually look for

Turning "people and numbers" into an actionable checklist, an AI investor usually runs through these before deciding:

1. **Realness of demand:** an "it must be AI" pain point, not "AI for AI's sake."
2. **Shape of the moat:** does your defense come from data, workflow, or distribution — or is it a thin wrapper (`startup-01-building`)?
3. **Unit economics:** how much of every dollar does inference cost eat (`econ-01-inference-cost`)? Is gross margin positive, or do you lose more as you scale (`industry-03-business-models`)?
4. **Team execution:** a track record of turning "commitments" into "shipments."
5. **Market imagination:** how big this could be in three years — but imagination only adds, it cannot replace the other four.

Investors are not daydreaming; they are **picking a high-probability bet** — your job is to prove the odds are in their favor.

## An unromantic reality-check table

The funding news is all unicorns, but survivorship bias is severe. Here is the sober list to run yourself through before signing anything:

| Check | The rosy version | Closer to reality |
|---|---|---|
| Funding odds | good ideas find money naturally | most startups never raise an A; seed is a minority's starting line |
| Valuation growth | doubles every year | valuation is set by the market; a bull-to-bear swing rewrites your story |
| Dilution | "it will rise back anyway" | every round thins founder equity and shrinks control and incentive |
| Compute cost | "models keep getting cheaper" | true short-term, but your volume grows too; the bill does not always fall |
| The "AI halo" | mentioning AI lifts the number | the halo is fading; investors increasingly want real revenue and margin |

> The one thing to remember: funding is a trade of ownership for growth capital, not a verdict on your company. A valuation has no objective formula — it is what revenue, scarce resources, and team argue into being across a negotiation. Before you sign, run the reality-check table yourself; do not let the unicorn headlines carry you.

## Next

Funding and valuation are mostly covered. The final hurdle is **mindset**: a lot of "common sense" floating around is actually myth — "the model is the moat," "more compute wins," "we need AGI." The next article takes each of these apart — `startup-03-myths`.

#### Q: Why do we say a valuation is negotiated rather than computed?

* Because valuations have no basis at all and are arbitrary

* Because the valuation is the consensus that investors and founders reach by arguing over revenue, scarce resources, and team

* Because only revenue can determine a valuation

* Because a valuation is only visible once a company goes public

> 💡 There is no single formula: revenue anchors a multiple, scarce resources lift expectations, and team carries the earliest trust. Each side brings a different argument to the table, and the closing price is the result of that negotiation.
