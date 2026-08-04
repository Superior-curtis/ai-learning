# The AI Industry Value Chain

> 📅 2026-08-04 · Core Concepts
> From GPU chips down to end users, the AI industry flows like a layered river. See who sells to whom at each layer — and where the margin sits — and the whole business comes into focus.

---

When a prompt goes into a model and the answer comes back in a blink, a whole industry just moved: **chips → cloud compute → model labs → model APIs → apps & infrastructure → end users.** Every layer is a separate business with its own sellers, buyers, margins, and moats.

This article unpacks the value chain. Once you see which segment earns the money, you have walked through the door of "AI as a business."

## The chain behind one prompt

All you see in a chat box is the last layer. But underneath, it stacks like this:

```text
[ chips / GPUs ]     — NVIDIA and friends, selling "compute bricks"
      ↓ sold to
[ cloud / compute ]  — AWS / GCP / Azure, renting GPU-hours
      ↓ sold to
[ model labs ]       — OpenAI / Anthropic / Google / DeepSeek, forging weights
      ↓ through
[ model APIs ]       — renting out model capability, priced per token
      ↓ sold to
[ apps / infra ]     — support, writing, agent platforms wrapping it for users
      ↓ sold to
[ end users ]        — you pay the SaaS; your company pays for internal rollout
```

Each layer charges the one below it and pushes its costs down the chain. The river starts at the chip, and empties into the app on your phone.

## What each layer sells

| Layer | Sellers | What they sell | Buyers |
|---|---|---|---|
| **Chips / GPUs** | NVIDIA, AMD, Cerebras | compute hardware + interconnects | clouds, labs |
| **Cloud / compute** | AWS, Azure, GCP | GPU-hours plus network | labs, enterprises |
| **Model labs** | OpenAI, Anthropic, Google, DeepSeek | trained model weights | themselves or via API |
| **Model APIs** | the labs' API products | per-token inference | developers, app companies |
| **Apps / infra** | SaaS, agents, integration platforms | packaged features and interfaces | end users, enterprises |
| **End users** | you, your company | — (the paying side) | — |

## Where the margin concentrates

Layers don't earn equally. The key question is **"who holds an irreplaceable, scarce resource with no easy substitute?"**

* **Widest moat: chips.** To build a large model, the world basically passes through NVIDIA's GPUs and its interconnect ecosystem (CUDA). It sells both the shovel and the channel, and in a shortage it sets the price.
* **Next: top clouds.** Hyperscalers have the capital and scale to rent out tens of thousands of GPUs as one cluster, with network, storage, and security around it — a single GPU purchase can't sustain massive inference.
* **Fierce competition: model labs.** The most visible and the most cash-burning. Every few months they spend hundreds of millions to retrain (see `train-01-pretraining`) just to stay at the top of the leaderboard. Fall behind on quality and customers drift to a rival.
* **Splitting margins: the app layer.** The ceiling varies wildly: packaged as a "productivity tool" it earns; built as a pretty toy it evaporates. What pays is embedding AI into an **existing workflow** and proving it saves money or adds output.

In one line: **the further up the chain you go, the more you sell shovels and the more resilient you are; the further down, the more you mine gold — but you live and die by real use cases.**

## Why the "landlord" beats the "miner" more often

Every gold rush has the same lesson: **the people who reliably make big money sell the shovels, water, and tools to the prospectors.** AI's cycle is no exception — the most certain demand, independent of any single model's fate, sits with the infrastructure providers.

The three big clouds are also pulling a **strategic move: vertical integration.** They run their own labs, forge their own models, and sell their own APIs, bundling "cloud + model + API" into one invoice. That squeezes anyone trying to live on a single layer.

## Every layer collapses toward concentration

The most counterintuitive thing about this chain: **each layer is extremely concentrated.** The chip layer is nearly one company, the cloud layer is three, the flagship model layer is four names, and the open-weight layer is a few families rotating on top.

Concentration is not a coincidence; it is the physics of this industry:

* **Huge capital barrier.** Forging a top model costs a pricetag in the hundreds of millions (see `train-01-pretraining`); not every company can play.
* **Network effects.** Developers, frameworks, and ecosystems cling to the biggest names, so the more a player is used, the harder it is to displace.
* **Talent concentration.** The people who can build frontier models or orchestrate tens of thousands of GPUs are few, and they flow toward the most funded places.

So every layer is "a handful of players putting cards on the table." Knowing this, it stops being surprising why so few star startups exist and the news keeps circling the same names.

## Use "gross margin" as the ruler for each layer

Different layers are healthy by different measures. A practical ruler: **what does the gross-margin structure of this layer look like?**

| Layer | Main cost | Margin profile | Key metric |
|---|---|---|---|
| **Chips / GPUs** | fab, R\&D, capacity | high and stable (pricing power in shortage) | shipments, share, ecosystem lock |
| **Cloud / compute** | capex, power, ops | improves as scale amortizes | utilization, return on capital |
| **Model labs** | training + inference | high revenue, margin still to amortize | inference cost, retention, API revenue |
| **Apps / infra** | pass-through inference + acquisition | most divergent | gross margin × retention |

Remember: **margin tells you more than revenue about whether a layer can stand on its own.** Revenue is the show; margin is the constitution.

## Upstream integration, a recurring "value redistribution"

Zoom out and you see a recurring move: **the upstream integrates downward and pockets the downstream margin.**

Clouds run their own labs, sell their own models and APIs, bundling "compute + capability + pricing" into one package. Thin intermediaries that merely resell compute get squeezed flat by that package.

This is not a moral judgment; it is the direction of commercial gravity: **whoever sits closest to the final payer and holds an irreplaceable resource can keep pulling the chain's profit upward.** That is why the app layer must build a wall of use case and data — otherwise its margin gets siphoned upstream sooner or later.

## Chips and clouds aren't the whole story

But reducing the industry to "NVIDIA plus the clouds" skips the economic detail that really matters: **inference cost.** Once a model is trained, every answer actually burns electricity and GPU-time to compute. That is the denominator that decides whether an app-layer business can make money.

This article plants the seed: **the cost structure of running one answer decides how companies price themselves and whether they survive.** We will open that bill properly in `econ-01-inference-cost`.

> The one thing to remember: the AI value chain is a layered business where the upstream lives on scarce resources and the downstream lives on use cases. Every layer is really doing the same thing — packaging the "predict the next token" machine (llmcore-01) as a paid service for the layer below.

## One ten-dollar bill walks the whole chain

Talking abstractly about "money concentrating upstream" is weak; follow one bill. Suppose you pay $10 for one AI feature (illustrative numbers, not a real settlement); here is how the money flows layer by layer:

```text
you pay $10 → app company
↓ keeps $4 (packaging, support, acquisition)
app pays $6 → model API
↓ takes $3 (billing, platform)
lab / cloud gets $3 → pays $1.5 for GPUs and cloud services
↓
NVIDIA (chips) finally receives about $1.5
```

Three takeaways:

* **The further downstream, the earlier the money arrives — but the less stays.** The app collects first, yet pays support, acquisition, and API; what survives depends on margin.
* **The upstream collects less, but it is nearly "net."** The $1.5 NVIDIA receives is almost pure margin — that is exactly what "selling shovels" means.
* **The anchor of the whole chain is "whether you will pay that $10."** If end demand cracks, every layer's share shrinks at once.

## How to actually use this map

Knowing the chain isn't about memorizing names; it answers three practical questions:

1. **"Which layer do I sell in?"** Locate your business in the river first, then you know who your rivals and costs look like.
2. **"Who controls my costs?"** In the app layer your margin rides on API pricing and inference cost; in a lab, it rides on GPU access.
3. **"Will the upstream eat me?"** The clouds keep integrating downward, so a pure-play app needs its own moat — a real workflow and proprietary data.

## Next

The value chain is the skeleton; now we put names on it: exactly which companies sit in each link, and how they trade blows — `industry-02-players`.

#### Q: In the AI value chain, why is the profit structure of the upstream (chips, cloud) usually more stable than the downstream (apps)?

* Because the downstream makes no money at all

* Because upstream sells irreplaceable scarce resources with high demand certainty and pricing power

* Because upstream costs are nearly zero

* Because the downstream never charges

> 💡 The upstream provides infrastructure everyone needs: demand is certain, competition is concentrated, and pricing power is strong. The downstream must prove real savings or new output through a specific use case, so margins vary wildly.
