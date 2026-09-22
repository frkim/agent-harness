# Harness evaluation

[Back to the guide](../README.md)

*Independent buyer's guide · 2026-09-22*

## Which harness fits your job?

Compare capabilities, understand the trade-offs, and choose the layer you
actually need.

- [Scoring rubric](#the-rubric)
- [Scorecard](#at-a-glance)
- [Evaluations](#evaluation-cards)
- [Decision tree](#start-with-your-requirements)
- [Capability quadrant](#capability-quadrant)
- [Before adopting](#before-adopting-turn-a-shortlist-into-evidence)

## A shortlist, not a universal leaderboard

This page covers the guide's implementation chapters and the
[dedicated framework and coding-harness sections](examples.md#dedicated-implementation-coverage).
Deep Agents, Google ADK, OpenAI Agents SDK, OpenHands, and Strands expand the
original shortlist; additional comparisons cover Gemini CLI, OpenCode, Pi,
Pydantic AI / Pydantic AI Harness, and CrewAI. Inclusion and ordering are
editorial priorities, not a measured popularity ranking or an exhaustive market
survey.

> **Editorial assessment, not benchmark results.** Scores reflect documented
> capabilities and integration effort for a bounded, tool-using agent as of
> September 22, 2026. No head-to-head runtime tests were performed. The
> evidence is the linked implementation chapters and official documentation;
> no release versions were pinned. Recheck your selected version, plan,
> region, and deployment. Ratings are not user reviews, GitHub star counts,
> security certifications, or a guarantee of task success.

Coding runtimes, application-agent frameworks, and managed hosting services are
**different layers**, compared in separate groups below. OpenHands SDK belongs
with programmable coding runtimes; Copilot is a hosted coding agent, not a
general-purpose hosting service. A lower total can be the better fit.
Microsoft's rating below covers
**Foundry Agent Service only**, not the separate Agent Framework. Cost, model
quality, latency, licensing, and data residency are excluded from scores
because no comparable measurements were collected; treat them as adoption
gates.

## The rubric

Each dimension uses the same anchored 1–5 scale: **1** = mostly
application-built; **2** = partial support or substantial gaps; **3** = useful
primitives with integration work; **4** = strong built-in support with
configuration; **5** = especially comprehensive support for that dimension.
These are judgments about supplied mechanisms, not measured reliability.
Unknown capabilities must be marked "not assessed," not scored as absent.

*Default weights for a bounded agent with human oversight*

| Dimension | Weight | What earns credit |
| --- | --- | --- |
| Control (C) | 25% | Scoped tools, permission or approval mechanisms, and explicit execution boundaries. Not a security audit. |
| Recovery (R) | 25% | Durable state, restart/resume, checkpoints, and documented replay behavior. Conversation history alone is not workflow recovery. |
| Extensibility (E) | 20% | Programmable tools, workflows, adapters, and replaceable components. Not a promise of universal model compatibility. |
| Visibility (V) | 15% | Inspectable tool activity, events, logs, and reviewable artifacts. Not independently verified correctness. |
| Setup ease (S) | 15% | Less application and infrastructure work to start the intended use case safely; higher means easier. |

**Final score / 100 = (25C + 25R + 20E + 15V + 15S) / 5.** Stars / 5 = final
score / 20, with fractional star fills and an exact numeric label. Example:
Claude = (25×4 + 25×3 + 20×4 + 15×4 + 15×4) / 5 = **75/100 = 3.75/5 stars**.
All five dimensions are assessed here. Do not compute a comparable total when a
dimension is unassessed. Change the weights for your own requirements before
choosing a winner.

## At a glance

*Editorial scorecard · C / R / E / V / S refer to the rubric above*

### Coding runtimes and toolkits

| Harness / layer | C | R | E | V | S | Final | Stars / 5 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| [Claude Code / Agent SDK](#claude-code--agent-sdk) · coding runtime | 4 | 3 | 4 | 4 | 4 | 75/100 | 3.75 |
| [OpenAI Codex](#openai-codex) · coding runtime | 4 | 3 | 4 | 4 | 4 | 75/100 | 3.75 |
| [GitHub Copilot cloud agent](#github-copilot-cloud-agent) · hosted coding agent | 4 | 2 | 3 | 4 | 5 | 69/100 | 3.45 |
| [DeepSeek Harness](#deepseek-harness) · preview runtime | 2 | 2 | 5 | 3 | 2 | 55/100 | 2.75 |
| [OpenHands Software Agent SDK](#openhands-software-agent-sdk) · programmable coding runtime | 4 | 3 | 5 | 4 | 3 | 76/100 | 3.80 |
| [Gemini CLI](#gemini-cli) · coding runtime | 4 | 3 | 4 | 4 | 4 | 75/100 | 3.75 |
| [OpenCode](#opencode) · multi-provider coding runtime | 2 | 3 | 5 | 4 | 4 | 69/100 | 3.45 |
| [Pi](#pi) · coding toolkit/runtime | 2 | 2 | 5 | 3 | 3 | 58/100 | 2.90 |

### Application-agent frameworks

| Framework / layer | C | R | E | V | S | Final | Stars / 5 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| [LangGraph](#langgraph) · orchestration library | 3 | 5 | 5 | 4 | 2 | 78/100 | 3.90 |
| [LangChain Deep Agents](#langchain-deep-agents) · packaged harness on LangGraph | 4 | 4 | 5 | 4 | 3 | 81/100 | 4.05 |
| [Google ADK](#google-adk) · agent/workflow framework | 3 | 4 | 5 | 4 | 3 | 76/100 | 3.80 |
| [OpenAI Agents SDK](#openai-agents-sdk) · application-agent framework | 4 | 3 | 5 | 5 | 4 | 82/100 | 4.10 |
| [Strands Agents](#strands-agents) · model-driven agent SDK | 3 | 3 | 5 | 4 | 3 | 71/100 | 3.55 |
| [Pydantic AI](#pydantic-ai-and-pydantic-ai-harness) · typed agent foundation | 3 | 3 | 5 | 4 | 3 | 71/100 | 3.55 |
| [Pydantic AI Harness](#pydantic-ai-and-pydantic-ai-harness) · capability library, Coder configuration | 2 | 3 | 5 | 4 | 2 | 63/100 | 3.15 |
| [CrewAI](#crewai) · crews and flows | 3 | 4 | 4 | 4 | 3 | 72/100 | 3.60 |

### Managed hosting services

| Service / layer | C | R | E | V | S | Final | Stars / 5 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| [Microsoft Foundry Agent Service](#microsoft-foundry-agent-service) · managed service | 4 | 4 | 3 | 4 | 3 | 73/100 | 3.65 |

**Compare within the layer you need.** A framework's extensibility does not
make it a substitute for a ready-made coding environment or a managed service.
These weights do not establish a cross-layer winner; use the decision tree to
shortlist by task.

Recovery scores credit the configured mechanisms described in each card, not
default installation behavior. Pydantic AI and its Harness are separate rows:
the latter assesses the Coder capability stack with step persistence, not a
second score for the base framework or a managed coding service. Opt-in durable
engines and hosting products must be assessed separately from their SDKs.

## Evaluation cards

The dimension-by-dimension rationale explains each score. Sources describe
mechanisms; they do not endorse our ratings.

### Claude Code / Agent SDK

*CODING RUNTIME · 75 / 100 · ★★★★☆ (3.75 / 5 stars)*

**Choose for:** coding automation with programmable tool permissions and
hooks.

**Why these scores:** C4: tool permission callbacks and pre-tool hooks; R3:
resumable transcripts, not workspace recovery; E4: SDK and tool hooks; V4:
hooks and session history expose activity; S4: an existing coding loop reduces
assembly work.

**Trade-off:** permissions do not replace OS isolation. Callback behavior
depends on permission mode; restored conversation state does not undo file
changes.

**Validate:** deny a write through a hook, then resume a session and inspect
the actual workspace.

[Implementation and verification](claude.md) ·
[Official SDK reference](https://platform.claude.com/docs/en/agent-sdk/python)

### OpenAI Codex

*CODING RUNTIME · 75 / 100 · ★★★★☆ (3.75 / 5 stars)*

**Choose for:** CLI-driven repository tasks or an application embedding coding
sessions.

**Why these scores:** C4: separate sandbox and approval controls; R3:
persisted threads, not workspace rollback; E4: TypeScript SDK and App Server;
V4: structured execution events; S4: ready-made CLI loop with integration
choices.

**Trade-off:** a noninteractive SDK run is not automatically an interactive
approval UI. App Server supports approval exchanges; minimize inherited
environment credentials.

**Validate:** reject an out-of-workspace write and confirm a resumed thread's
diff and test evidence.

[Implementation and verification](codex.md) ·
[Official security guide](https://developers.openai.com/codex/security) ·
[App Server](https://developers.openai.com/codex/app-server)

### GitHub Copilot cloud agent

*HOSTED CODING AGENT · 69 / 100 · ★★★★☆ (3.45 / 5 stars)*

**Choose for:** delegating bounded GitHub repository work with a reviewable
branch or pull request.

**Why these scores:** C4: bounded ephemeral development environment and review
workflow; R2: persisted changes and session iteration are not general workflow
checkpoints; E3: repository setup and agent customization within a hosted
workflow; V4: diffs, commits, and logs; S5: managed task execution avoids
building a runtime.

**Trade-off:** optimized for GitHub development, not a general application
runtime. A pull request is not proof of correctness, and plan access must be
checked.

**Validate:** delegate a small patch, inspect changed files and test logs, and
require human review.

[Guide context](examples.md#existing-projects-to-study) ·
[Official overview](https://docs.github.com/en/copilot/concepts/agents/cloud-agent/about-cloud-agent) ·
[Responsible use](https://docs.github.com/en/copilot/responsible-use/agents)

### LangGraph

*ORCHESTRATION LIBRARY · 78 / 100 · ★★★★☆ (3.90 / 5 stars)*

**Choose for:** custom stateful workflows with branching, durable checkpoints,
and human pauses.

**Why these scores:** C3: interrupts require application authorization; R5:
explicit checkpoints, durability modes, and replay semantics; E5:
application-defined graph, state, and tools; V4: inspectable persisted
workflow state; S2: you assemble storage, tools, execution isolation, and
policy.

**Trade-off:** an interrupt is not an approval policy. Resuming can rerun node
code; external side effects need idempotency. An in-memory checkpointer is not
durable.

**Validate:** restart at an approval pause, reject the action, then exercise
crash recovery without duplicate writes.

[Implementation and verification](langgraph.md) ·
[Official persistence guide](https://docs.langchain.com/oss/python/langgraph/persistence) ·
[Interrupts](https://docs.langchain.com/oss/python/langgraph/interrupts)

### Microsoft Foundry Agent Service

*MANAGED SERVICE · 73 / 100 · ★★★★☆ (3.65 / 5 stars)*

**Choose for:** Azure-hosted agents when managed compute and service
integration are requirements.

**Why these scores:** C4: hosted sandbox boundaries and configurable MCP
approvals; R4: persisted session files and conversation state, with separate
opt-in resilience; E3: prompt and custom-code hosted agents within service
constraints; V4: stored conversation/response records; S3: hosting is managed,
but Azure and policy setup remain.

**Trade-off:** long-running resilience is preview, not an assumed default.
File restoration, conversation history, and framework checkpoints are
separate. Agent Framework alone does not inherit hosted service controls.

**Validate:** reject an MCP action, then test recovery under your chosen
hosting and resilience settings.

[Implementation and verification](microsoft.md) ·
[Official hosting guide](https://github.com/MicrosoftDocs/azure-ai-docs/blob/main/articles/foundry/agents/concepts/hosted-agents.md) ·
[Preview resilience](https://github.com/MicrosoftDocs/azure-ai-docs/blob/main/articles/foundry/agents/concepts/long-running-agent-resilience.md)

### DeepSeek Harness

*EXPERIMENTAL RUNTIME · DEVELOPER PREVIEW · 55 / 100 · ★★★☆☆ (2.75 / 5 stars)*

**Choose for:** isolated research into replaceable agent loops, tools, and
model adapters.

**Why these scores:** C2: profile-dependent providers and an unaudited
preview; R2: append-only history with documented interruption gaps; E5:
replaceable Cordis plugins; V3: session events, with live/durable differences;
S2: profile review and independently enforced isolation add setup work.

**Trade-off:** the official safety notice says it must not be treated as
secure or production-ready. This is the harness runtime, not simply the
DeepSeek model API.

**Validate:** use a disposable, credential-free workspace; test denied actions
and interruption around a mock write.

[Implementation and verification](deepseek.md) ·
[Official architecture](https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/architecture.md) ·
[Safety notice](https://github.com/deepseek-ai/deepseek-harness/blob/master/SAFETY.md)

### LangChain Deep Agents

*APPLICATION HARNESS · 81 / 100 · 4.05 / 5 stars*

**Choose for:** a supplied planning, filesystem, context-management, and subagent
loop rather than assembling those capabilities directly on LangGraph.

**Why these scores:** C4: named-tool interrupts and subagent approval settings;
R4: LangGraph-backed checkpoints with configured durable storage, assessed here
for the packaged loop rather than all custom graph recovery features; E5:
replaceable tools, backends, and subagents; V4: graph/tool inspection and tracing;
S3: the loop is supplied, but storage, policy, and isolation still need setup.

**Trade-off:** Deep Agents and LangGraph are different layers. Cross-thread
memory is not an execution checkpoint, and filesystem tools are not a sandbox.

**Validate:** restart at an approval pause, reject a write, and repeat through a
subagent. Check external-effect deduplication independently.

[Implementation, exercise, and official sources](examples.md#langchain-deep-agents)

### Google ADK

*APPLICATION FRAMEWORK · 76 / 100 · 3.80 / 5 stars*

**Choose for:** general-purpose tools, agent hierarchies, workflow agents, and
artifact services—not merely coding assistance.

**Why these scores:** C3: callbacks and experimental confirmation with backend
compatibility constraints; R4: opt-in resumable workflows with persistent
sessions and documented at-least-once tool semantics; E5: custom agents, tools,
and services; V4: OpenTelemetry workflow/model/tool traces; S3: service selection
and production deployment remain application work.

**Trade-off:** the reviewed confirmation docs exclude some persistent session
backends. Do not infer durable approvals from separate persistence and
confirmation features, or treat `adk web` as production hosting.

**Validate:** resume an interrupted workflow with the original invocation ID,
check idempotency, and test approval/storage compatibility separately.

[Implementation, exercise, and official sources](examples.md#google-adk)

### OpenAI Agents SDK

*APPLICATION FRAMEWORK · 82 / 100 · 4.10 / 5 stars*

**Choose for:** application-agent handoffs, tool approvals, guardrails, sessions,
and integrated tracing. Choose Codex separately for a ready-made coding loop.

**Why these scores:** C4: supported-tool approval interruptions and guardrails;
R3: sessions and serialized approval-paused `RunState`, not arbitrary mid-tool
recovery; E5: programmable tools, agents, and handoffs; V5: built-in tracing of
models, tools, handoffs, and guardrails; S4: compact runner and supplied session
and tracing mechanisms, though application tools still require policy.

**Trade-off:** input/output guardrails do not authorize every intermediate
action. Resume state needs trusted storage; traces need data-handling controls.

**Validate:** persist a pending approval, restart, reject it, and reject duplicate
resume requests without executing the action.

[Implementation, exercise, and official sources](examples.md#openai-agents-sdk)

### Strands Agents

*APPLICATION FRAMEWORK · 71 / 100 · 3.55 / 5 stars*

**Choose for:** a model-driven tool loop with multi-agent composition rather than
requiring every transition to be graph-defined.

**Why these scores:** C3: tool hooks/interrupts with concurrency and direct-call
boundaries; R3: stored history/state, approval interruptions, and explicit
snapshots, not general mid-tool replay; E5: providers, tools, and multi-agent
patterns; V4: OpenTelemetry agent/model/tool traces; S3: persistence, policy, and
deployment need deliberate integration.

**Trade-off:** pausing one tool does not stop other concurrent tools. Direct
tool calls do not acquire interrupt-based approval automatically.

**Validate:** pause a mock write beside a harmless read, reconstruct the session,
and reject the write using its original interrupt ID.

[Implementation, exercise, and official sources](examples.md#strands-agents)

### Pydantic AI and Pydantic AI Harness

*APPLICATION FOUNDATION: 71 / 100 · 3.55 / 5 stars*

*HARNESS CAPABILITY STACK: 63 / 100 · 3.15 / 5 stars*

**Choose for:** typed tools, dependencies, and outputs with Pydantic AI; add
Harness capabilities when filesystem/shell/context facilities are needed.

**Why these scores—Pydantic AI:** C3: typed interfaces and deferred approval,
with application-owned authorization/isolation; R3: stored history and deferred
continuation, with durable engines integrated separately; E5: tools, providers,
and dependencies; V4: optional OpenTelemetry instrumentation; S3: application
tooling, storage, and execution policy must be assembled.

**Why these scores—Harness:** C2: Coder's host shell has no command allowlist and
shell guardrails are not isolation; R3: continuable step snapshots and a
tool-effect ledger, not full workflow/workspace recovery; E5: composable
capabilities; V4: step events plus underlying model/tool tracing; S2: containment,
reconciliation, and a separate 0.x API stability policy add integration work.

**Trade-off:** conversation storage, step persistence, cross-conversation memory,
and durable execution are distinct. Base-framework guarantees must not be
silently extended to every Harness capability.

**Validate:** interrupt after a mock external effect, inspect unresolved tool
records, and reconcile before continuation. Test the chosen durable engine
separately from history restoration.

[Implementation, exercise, and official sources](examples.md#pydantic-ai-and-pydantic-ai-harness)

### CrewAI

*APPLICATION FRAMEWORK · 72 / 100 · 3.60 / 5 stars*

**Choose for:** role-based crews plus explicit flow control over delegated work.

**Why these scores:** C3: scoped tool lists and blocking hooks, with documented
hook-error caveats; R4: execution checkpoints and restore that skips completed
tasks, with best-effort automatic writes; E4: agents, tasks, tools, and flows;
V4: task/tool/model tracing with configured AMP integration; S3: role/flow design,
storage, and policy remain application work.

**Trade-off:** memory is not checkpointing. Ordinary hook exceptions can fail
open; safe code execution does not isolate all custom tools. Version-pin the
checkpoint behavior described in the linked documentation.

**Validate:** restore after task one, verify it is skipped, and repeat with
storage unavailable. Verify policy failure cannot permit a protected tool.

[Implementation, exercise, and official sources](examples.md#crewai)

### OpenHands Software Agent SDK

*CODING RUNTIME · 76 / 100 · 3.80 / 5 stars*

**Choose for:** programmable software-development agents with selectable
execution workspaces, rather than a general application workflow framework.

**Why these scores:** C4: confirmation policies and configurable workspace
boundaries; R3: persisted conversation events and execution/base state, with
workspace/process reconciliation still required; E5: custom tools, workspaces,
and Agent Server integration; V4: event callbacks, usage metrics, and OTLP
traces; S3: runtime, persistence, and isolation need configuration.

**Trade-off:** this is the SDK, not the hosted OpenHands product. A local
workspace is not a sandbox; confirmation does not restore a lost workspace.

**Validate:** reconstruct an approval-paused conversation and reject the edit.
Separately interrupt a shell command and reconcile actual process/file state.

[Implementation, exercise, and official sources](examples.md#openhands-software-agent-sdk)

### Gemini CLI

*CODING RUNTIME · 75 / 100 · 3.75 / 5 stars*

**Choose for:** a Google coding CLI with shell/file tools, hooks, headless events,
and explicit sandbox configuration.

**Why these scores:** C4: tool policy and sandbox choices; R3: resumable sessions
plus opt-in local file checkpoints, not durable external-effect recovery; E4:
hooks, tools, and headless interfaces; V4: structured tool events and telemetry;
S4: supplied coding loop, with policy/sandbox setup still necessary.

**Trade-off:** checkpointing is off by default. Headless approval-required tools
are denied, and sandbox profiles do not all impose the same access restrictions.

**Validate:** test headless denial, conversation resume, and checkpoint file
restore independently; confirm remote effects require separate reconciliation.

[Implementation, exercise, and official sources](examples.md#gemini-cli)

### OpenCode

*CODING RUNTIME · 69 / 100 · 3.45 / 5 stars*

**Choose for:** provider choice and typed client/server SDK integration.

**Why these scores:** C2: permission rules are explicitly not security
isolation, and server authentication requires configuration; R3: persisted
sessions and Git-dependent undo/redo, not workflow replay; E5: multi-provider
configuration and programmable SDK; V4: session/message APIs and event streams;
S4: supplied coding runtime, with server security and containment still required.

**Trade-off:** defaults are permissive. Authenticate exposed servers and isolate
execution independently; external MCP servers add their own trust boundary.

**Validate:** reject an unauthenticated request, deny a tool, interrupt a mock
write, and reconcile events with effects before retrying.

[Implementation, exercise, and official sources](examples.md#opencode)

### Pi

*CODING TOOLKIT / RUNTIME · 58 / 100 · 2.90 / 5 stars*

**Choose for:** a small composable model/agent/coding stack with SDK and RPC
integration, accepting more application-owned controls.

**Why these scores:** C2: project trust is not tool confinement and no built-in
sandbox is supplied; R2: saved session trees support conversation continuation,
not file rollback or workflow checkpoints; E5: tools, extensions, SDK, and RPC;
V3: structured execution events and session usage, with monitoring assembled by
the application; S3: the coding loop is supplied, but policy/isolation are not.

**Trade-off:** extensions run with process privileges. A low total here reflects
deliberately unbundled controls, not measured coding quality.

**Validate:** resume after interruption inside an isolated environment, inspect
actual files/processes, and confirm history forks do not restore the workspace.

[Implementation, exercise, and official sources](examples.md#pi)

## Start with your requirements

Follow the matching branch, then apply the adoption gates below. Branches may
overlap: a coding harness can be one bounded tool inside a larger workflow.

1. **Is the primary output a repository change?**
   - **Yes → Want work delegated inside GitHub with branch/PR review?**
     - Yes → [Copilot cloud agent](#github-copilot-cloud-agent); check plan
       access and repository policies.
     - No → Need a local or embedded coding loop?
       - Permission callbacks and pre-tool hooks are central →
         [Claude Code / Agent SDK](#claude-code--agent-sdk).
       - CLI automation, structured events, or a bidirectional approval
         integration → [Codex](#openai-codex). Use App Server for approval
         exchanges; define denied-action behavior for noninteractive runs.
       - Google coding CLI with hooks and headless execution →
         [Gemini CLI](#gemini-cli); configure sandboxing explicitly.
       - Multi-provider coding runtime with client/server embedding →
         [OpenCode](#opencode).
       - Minimal loop and extension-oriented toolkit → [Pi](#pi); budget
         for application-owned controls.
       - Programmable software-development agents with execution workspaces →
         [OpenHands SDK](#openhands-software-agent-sdk).
       - Several fit → trial them with the same tasks, models where
         comparable, budgets, and acceptance checks. Provider choice is an
         adoption gate, not evidence of equivalent model behavior.
   - **No → 2. Is Azure-managed hosting a requirement?**
     - Yes → [Foundry Agent Service](#microsoft-foundry-agent-service);
       confirm region, data handling, and required feature maturity. Add
       [Agent Framework](microsoft.md) if you need code-first orchestration;
       it is a separate choice.
     - No → **3. Which application-agent abstraction do you need?**
       - Explicit branching, durable state, and human checkpoints →
         [LangGraph](#langgraph), with a durable checkpointer and
         application-owned authorization.
       - A packaged planning/filesystem/subagent harness rather than building
         directly on that graph → [Deep Agents](#langchain-deep-agents).
       - Agent hierarchies, workflow agents, and artifact services →
         [Google ADK](#google-adk).
       - Handoffs, guardrails, sessions, and tracing for application agents →
         [OpenAI Agents SDK](#openai-agents-sdk), not Codex.
       - A model-driven tool loop with multi-agent composition →
         [Strands Agents](#strands-agents).
       - Typed tools and outputs, with an explicit durable-execution
         integration when needed → [Pydantic AI](#pydantic-ai-and-pydantic-ai-harness);
         evaluate its Harness separately.
       - Role-based delegation plus explicitly controlled workflows →
         [CrewAI](#crewai).
       - None → **4. Is this isolated research requiring replaceable runtime
         internals?** If preview risk is acceptable, consider
         [DeepSeek Harness](#deepseek-harness), not for production trust
         boundaries. Otherwise start with a
         [small bounded tool loop](execution.md#bounded-controller-sketch).
         Do not adopt a larger harness just because it scores well.

<details>
<summary>Hard requirements override every recommendation</summary>

Reject any option that cannot meet your data residency, licensing,
model/provider, network, identity, or deployment constraints. For strict
self-hosting, evaluate the actual runtime and model endpoints separately:
running a CLI locally does not imply local inference. For regulated or
high-impact work, require independent isolation, scoped credentials, audit
retention, and authenticated approvals. None of these ratings establishes
compliance.

For budget or speed requirements, measure cost per accepted task, latency, and
human review time on your workload. The scores contain no pricing or
performance comparison.

</details>

## Capability quadrant

A Gartner-style four-quadrant layout, using our own rubric axes. **This is
independent editorial analysis, not a Gartner Magic Quadrant, Gartner
research, or an endorsement.** Position represents capability fit, not market
share or measured execution quality.

The tables place extensibility (horizontal, 1–5) against recovery support
(vertical, 1–5), with quadrant boundaries at 3.5 on each axis. Read positions
**within each comparison group**, not as interchangeable product choices:

| Quadrant | Extensibility | Recovery |
| --- | --- | --- |
| Durable specialists | < 3.5 | ≥ 3.5 |
| Composable workflows | ≥ 3.5 | ≥ 3.5 |
| Focused runtimes | < 3.5 | < 3.5 |
| Flexible foundations | ≥ 3.5 | < 3.5 |

| Comparison group | Quadrant | Options at (E, R) |
| --- | --- | --- |
| Coding runtimes/toolkits | Focused runtimes | Copilot (3, 2) |
| Coding runtimes/toolkits | Flexible foundations | Claude, Codex, Gemini CLI (4, 3); OpenHands SDK, OpenCode (5, 3); DeepSeek, Pi (5, 2) |
| Application-agent frameworks | Composable workflows | LangGraph (5, 5); Deep Agents, Google ADK (5, 4); CrewAI (4, 4) |
| Application-agent frameworks | Flexible foundations | OpenAI Agents SDK, Strands, Pydantic AI, Pydantic AI Harness (5, 3) |
| Managed hosting services | Durable specialists | Foundry (3, 4) |

Positions are exactly (E, R) from the scorecard, on 1–5 scales; boundaries are
at 3.5. Options with equal values share a point rather than being artificially
separated; unoccupied quadrants are omitted from the grouped table. Quadrant
names describe relative strengths, not quality tiers.
Control, visibility, and setup ease affect the final score but are not shown
in this two-dimensional view.

## Before adopting: turn a shortlist into evidence

1. **Pin the setup:** product/runtime revision, model, tools, policies,
   storage, deployment, and budget. Revisit source links and preview
   warnings; this page is a dated snapshot.
2. **Run representative tasks:** include ordinary success, ambiguity, denied
   tool use, malicious retrieved instructions, and budget exhaustion. Use
   independent acceptance checks.
3. **Exercise recovery:** restart during a pending approval and around a mock
   external write. Confirm no unauthorized or duplicate side effects. Session
   resume is not rollback.
4. **Compare outcomes:** accepted-task rate, unauthorized effects, human
   intervention, cost per accepted task, and latency—not just attractive
   output. Record failures and unavailable checks.
5. **Reweight and decide:** prioritize your hard requirements, run a limited
   pilot, and expand permissions only after the controls work.

Continue with [safety and independent verification](safety-and-verification.md)
and the [implementation checklist](operations.md#implementation-checklist).

---

Part of the [Agent Harness design guide](../README.md). Product names belong
to their respective owners.
