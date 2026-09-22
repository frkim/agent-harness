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

This page covers widely used coding and orchestration options already discussed
in this guide: Claude Code / Agent SDK, OpenAI Codex, GitHub Copilot cloud
agent, LangGraph, and Microsoft Foundry Agent Service. DeepSeek Harness is
included as an emerging experimental alternative. Inclusion is editorial, not a
measured popularity ranking or an exhaustive market survey.

> **Editorial assessment, not benchmark results.** Scores reflect documented
> capabilities and integration effort for a bounded, tool-using agent as of
> September 22, 2026. No head-to-head runtime tests were performed. The
> evidence is the linked implementation chapters and official documentation;
> no release versions were pinned. Recheck your selected version, plan,
> region, and deployment. Ratings are not user reviews, GitHub star counts,
> security certifications, or a guarantee of task success.

Coding agents, orchestration libraries, and managed services are **different
layers**. A lower total can be the better fit. Microsoft's rating below covers
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

| Harness / layer | C | R | E | V | S | Final | Stars / 5 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| [Claude Code / Agent SDK](#claude-code--agent-sdk) · coding runtime | 4 | 3 | 4 | 4 | 4 | 75/100 | 3.75 |
| [OpenAI Codex](#openai-codex) · coding runtime | 4 | 3 | 4 | 4 | 4 | 75/100 | 3.75 |
| [GitHub Copilot cloud agent](#github-copilot-cloud-agent) · hosted coding agent | 4 | 2 | 3 | 4 | 5 | 69/100 | 3.45 |
| [LangGraph](#langgraph) · orchestration library | 3 | 5 | 5 | 4 | 2 | 78/100 | 3.90 |
| [Microsoft Foundry Agent Service](#microsoft-foundry-agent-service) · managed service | 4 | 4 | 3 | 4 | 3 | 73/100 | 3.65 |
| [DeepSeek Harness](#deepseek-harness) · preview runtime | 2 | 2 | 5 | 3 | 2 | 55/100 | 2.75 |

**Highest under these weights:** LangGraph (78), driven by explicit workflow
state and extensibility—not turnkey operation. Claude and Codex tie at 75; the
evidence does not justify declaring one universally better. Use the decision
tree to shortlist by task.

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
       - Either fits → trial both with the same tasks, models where
         comparable, budgets, and acceptance checks.
   - **No → 2. Is Azure-managed hosting a requirement?**
     - Yes → [Foundry Agent Service](#microsoft-foundry-agent-service);
       confirm region, data handling, and required feature maturity. Add
       [Agent Framework](microsoft.md) if you need code-first orchestration;
       it is a separate choice.
     - No → **3. Do you need durable, branching workflows with human
       checkpoints?**
       - Yes → [LangGraph](#langgraph), with a durable checkpointer and
         application-owned authorization.
       - No → **4. Is this isolated research requiring replaceable runtime
         internals?**
         - Yes, and preview risk is acceptable →
           [DeepSeek Harness](#deepseek-harness). Not for production trust
           boundaries.
         - No → Start with a
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

The chart plots extensibility (horizontal, 1–5) against recovery support
(vertical, 1–5), with quadrant boundaries at 3.5 on each axis:

| Quadrant | Extensibility | Recovery | Harnesses |
| --- | --- | --- | --- |
| Durable specialists (low E, high R) | < 3.5 | ≥ 3.5 | Foundry (3, 4) |
| Composable workflows (high E, high R) | ≥ 3.5 | ≥ 3.5 | LangGraph (5, 5) |
| Focused runtimes (low E, low R) | < 3.5 | < 3.5 | Copilot (3, 2) |
| Flexible foundations (high E, low R) | ≥ 3.5 | < 3.5 | Claude and Codex share (4, 3); DeepSeek (5, 2) |

Positions are exactly (E, R) from the scorecard, on 1–5 scales; boundaries are
at 3.5. Claude and Codex share a point rather than being artificially
separated. Quadrant names describe relative strengths, not quality tiers.
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
