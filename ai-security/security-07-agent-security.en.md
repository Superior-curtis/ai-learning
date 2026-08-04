# Multi-Agent System Security: When AI Can Take Action

> 📅 2026-08-04 · Deep Dive
> When an agent calls tools, browses the web, and operates real systems, injection escalates from 'saying the wrong thing' to 'doing the wrong thing.' This post covers tool permission boundaries, human-in-the-loop, and least privilege for MCP tools.

---

Everything up to now has been about a model that "says the wrong thing." `security-06-guardrails` keeps a single model under control within a single inference. But the moment you give the model **hands** — tools, a browser, access to real systems — the stakes change.

A tool-calling agent (`agents-01-what-is-agent`) does not just answer your questions. It does things for you. And `security-01-prompt-injection` told you the model cannot reliably tell instructions from data. Put those two facts together and you get an uncomfortable conclusion — **once an agent can act, injection can become an unauthorized action.**

This post covers that added attack surface: how data flows into actions, where the tool permission boundary sits, why risky actions need a human in the loop, and least privilege for MCP tools.

## From "speaking" to "acting": the surface grows

Watch how the attack surface grows. An agent reads all kinds of **untrusted content** into context: web pages, RAG documents, email, JSON returned by tools, the output of other agents (see `agents-02-orchestration`). In a model that only speaks, that content can at most steer the answer. In an agent, **text in context can become the arguments of a tool call.**

```attack-surface
Only speaking: web content ─→ steers the "answer"
Calling tools:   web content ─→ steers the "action"

untrusted content (web / documents / tool output / sub-agent output)
      ↓
 enters context
      ↓
 becomes tool-call arguments (send email, write to DB, change settings...)
```

This is indirect injection upgraded to the agent setting: the "forward the email" scenario from `security-01-prompt-injection` is, here, a realistic daily path. So the first lesson of agent security: **treat every "data → action" path as a potential authorization boundary to design for, rather than assuming the model will judge the source.**

## Data flowing into actions: route tool calls through a checkpoint

Since untrusted content can become tool arguments, defense belongs at the "tool call" pivot — hard checks that do not rely on the model, before arguments take effect:

* **Allowlist**: only explicitly permitted tools can be called; everything else is refused.
* **Argument schema validation**: every tool defines argument types and ranges; non-conforming calls are blocked.
* **Data-flow tagging**: when arguments come from untrusted sources (web, documents, external API output), tag them as "unverified" and raise the chance of human review.
* **Sensitive-action blocking**: scan arguments for send, delete, transfer patterns, and hold before asking.

```python
def authorize_tool_call(tool: str, args: dict, source: str) -> Action:
    if tool not in TOOL_ALLOWLIST:
        raise PermissionError(f"Tool not in allowlist: {tool}")
    validated = TOOL_SCHEMAS[tool].validate(args)
    if source == "untrusted" and tool in SENSITIVE_TOOLS:
        return Action.NEEDS_HUMAN   # untrusted source + sensitive tool → human in the loop
    return Action.EXECUTE
```

The key is "source awareness": **the same tool call, from the user directly versus from web content, has a completely different risk profile.** Carrying a source tag into the authorization decision is a guardrail unique to agents; a traditional backend has no such concept.

## Tool permission boundaries

The more tools an agent has, the more windows it opens. A few firm rules — consistent with `security-04-secure-apps`, but stricter in the agent setting:

* **Minimal tool set**: expose only the tools the current task needs. If it can read, do not give write; if it can list, do not give delete.
* **Bind to the current user**: the agent acts as "the current user", not as a powerful service account.
* **Read-only by default**: unless the task explicitly needs writes, hand out read-only access.
* **Scope the data range**: even when querying is allowed, constrain it to that user's own data.

| Design decision | High risk | Low risk |
|---|---|---|
| Tool exposure | The whole tool library | Only the few tools the task needs |
| Database role | Admin account | Read-only, scoped role |
| Write actions | Auto-execute | Needs confirmation or human in the loop |
| Account identity | Shared service account | Bound to the current user |

## Risky actions: keep a human in the loop

Even the best checks leak. The final safety net is to **pause high-risk actions for a person** — human-in-the-loop. Judge "should we wait for a person" on three signals:

1. **Irreversible**: delete, overwrite, send, transfer — actions that cannot be undone if wrong.
2. **Wide blast radius**: actions that affect other users or system-level state.
3. **Untrusted source**: arguments that come from web pages, documents, or external tool output.

If any one matches, pause, show "the action about to run and its arguments", and wait for the user to confirm before executing. This is not sacrificing efficiency — it trades "the agent did something it should not" from "left to the model's judgment" to "left to a human decision."

> The first principle of agent security: do not hand "whether this should happen" entirely to the model. The model is a statistical next-token predictor over a single token stream (llmcore-01-next-token), and it cannot reliably tell "this instruction came from the system" from "this instruction came from a web page." So permissions and approvals must be enforced by code outside the model: allowlists, argument validation, source tags, and human confirmation for risky actions.

## Least privilege for MCP tools

When an agent connects to tools over MCP (`agents-03-mcp`), those principles have to land in **the permission configuration of every MCP server.** MCP lets you wire external tools into the agent, but it also outsources part of your attack surface to those tools' implementations — so permissions must be set the moment you connect:

* **Per-tool authorization**: a server may expose many tools; open only the ones this task actually uses, not all of them.
* **Scoped credentials**: each server uses its own, narrowly privileged credentials, not a shared master key.
* **Data-scope limits**: the data a server can reach is limited to this user's or this task's range.
* **Flag sensitive tools**: tools that write, send data out, or delete are marked high-risk and trigger human-in-the-loop.
* **Audit and monitor**: record every tool call's arguments and source, so incidents can be traced.

If you write your own MCP server, `mcp-03-build-your-own` walks through designing the tool inputs and permissions properly — security has to be built in the moment you expose a tool, not bolted on later.

## Sub-agent trust: do not let downstream follow the upstream

Multi-agent systems add one more unique attack surface: **the trust relationship between agents.** When a main agent hands tasks to sub-agents (see `agents-02-orchestration`), the sub-agents' output comes back into the main agent's context — and whatever the sub-agent read can steer the main agent.

* **Treat sub-agent output as untrusted content**: a sub-agent's output is like a web page — "data", not "the system reporting back". Run it through the same content isolation and tagging before it reaches the main agent (see `security-06-guardrails`).
* **Do not hand sub-agents inherited full power**: a sub-agent should get tools and permissions that are narrower than the main agent's, not the same or broader.
* **Constrain what a sub-agent can do**: the more single-purpose the sub-agent, the less damage it can do when it is steered — that itself is a form of least privilege.

In one line: **multi-agent makes the "data → action" paths more numerous, not safer.** Every cross-agent path still has to cross the same checkpoints.

## After the incident: reconstruct from the audit log

Even with every control in place, plan for "something will go wrong." The standard moves when an incident is suspected:

1. **Stop the bleeding**: disable the affected tool permissions, pause the agent, roll back the problematic settings.
2. **Forensics**: reconstruct "which data → which action" from the audit log — source tags and tool-call records are the lifeline here.
3. **Add tests**: turn this technique into an automated test case so it does not come back.
4. **Review**: which checkpoint failed to stop it — allowlist, argument validation, source tag, or human in the loop?

The key realization: **the audit log is not post-hoc decoration. It is the map for incident response.** Without records there is no reconstruction, and therefore no "never again."

## Designing security into the agent architecture

Finally, put the gates together and look at a secure agent pipeline:

```text
untrusted content → [content isolation] → context
↓
user instruction → [input filter] → agent → tool call
↓
[authz: allowlist + args + source]
↓
risky? ──yes──→ [human in the loop: wait]
↓no
[audit log] → outside world
```

Every "data → action" path crosses at least one check that does not depend on the model. This is not about giving up automation — it is about **automating only the safe actions.** What stays for humans is, always, the irreversible, the wide-reaching, and the untrusted-source actions.

## One-sentence summary

> Multi-agent system security is not "making the agent more obedient." It is **placing a code-enforced, not model-enforced, checkpoint on every path from data to action — allowlists, argument validation, source tags, least privilege, and human confirmation for risky actions.** The model's cleverness does the right thing; the framework's rules stop the wrong thing.

#### Q: Why should the same tool call be treated differently when the arguments come from the user versus from web content?

* Because web content uses a different argument format that needs converting first

* Because the two have different trust levels: the latter may be coaxing hidden in untrusted content, so a higher bar is needed before the action fires

* Because user input is faster and should skip all checks

* Because web content cannot be tool arguments, only plain-text references

> 💡 An agent reads untrusted content into context, and that text can become tool arguments. Instructions from the user carry some baseline of trust; arguments from sources like web pages may be planted coaxing. Splitting the two with a source tag, and raising the bar — or triggering human-in-the-loop — for sensitive actions from untrusted sources, is the core move of agent security.
