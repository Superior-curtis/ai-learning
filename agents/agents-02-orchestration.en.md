# Agent Orchestration: Single vs Multi-Agent

> 📅 2026-08-01 · Core Mechanics
> From sequential to hierarchical to swarm, meet the three mainstream orchestration patterns and learn when to split work across sub-agents.

---

## Why orchestration matters

A single agent can do a lot, but some tasks are naturally about division of labor: research needs to search, writing needs to turn the research into prose, review needs to catch errors. Rather than stuffing every capability into one agent, it often pays to **coordinate multiple specialized agents** — and how you organize them is orchestration.

Orchestration answers three questions:

* Who goes first, who goes next? (flow order)
* Who directs, who executes? (authority)
* How does the result reach the next step? (handoff)

***

## Three basic architectures

### 1. Sequential

Agents run one after another; each one's output is the next one's input.

```
researcher → writer → reviewer
```

Best for fixed pipelines with clear steps, such as "search → write → proofread". The upside is simplicity and predictability; the downside is rigidity — one failed step stalls the whole chain.

### 2. Hierarchical

A coordinator breaks the task into pieces, dispatches them to sub-agents, then merges the results.

```
             coordinator
              /        \
        researcher     writer
             \        /
              merged
```

Best for tasks that change and need dynamic decisions about "who does what". The coordinator decides the division of labor; sub-agents focus on execution.

### 3. Swarm

A group of peer agents hands work to each other via **handoffs**, with no fixed commander. It fits scenarios like customer support, where the active role switches dynamically.

```
  A ──handoff──▶ B ──handoff──▶ C
  ▲                             │
  └─────────handoff─────────────┘
```

> Swarm is the most flexible but also the hardest to debug: the flow of work is dynamic and hard to predict or reproduce.

***

## Three framework examples

### LangGraph: model the flow as a graph

LangGraph models orchestration as a **state graph**: nodes are processing steps, edges are flow, and state flows between nodes.

```python
from typing import TypedDict
from langgraph.graph import StateGraph, START, END

class State(TypedDict):
    query: str
    draft: str
    review: str

def draft(state: State) -> dict:
    return {"draft": f"draft: {state['query']}"}

def review(state: State) -> dict:
    return {"review": f"review passed: {state['draft']}"}

graph = StateGraph(State)
graph.add_node("draft", draft)
graph.add_node("review", review)
graph.add_edge(START, "draft")
graph.add_edge("draft", "review")
graph.add_edge("review", END)

app = graph.compile()
result = app.invoke({"query": "write a product press release"})
```

### CrewAI: model the work as a team

CrewAI uses "role + task + process": every agent has a role, every task names who owns it, and the Crew decides the flow.

```python
from crewai import Agent, Task, Crew, Process

researcher = Agent(
    role="researcher",
    goal="gather accurate market data",
    backstory="focused on data and facts",
)

writer = Agent(
    role="writer",
    goal="turn the research into a readable article",
    backstory="skilled at clear communication",
)

research_task = Task(
    description="Summarize the AI agent market size in 2026",
    expected_output="A summary with data sources",
    agent=researcher,
)

write_task = Task(
    description="Turn the research summary into an 800-word article",
    expected_output="A complete article",
    agent=writer,
)

crew = Crew(
    agents=[researcher, writer],
    tasks=[research_task, write_task],
    process=Process.sequential,  # or Process.hierarchical
)

result = crew.kickoff()
```

### AutoGen: collaborate as a conversation team

AutoGen's modern `autogen-agentchat` turns multiple agents into a "team" whose members take turns speaking until the task completes.

```python
from autogen_agentchat.agents import AssistantAgent
from autogen_agentchat.teams import RoundRobinGroupChat
from autogen_ext.models.openai import OpenAIChatCompletionClient

model_client = OpenAIChatCompletionClient(
    model="gpt-4o",
    api_key="your-key",
)

writer = AssistantAgent(
    name="writer",
    system_message="You draft the article.",
    model_client=model_client,
)
reviewer = AssistantAgent(
    name="reviewer",
    system_message="You review the article and propose edits.",
    model_client=model_client,
)

team = RoundRobinGroupChat([writer, reviewer], max_turns=6)
await team.run(task="Write a short essay about AI agents")
```

***

## When should you use sub-agents?

Sub-agents have a cost: they re-establish context, re-explore, report back, and the main agent has to read their reports. Use them only when:

* The work is genuinely **parallelizable** — for example, reviewing 20 files at once, each independent.
* The task needs **distinct specialist roles** — research, writing, and review are independent concerns.
* The main agent's context is near its limit — outsourcing part of the work saves the main loop's tokens.

Conversely, if the main agent can finish something in a few tool calls, do not split it — an extra layer of agents only makes things slower and more expensive.

***

## Common pitfalls

* **Over-decomposition** — spawning sub-agents for trivial tasks adds cost and latency for nothing.
* **Ignoring handoff** — sub-agent results lack a clear format, so the coordinator cannot read them.
* **No cap** — sub-agents can keep spawning, and so does the token bill.
* **Shared state** — sub-agents have no channel to communicate and make contradictory decisions.

***

## Key takeaways

* Orchestration answers three questions: order, authority, handoff.
* Three basic patterns: sequential, hierarchical, swarm.
* LangGraph uses graphs, CrewAI uses roles and tasks, AutoGen uses team conversation.
* Sub-agents have real costs; use them for parallelism, specialization, or saving context.
* Start with a single agent, and split only when you need to.
