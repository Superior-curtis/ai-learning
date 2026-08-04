# Planning & Sub-Agents: Decomposing Tasks and Working in Parallel

> 📅 2026-08-04 · Deep Dive
> Planning is an agent's navigation system: from decomposing tasks and ordering steps to spawning sub-agents for parallel work — and when planning backfires.

---

## Opening: one task, two approaches

"Write a report on the AI market in 2026." Handed to an agent with no planning ability, it starts grinding from the first word, researching as it writes, and halfway through discovers earlier paragraphs are outdated and has to redo them — like driving without a map.

With planning first, everything changes: it lists what data it needs, assigns "who researches which part," decides the writing order, and reviews at the end. Same task, lower cost, steadier quality. **Planning is an agent's navigation system.**

***

## What planning actually does

Planning breaks a big goal into executable steps. The decomposition works at three levels:

| Level | Question it answers | Example |
|-------|--------------------|---------|
| Goal | What are we ultimately delivering | A 2026 market report |
| Steps | Which stages are needed | research → outline → write → review |
| Order | What runs first, what can run in parallel | research can parallelize; writing waits on data |

> Planning is essentially deferred decisions: instead of writing the first paragraph, you first think through how the whole thing should flow, then act. Time spent thinking is not wasted — it is rework avoided.

Where does a model's planning ability come from? Recall `llmcore-01-next-token` — as a model predicts the next token, it is also predicting the most sensible overall trajectory of the whole response. Give it a clear task and tools, and it can first produce a step list, then execute step by step.

***

## Planner-Executor: the classic planning pattern

Splitting "planning" and "execution" between two roles is the most common agent architecture:

* **Planner**: does the thinking, produces a step list (a plan).
* **Executor**: does the doing, executes each step in order and reports back.

```text
user request
│
▼
Planner → produces plan: step 1, 2, 3 …
│                     │
▼                     ▼
Executor runs step 1   Executor runs step 2  ← in parallel
│                     │
└──────────┬──────────┘
▼
merge results → report to user
```

Why split into two roles? Because "thinking" and "doing" optimize differently: the planner needs a global view, the executor needs focus on the present. Split, the planner can revise the plan as execution proceeds, and the executor never worries about "what comes next."

***

## Going parallel with sub-agents

The second step of planning is delegation. `agents-02-orchestration` already covered multi-agent orchestration; here we focus on one key decision: **when should you spawn sub-agents to work in parallel?**

### Three situations that favor parallelism

* **Truly independent work**: e.g. looking up financial reports for five companies, none related to another.
* **Clear role specialization**: research, writing, and review each do their own thing.
* **The main agent's context is tight**: outsourcing work frees up its tokens.

### Three situations that kill it

* **Steps with dependencies**: writing must wait for data; splitting is just waiting around.
* **Heavy shared state**: sub-agents each make decisions, and the contradictions never converge.
* **Tiny single steps**: the restart cost of a sub-agent exceeds just doing it yourself.

```text
Good split: research A, research B, research C → three sub-agents run at once → merge
Bad split: write paragraph 1 → sub-agent writes paragraph 2 → sub-agent writes paragraph 3
           (paragraphs must flow together; splitting makes it worse)
```

> Sub-agents are not accelerators; they are overhead: every spawn rebuilds context, reports back, and gets read by the main agent again. Only split when the parallelism payoff exceeds that overhead.

***

## A complete planning example

Say the task is "prepare a company pitch deck." The planner decomposes it like this:

```text
Plan:
  1. In parallel: research company background / collect market data / build competitor matrix
  2. In order: draft outline from the data → write body copy → review and polish
  3. Finish: produce the final deck file
```

At execution, the three research sub-agents start simultaneously; only when all three data sets are reported does writing begin. This is the hybrid "parallel first, sequential after" strategy — and the shape of most real-world agent applications.

```json
{
"goal": "build a company pitch deck",
"steps": [
  { "id": 1, "task": "research company background", "depends_on": [] },
  { "id": 2, "task": "collect market data",         "depends_on": [] },
  { "id": 3, "task": "build competitor matrix",     "depends_on": [] },
  { "id": 4, "task": "draft outline",               "depends_on": [1, 2, 3] },
  { "id": 5, "task": "write body copy",             "depends_on": [4] }
]
}
```

Writing the plan as data has one big payoff: **it becomes sortable**. Steps with an empty `depends_on` (1, 2, 3) have no dependencies and are parallel candidates; dependent steps (4, 5) must wait. The planner produces the plan, a scheduler sorts by dependency and delegates — this logic is the seed of the "task graph" in most agent frameworks.

***

## When planning backfires

Planning is no silver bullet. In three situations it actively hurts:

### 1. The task is trivial

"Translate this English sentence to Chinese." Planning only adds latency. **Planning has a cost** — every plan is extra tokens and time.

### 2. The environment changes too fast

Planning assumes the future is predictable. If the requirements shift mid-execution, a rigid plan goes stale; adapting as you go beats committing to a stale map.

### 3. Planning itself is unreliable

If the model is unfamiliar with the domain, its step list may simply be wrong — a wrong plan is worse than no plan, because it grants false confidence.

The rule in one line: **plan for predictable complex tasks, just do trivial ones, and for fast-moving tasks adapt as you go.**

***

## Next stop

Planning lets an agent think through what to do, but no matter how good the plan, execution still breaks down: sub-agents loop, wrong tools get picked, context balloons. The next article, `agents-08-reliability`, covers agent reliability and debugging — failure modes, guardrails and budgets, and how to retry and verify. Think well, act well, and you still need to **stay upright when things go wrong**.

***

## Key takeaways

* Planning = breaking a big goal into steps, at three levels: goal, steps, order.
* Planner-Executor separates "thinking" from "doing" — the classic architecture.
* Sub-agents are overhead; split only when the parallel payoff beats the cost.
* The good shape is a hybrid: parallel first, sequential after.
* Planning backfires when: the task is trivial, the environment shifts fast, or the plan itself is unreliable.

#### Q: When should you NOT spawn sub-agents for parallel work?

* When the work items are fully independent of each other

* When steps have dependencies and need the previous stage results

* When multiple independent data sources need researching

* When the main agent context is nearly full

> 💡 Dependent steps cannot truly parallelize — later steps must wait on earlier results, so splitting just adds sub-agent overhead for no gain; the other three (independent work, multi-source research, saving context) are exactly the cases that favor sub-agents.
