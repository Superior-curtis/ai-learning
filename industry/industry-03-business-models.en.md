# Business Models

> 📅 2026-08-04 · Core Concepts
> How do AI companies actually make money? Metered tokens, SaaS wrapping, apps and agents, open weights plus services — and the cruelest part, unit economics: does the math work?

---

Once you hold the value chain (`industry-01-value-chain`) and the player map (`industry-02-players`), one practical question remains: **how does this business actually collect revenue?**

The answer is not a single sentence but a mix of models, each with its own pricing logic, customers, and failure mode. This article puts the mainstream five on the table and then runs the brutal "burn vs. revenue" reality check.

## Model one: the metered-token API

The most intuitive model: **pay for what you use.** The unit of a model is not the hour; it is the "per 1,000 tokens" (roughly a few hundred words).

```text
input tokens (what you send in) → output tokens (what the model emits)
            ↓ each is billed
one call = input price × volume + output price × volume
```

* **Who it suits:** developers. Costs are predictable, no GPUs to provision, and it scales from zero to a million users on demand.
* **Who it hurts:** heavy-volume companies. Long documents, agent loops, and large batches push the bill up linearly with usage.
* **The economics:** you are selling the commodity called "inference" — so inference cost is the denominator of your gross margin. See `econ-01-inference-cost`.

## Model two: the SaaS wrapper

Wrap the model inside a product shell you already know: documents, spreadsheets, support desks, writing tools. Users are not buying tokens; they buy **"one flat monthly fee."**

* **Pricing edge:** a predictable subscription instead of a floating usage bill.
* **Retention edge:** customers pay for a problem-solving workflow, not just for a model — switching models is easier than switching workflows.
* **Risk:** wrap it too thinly and users can simply call the API themselves and clone you; wrap it thick enough (process, data, integration) and you have a moat.

## Model three: apps and agent products

Beyond "wrapping," this is **actually finishing a task for the user**: auto-support, auto-reporting, software that operates itself (agents, see `agents-01-what-is-agent`).

* Pricing often starts from "outcome": complete a task, process an invoice, close a ticket.
* It justifies the price better than "per token" — the customer pays for the value of the outcome, not the compute behind it.
* But the cost hides where you are not looking: one agent run may fire a dozen model calls, multiplying the cost of a single answer by an order of magnitude. Inference cost eating the margin is this model's most common crash.

## Model four: open weights plus services

Instead of training your own, **stand on open weights (Llama, Qwen, DeepSeek) and sell "deployment + operations + customization + consulting."**

* Customers buy a turnkey way to run a model safely on their own servers — common in regulated fields like healthcare, finance, and government (echoing `models-01-open-vs-closed`).
* Revenue comes from project fees, retainers, and labor time rather than metered usage — higher price per deal, but the margin profile resembles traditional software services.

## The five models at a glance

| Model | What you sell | How you bill | Main risk |
|---|---|---|---|
| **Token API** | inference capability | by usage | inference cost squeezes margin; big customers self-host |
| **SaaS wrapper** | a product that solves problems | monthly / annual | wrap too thin, bypassed by API |
| **Apps / agents** | the outcome of a finished task | per task / outcome | repeated calls blow up cost |
| **Open weights + services** | a whole self-hosted solution | project / retainer / days | hard to scale, labor-intensive |
| **Ads / traffic (auxiliary)** | free AI to draw traffic | ads or ecosystem funnel | does not sell AI directly; margin lives elsewhere |

## The brutal unit economics: burn vs. revenue

Every model shares one denominator — the **"do I profit on each delivery"** ledger:

```text
gross profit = price - delivery cost
delivery cost ≈ inference + customer acquisition + support

Big revenue is not profit:
  sell a million "single answers,"
  but if inference cost eats more than half the price each time
  → the bigger you scale, the more you lose
```

* **The labs' situation:** both training (`train-01-pretraining`) and inference are expensive; they lean on massive API revenue to amortize. Much "revenue growth" here is paid for with cash burned.
* **The apps' situation:** revenue grows fast, but if each dollar of AI feature costs more than a dollar of inference and acquisition, you lose faster the bigger you get.
* **The healthy signals:** not revenue, but **"gross profit per unit" and "customer retention."** When users come back and repeat, it means what they paid for is genuinely worth it.

> The one thing to remember: the verdict on an AI business is not revenue growth but the delivery cost behind every dollar of it — above all inference cost (econ-01-inference-cost). Growth that is gross-margin positive and keeps customers is real growth.

## A worked example: the unit ledger of one agent run

Saying abstractly that "cost eats margin" is weaker than running the numbers. Assume $3 per million input tokens and $15 per million output tokens (order-of-magnitude realistic), and an auto-support agent that fires four model calls to close one ticket:

```concept:&#x20;the&#x20;inference&#x20;cost&#x20;of&#x20;one&#x20;task
# input $3 / 1M tokens, output $15 / 1M tokens
# auto-support closing one ticket = 4 model calls

step          input   output  cost ($)
1. read ticket  0.5K   0.1K   0.003
2. query system 0.6K   0.3K   0.006
3. draft reply  1.0K   0.2K   0.006
4. summary      0.8K   0.1K   0.004
────────────────────────────────────
total           2.9K   0.7K   0.019

→ inference cost of one ticket ≈ $0.02
price $0.05 → 60% gross margin
price $0.01 → lose $0.01 per ticket, lose more as you scale
```

Read that ledger and you see why outcome-priced agent products are especially dangerous: **the customer thinks they bought "one finished task," but your cost is "N model calls behind the task."** The more complex the task, the more tools, the more retries — the more the unit cost runs away.

## The price war: models racing to get cheaper

Another live dynamic is that **pricing itself is being used as a weapon.**

* New model releases often arrive "stronger and cheaper" at once, buying share with price cuts.
* Open weights (`models-01-open-vs-closed`) push the price floor near zero — self-hosted, you only pay the compute.
* The result: **the unit price of tokens is falling, but usage is exploding.** For labs it is a "thin margin, huge volume" bet; for the app layer it keeps inflating the "inference cost" denominator.

Every one of these rulers eventually points at the same question: what does one answer actually cost — `econ-01-inference-cost`.

## Scale decides the playbook: same chain, different moves

Facing the same five models, players of different sizes make completely different choices. Use "scale" as the coordinate and the strategic pattern appears:

| Player scale | Typical model | Why |
|---|---|---|
| **Individual / small team** | start on Token API | zero upfront cost, grows with usage, cheapest way to validate demand |
| **Growing startup** | Token API + SaaS wrapper | prove willingness to pay first, then talk about lock-in |
| **Mid-size product company** | self-host open weights + services | when usage gets scary on the meter, run the self-host math |
| **Large enterprise** | mix: API + self-host + private data | route sensitive vs. non-sensitive, balance risk and cost |
| **Platform giant** | full-chain integration | hold model, compute, and distribution at once |

One practical rule: **when usage is small enough to be predictable, use the API; when usage starts to govern your margin, consider self-hosting; at giant scale you are no longer picking a model — you are building an ecosystem.**

## The consolidation trend: thin layers get squeezed

Finally, zoom out. These models are being crushed by two forces:

1. **Upstream integrates down:** clouds and labs bundle "model + API + platform" themselves, squashing the thin intermediaries who merely resell API access.
2. **Downstream consolidates up:** app companies with real use cases and data get acquired by capital or the majors, stitching "model" and "distribution" together.

The result is "**either own an irreplaceable use case and data, or own scaled infrastructure**" — the middle ground is vanishing. That is exactly why we keep seeing waves of M\&A and alliance news: people are not fighting over models; they are fighting over **position.**

## Next

Industry, players, and business models are covered. Now the lens turns to the hardest numbers: **what one answer actually costs, and what decides it** — `econ-01-inference-cost`.

#### Q: Why can a metered-token API business look like it is growing fast on revenue yet still not be profitable?

* Because token pricing is too expensive for anyone to buy

* Because every delivery carries an inference cost — profit is the margin between price and cost, so big revenue does not mean positive margin

* Because API businesses cannot scale

* Because users never pay

> 💡 Revenue equals price times volume, but every delivery also costs inference, acquisition, and support. If each dollar sold costs more than a dollar to deliver, you lose more the bigger you scale — so watch gross margin and retention, not just top-line revenue.
