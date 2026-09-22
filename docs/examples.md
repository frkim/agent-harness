# Examples

[Back to the guide](../README.md) · Next: [Operations](operations.md)

## Worked example: a repository maintenance agent

Consider an order-service team receiving this request:

> Reject negative quantities in the total calculator, preserve zero-quantity
> behavior, and prepare a patch for review. Do not publish or deploy anything.

1. **Receive:** record the two behavior requirements and the no-publication
   constraint in the [task contract](execution.md#task-and-execution-contracts).
2. **Orient:** inspect the existing calculator, tests, repository instructions,
   and working-tree changes. Clarify the exception type if it is not established
   by the API contract.
3. **Plan:** add a negative-input regression test, add the smallest guard, and
   run the relevant checks. Leave unrelated pricing behavior untouched.
4. **Act:** edit only the two authorized files in an isolated checkout. A
   hypothetical Python implementation might contain this guard:

   ```python
   def total_price(unit_price, quantity):
       if quantity < 0:
           raise ValueError("quantity must be non-negative")
       return unit_price * quantity
   ```

   This assumes the application already validates numeric types and represents
   currency appropriately; it is not a complete order or money-handling API.
5. **Verify:** confirm the negative-input test fails on the baseline and passes
   on the candidate. Check zero and positive quantities, inspect the diff, and
   run applicable broader checks. If dependencies are unavailable, report that
   verification is blocked rather than reporting a verified fix.
6. **Report:** return the patch, checked revision, actual check results, and
   limitations. Publication is not authorized by this task.

If the product later adds pull-request creation, make it a separately
authorized action:

```mermaid
sequenceDiagram
    actor User
    participant Harness
    participant Model
    participant Sandbox
    participant Reviewer
    participant Git as Repository service
    User->>Harness: Request a patch with acceptance criteria
    Harness->>Model: Scoped context and permitted tools
    Model-->>Harness: Proposed edits
    Harness->>Harness: Validate permissions and limits
    Harness->>Sandbox: Apply edits and run required checks
    Sandbox-->>Harness: Candidate revision, diff, and evidence
    Harness-->>User: Patch and verification report
    opt Separately requested pull-request publication
        Harness->>Reviewer: Exact candidate and publication action
        alt Approved
            Reviewer-->>Harness: Scoped, expiring approval
            Harness->>Harness: Revalidate authorization and revision
            Harness->>Git: Publish approved candidate if still valid
            Git-->>Harness: Confirmed result
            Harness-->>User: Publication result
        else Rejected or expired
            Harness-->>User: Patch retained, nothing published
        end
    end
```

## Project and product examples

### Products you could build

| Product | Bounded agent task | Tools and context | Verification and human boundary |
| --- | --- | --- | --- |
| Repository maintenance assistant | Prepare a narrowly scoped bug-fix patch | Source search, file edits, sandboxed checks | Regression evidence and diff review; separate approval to publish |
| Customer-support copilot | Draft a response using approved help content | Tenant-scoped ticket lookup and knowledge retrieval | Check cited policy and redact personal data; human approves sending or refunds |
| Analytics assistant | Answer a business question from approved datasets | Catalog lookup and read-only, resource-limited queries | Validate aggregates and freshness; restrict sensitive rows and exports |
| Incident triage assistant | Summarize symptoms and propose a runbook step | Redacted logs, metrics, and read-only service status | Link claims to telemetry; operator authorizes production mutations |

Read-only does not mean risk-free: retrieval can expose private data and queries
can exhaust resources. Apply authorization, data minimization, and resource
limits even when no writes are allowed.

### Existing projects to study

These examples illustrate different parts of a harness, not interchangeable
solutions or dependencies of this repository. Consult their official
documentation for current behavior and configuration.

| Project or product | Relevant capabilities | Design lesson and boundary |
| --- | --- | --- |
| [LangGraph](https://docs.langchain.com/oss/python/langgraph/overview) | Stateful orchestration, durable execution, and human-in-the-loop workflows | Resumable approval checkpoints need configured persistence and thread identity; orchestration is not an authorization system. |
| [DeepSeek Harness](deepseek.md) | Plugin-based agent loop, tools, runtime profiles, and session persistence | An unaudited developer preview; plugin composition and approval prompts are not substitutes for independent isolation. |
| [OpenAI Agents SDK](#openai-agents-sdk) | Agent/tool orchestration, handoffs, guardrails, and tracing | Agent-level input/output guardrails do not inspect every intermediate tool call; enforce authorization at tool execution boundaries. |
| [OpenHands Software Agent SDK](#openhands-software-agent-sdk) | Software-development tools, Docker workspaces, and configurable action confirmation | Runtime isolation and approval policies are separate controls; confirmation behavior depends on configuration. |
| [GitHub Copilot cloud agent](https://docs.github.com/en/copilot/concepts/agents/cloud-agent/about-cloud-agent) | Repository-oriented work in an ephemeral environment, test/lint execution, and reviewable changes | A reviewable pull request is an artifact, not proof of correctness; humans still review changes and verification evidence. |

For details behind these boundaries, see LangGraph's
[interrupts](https://docs.langchain.com/oss/python/langgraph/interrupts),
the Agents SDK's [guardrails](https://openai.github.io/openai-agents-python/guardrails/),
OpenHands' [security guide](https://docs.openhands.dev/sdk/guides/security),
and GitHub's [responsible-use guidance](https://docs.github.com/en/copilot/responsible-use/agents).

Choose based on the missing layer: orchestration for resumable workflows,
an SDK for tool-driven application logic, or a coding-oriented environment for
repository tasks. In every case, decide which controls the product supplies
and which remain your application's responsibility.

## Dedicated implementation coverage

Source review: **2026-09-22**, using the official sources linked in each
section. These are implementation maps, not installed integrations or
runtime-tested examples. Pin a release and verify the chosen configuration;
default-branch documentation can describe features absent from that release.

The first five priorities are Deep Agents, Google ADK, OpenAI Agents SDK,
OpenHands SDK, and Strands Agents. They appear in their respective groups below,
not in popularity order. Pydantic AI and CrewAI broaden the application-framework
comparison; Gemini CLI, OpenCode, and Pi broaden the coding-runtime comparison.

| Group | Dedicated sections |
| --- | --- |
| Application-agent frameworks | [Deep Agents](#langchain-deep-agents), [Google ADK](#google-adk), [OpenAI Agents SDK](#openai-agents-sdk), [Strands Agents](#strands-agents), [Pydantic AI / Harness](#pydantic-ai-and-pydantic-ai-harness), [CrewAI](#crewai) |
| Coding runtimes and toolkits | [OpenHands SDK](#openhands-software-agent-sdk), [Gemini CLI](#gemini-cli), [OpenCode](#opencode), [Pi](#pi) |
| Managed hosting services | [Foundry Agent Service](microsoft.md); a separate deployment choice, not a capability automatically supplied by the frameworks above |

For every mapping, the application owns caller identity, scoped credentials,
budgets, independent acceptance checks, and authorization for publication.
Multi-agent delegation must not silently broaden a worker's authority. Retained
messages, restored workspace files, and durable workflow execution are distinct
recovery mechanisms; none by itself guarantees exactly-once external effects.

### Application-agent frameworks

#### LangChain Deep Agents

**What it provides:** a packaged harness with planning, filesystem access,
context management, and subagent delegation. It builds on LangGraph's execution
and checkpoint machinery; [LangGraph coverage](langgraph.md) alone does not
explain these supplied agent-loop capabilities. LangSmith hosting is separate.

**Mapping the harness:**

1. Configure `create_deep_agent` with a tool-calling model, scoped tools, and
   deliberately chosen filesystem or sandbox backends.
2. Use `interrupt_on` for protected tools and review subagent approval
   configuration, which can inherit or override the parent's settings.
3. Attach a durable checkpointer and stable, caller-bound `thread_id` for
   restartable work. Record graph/tool activity, optionally through LangSmith,
   and run independent checks before returning a candidate artifact.

**Boundaries:** approval configuration is not OS isolation. A filesystem
abstraction does not itself make arbitrary shell execution safe. Thread scratch
state, cross-thread memory, and execution checkpoints serve different purposes.
An in-memory checkpointer cannot establish process-restart durability; retries
and checkpoints do not make external effects exactly-once.

**Verification exercise:** pause a mock write at approval, restart using the
same durable store and thread, reject it, and inspect the file. Repeat through a
subagent. Inject a timeout after a mock external effect and verify reconciliation
or deduplication before retry.

**Official sources:** [Harness overview](https://github.com/langchain-ai/deepagents),
[human-in-the-loop](https://github.com/langchain-ai/docs/blob/main/src/oss/deepagents/human-in-the-loop.mdx),
[memory scopes](https://github.com/langchain-ai/docs/blob/main/src/oss/deepagents/memory.mdx),
[production and tracing](https://github.com/langchain-ai/docs/blob/main/src/oss/deepagents/going-to-production.mdx).

#### Google ADK

**What it provides:** a general-purpose Agent Development Kit with tool-using
agents, multi-agent/workflow composition, artifacts, and execution callbacks.
It is not merely a coding assistant, and installing ADK does not provision
managed hosting.

**Mapping the harness:**

1. Define a root agent and scoped tools; choose agent delegation or an explicit
   workflow for the task. Configure session and artifact services independently.
2. Use tool confirmation and callbacks where appropriate, while keeping caller
   authorization in the tool/service boundary.
3. For workflow recovery, explicitly enable app resumability, retain session,
   user, and invocation identifiers, and configure persistent storage. Export
   OpenTelemetry traces for workflow, model, and tool activity.

**Boundaries:** conversation sessions and searchable memory are not the same
as resumable execution. ADK documents at-least-once tool execution: an interrupted
tool may run again. Custom agents need explicit resume support. The reviewed
confirmation guide marks the feature experimental and lists
`DatabaseSessionService` and `VertexAiSessionService` as unsupported; do not
assume approval and durable storage compose in your chosen release.
`adk web` is a development interface, not a production deployment.

**Verification exercise:** interrupt the last tool in a three-step workflow
after a mock external write. Resume the original invocation and verify completed
steps are retained while an idempotency key prevents a duplicate write. Test
confirmation separately against the exact session backend you intend to deploy.

**Official sources:** [Agents and workflows](https://github.com/google/adk-docs/blob/main/docs/agents/index.md),
[tool confirmation](https://github.com/google/adk-docs/blob/main/docs/tools-custom/confirmation.md),
[resumability](https://github.com/google/adk-docs/blob/main/docs/runtime/resume.md),
[tracing](https://github.com/google/adk-docs/blob/main/docs/observability/traces.md).

#### OpenAI Agents SDK

**What it provides:** application-agent orchestration through agents, tools,
handoffs, guardrails, sessions, and tracing. This is separate from
[Codex](codex.md), which supplies a coding runtime.

**Mapping the harness:**

1. Define an `Agent`, bounded tools, and handoff destinations; execute through
   `Runner`. A handoff changes routing, not the caller's authority.
2. Mark protected supported tools with `needs_approval`, inspect interruptions,
   and retain the resulting `RunState` in trusted application storage.
   Authenticate approval or rejection before resuming.
3. Select a conversation-history strategy and configure tracing/redaction.
   Do not combine SDK sessions with run-level server-managed continuation
   options such as `conversation_id` or `previous_response_id`.

**Boundaries:** input/output guardrails are not authorization for every
intermediate tool call. Sessions retain conversation items; `RunState` preserves
paused execution and pending approvals, not arbitrary crash-safe tool execution.
Serialized state does not authenticate itself or its reviewer. Built-in tracing
can include sensitive model/tool data; review capture and export settings.

**Verification exercise:** persist an approval-paused mock action, restart,
reject it, and confirm no effect and correct session continuity. Submit the
same decision twice and confirm the application prevents duplicate resumption.
Test denied intermediate tools independently of final-output guardrails.

**Official sources:** [Quickstart](https://github.com/openai/openai-agents-python/blob/main/docs/quickstart.md),
[approval and RunState](https://github.com/openai/openai-agents-python/blob/main/docs/human_in_the_loop.md),
[sessions](https://github.com/openai/openai-agents-python/blob/main/docs/sessions/index.md),
[tracing](https://github.com/openai/openai-agents-python/blob/main/docs/tracing.md).

#### Strands Agents

**What it provides:** a model-driven agent SDK with tools and multi-agent
patterns. The model-driven loop is an alternative to defining every transition
as a graph, not an alternative to enforcing tool policy. Hosting is separate.

**Mapping the harness:**

1. Configure an `Agent` with the chosen provider, model, and scoped tools;
   deliberately define authority when composing multiple agents.
2. Use supported tool interrupts or `BeforeToolCallEvent` hooks for approval.
   Match authenticated decisions to interrupt IDs and cancel rejected tools.
3. Configure a persistent session manager and stable identifiers. Retain
   snapshots when explicit checkpoint selection is required, and export
   OpenTelemetry traces for model, agent-loop, and tool activity.

**Boundaries:** an interrupt is not a global barrier: other concurrent tools
can execute. Direct tool calls do not support tool interrupts. Session state and
persisted approval interruptions support continuation, but do not guarantee
arbitrary mid-tool recovery or exactly-once effects. Snapshot storage and
external-effect reconciliation remain application responsibilities.

**Verification exercise:** issue an approval-gated mock write alongside a
harmless read. Confirm the write waits even if the read completes. Reconstruct
the agent using the persisted session, reject the original interrupt, and
verify no write. Test retries separately from recovery.

**Official sources:** [Quickstart](https://github.com/strands-agents/docs/blob/main/site/src/content/docs/user-guide/quickstart/python.mdx),
[interrupts](https://github.com/strands-agents/docs/blob/main/site/src/content/docs/user-guide/concepts/interrupts.mdx),
[session management](https://github.com/strands-agents/docs/blob/main/site/src/content/docs/user-guide/concepts/agents/session-management.mdx),
[tracing](https://github.com/strands-agents/docs/blob/main/site/src/content/docs/user-guide/observability-evaluation/traces.mdx).

#### Pydantic AI and Pydantic AI Harness

**What they provide:** Pydantic AI is a typed application-agent foundation:
tools, structured outputs, dependencies, and deferred tool approval/execution.
Pydantic AI Harness is a separate capability library built on it. Its `Coder`
stack adds filesystem tools, a persistent shell, repository context, and context
management; individual capabilities can also be composed.

**Mapping the harness:**

1. Define typed tools and outputs on an `Agent`; inject caller-scoped
   dependencies rather than letting model arguments select credentials.
2. Use deferred tools for protected actions. Persist the required history and
   resolve requests with authenticated decisions through `DeferredToolResults`.
3. Choose storage for the requirement: message history for conversations,
   Harness `StepPersistence` for continuable snapshots and tool-effect records,
   or an explicit durable-execution integration such as Temporal, DBOS, or
   Prefect. Enable model/tool tracing with the intended OpenTelemetry backend.

**Boundaries:** installing Pydantic AI does not activate durable execution.
Harness step persistence does not capture full graph state, workspace snapshots,
or capability state. An unresolved tool may already have acted; its ledger is
not permission to replay it. Harness has a separate 0.x stability policy.
`Coder` runs shell commands on the host without a command allowlist; standalone
shell controls are best-effort guardrails, not isolation.

**Verification exercise:** test denial and malformed outputs with the framework's
test models. Then terminate after a mock external write but before its result is
recorded. Inspect unresolved effects and reconcile them before continuation.
Test worker recovery separately under the selected durable engine; do not infer
it from restored messages or assume a continuation restores files.

**Official sources:** [Storage distinctions](https://github.com/pydantic/pydantic-ai/blob/main/docs/storage.md),
[durable execution](https://github.com/pydantic/pydantic-ai/blob/main/docs/durable_execution/overview.md),
[Harness Coder](https://github.com/pydantic/pydantic-ai-harness/blob/main/docs/coder.md),
[step persistence](https://github.com/pydantic/pydantic-ai-harness/blob/main/docs/step-persistence.md).

#### CrewAI

**What it provides:** role-based agents, tasks, and crews, alongside Flows for
explicit state, branching, and event-driven orchestration. A crew delegates
work; a flow controls when that work runs. Neither is a managed hosting service.

**Mapping the harness:**

1. Assign narrowly scoped tools to each role and define task acceptance criteria.
   Use a flow to separate preparation, review, and authorized publication.
2. Put consequential tool actions behind pre-tool controls. Treat delegated
   workers as separate permission scopes, not extensions of an unrestricted lead.
3. Configure checkpoint storage and tracing deliberately. The reviewed
   versioned documentation describes Crew, Flow, and Agent checkpoints, including
   restoration that skips completed tasks; memory alone is not that mechanism.

**Boundaries:** automatic checkpoint writes are best-effort, so monitor storage
failures. Tool-hook exceptions can fail open unless the documented abort
mechanism is used. Docker-backed safe code execution does not isolate every
custom tool. Tracing through CrewAI AMP requires its own configuration and
data-handling review; local execution does not imply traces stay local.

**Verification exercise:** interrupt a two-task crew after the first task's
checkpoint and verify restore skips that task. Repeat with unavailable storage
and a mock external write. Test a rejecting hook and a hook that throws an
ordinary exception; confirm the protected effect cannot escape the policy.

**Official sources:** [Flows](https://github.com/crewAIInc/crewAI/blob/main/docs/v1.15.22/en/concepts/flows.mdx),
[checkpointing](https://github.com/crewAIInc/crewAI/blob/main/docs/v1.15.22/en/concepts/checkpointing.mdx),
[tool hooks](https://github.com/crewAIInc/crewAI/blob/main/docs/v1.15.22/en/learn/tool-hooks.mdx),
[tracing](https://github.com/crewAIInc/crewAI/blob/main/docs/v1.15.22/en/observability/tracing.mdx).
These sources describe that documentation version, not every installed release.

### Coding runtimes and toolkits

#### OpenHands Software Agent SDK

**What it provides:** programmable software-development agents, tools,
conversations, execution workspaces, and an Agent Server. This section covers
the SDK, not the hosted OpenHands product or its UI lifecycle.

**Mapping the harness:**

1. Configure the model, register terminal/file tools, and create an agent and
   conversation. Select a local, container-backed, or remote workspace
   deliberately; a local workspace is not automatically isolated.
2. Set a confirmation policy and handle pending actions explicitly, including
   rejection. Risk analysis and confirmation policy are separate mechanisms.
3. Configure persistence, retain the conversation ID/store and workspace
   identity, and collect event callbacks, usage metrics, or OTLP traces.
   Review the resulting diff and independent checks before publication.

**Boundaries:** persistence saves event history and execution/base state, not
just messages. It does not establish restoration of arbitrary live processes or
rollback of remote effects. Reconnecting a conversation does not recreate a
missing workspace. Configure isolation and scoped credentials independently
from confirmation, and check storage locking requirements before using a
shared filesystem.

**Verification exercise:** persist a conversation waiting for confirmation,
recreate it with the same workspace/ID/store, and reject the pending edit.
Check event continuity and unchanged files. Separately interrupt a running shell
command and inspect actual process/workspace state before resuming.

**Official sources:** [SDK setup](https://github.com/OpenHands/docs/blob/main/sdk/getting-started.mdx),
[confirmation example](https://github.com/OpenHands/software-agent-sdk/blob/main/examples/01_standalone_sdk/04_confirmation_mode_example.py),
[persistence](https://github.com/OpenHands/docs/blob/main/sdk/guides/convo-persistence.mdx),
[observability](https://github.com/OpenHands/docs/blob/main/sdk/guides/observability.mdx).

#### Gemini CLI

**What it provides:** a coding runtime with shell/file tools, hooks, interactive
sessions, and headless operation. It is not Google ADK: the CLI supplies a coding
loop rather than an application-agent workflow framework.

**Mapping the harness:**

1. Run in an isolated checkout and configure tool policy and sandboxing
   separately. Check the selected platform's filesystem and network profile.
2. For automation, use headless prompts and JSON or streaming JSON events.
   Handle tool failures and exit status rather than treating printed text as
   success. Approval-required `ask_user` actions are denied in headless mode.
3. Retain session data for conversation resume. Enable optional checkpointing
   when local file restoration is needed; inspect event records and the final diff.

**Boundaries:** checkpointing is disabled by default. Its shadow-Git snapshots
and `/restore` differ from `--resume`, and do not undo remote side effects.
Sandbox profiles differ: an enabled sandbox is not necessarily a network-denied
or secret-free environment. Verify policy loading against your release; the
reviewed policy reference warns that workspace-level policies are nonfunctional.

**Verification exercise:** deny a tool in a headless run and verify no effect
occurred. Interrupt an approved edit, then test session resume and checkpoint
restore separately. Compare actual files and a mock external service, not just
the recovered transcript.

**Official sources:** [Headless mode](https://github.com/google-gemini/gemini-cli/blob/main/docs/cli/headless.md),
[policy engine](https://github.com/google-gemini/gemini-cli/blob/main/docs/reference/policy-engine.md),
[checkpointing](https://github.com/google-gemini/gemini-cli/blob/main/docs/cli/checkpointing.md),
[sandboxing](https://github.com/google-gemini/gemini-cli/blob/main/docs/cli/sandbox.md).

#### OpenCode

**What it provides:** a multi-provider coding runtime with a client/server
architecture. Its typed JavaScript/TypeScript SDK can start a server or connect
to one, manage sessions, respond to permissions, and consume execution events.

**Mapping the harness:**

1. Select the provider/model and inspect tool and MCP configuration. Configure
   explicit `allow`, `ask`, and `deny` rules rather than relying on defaults.
2. Bind the server appropriately and configure authentication before exposing
   it. Use the SDK's session and event interfaces to integrate with your UI.
3. Persist session identity and inspect tool results, diffs, and checks.
   Keep publication outside the edit permission scope.

**Boundaries:** the official security policy calls permissions a UX feature,
not security isolation; use a container or VM for that boundary. Server mode
is unauthenticated unless configured otherwise. Session resume/fork and
Git-dependent undo/redo are not guarantees of durable workflow replay or
rollback of external effects. Provider support does not make models equivalent.

**Verification exercise:** verify configured authentication rejects an
unauthenticated request, and independently check a denied tool has no effect.
Interrupt a mock write, reconnect through the SDK, and reconcile session events
with actual state before retrying. Test Git-dependent undo separately.

**Official sources:** [SDK](https://github.com/anomalyco/opencode/blob/dev/packages/web/src/content/docs/sdk.mdx),
[permissions](https://github.com/anomalyco/opencode/blob/dev/packages/web/src/content/docs/permissions.mdx),
[session and undo controls](https://github.com/anomalyco/opencode/blob/dev/packages/web/src/content/docs/tui.mdx),
[security boundary](https://github.com/anomalyco/opencode/blob/dev/SECURITY.md).

#### Pi

**What it provides:** a composable toolkit spanning model access, an agent loop,
and a coding CLI, with an embeddable SDK and process-oriented RPC/JSON modes.
It is an architectural contrast to more opinionated runtimes, not a complete
permission or hosting layer.

**Mapping the harness:**

1. Choose CLI automation, RPC, or SDK embedding; select the model and only the
   tools required for the task. Review extensions before loading them.
2. Supply an independently isolated environment for untrusted or unattended
   tasks. Pi runs with the launching user's privileges and has no built-in sandbox.
3. Consume message and tool events, retain session JSONL, and verify the patch
   independently. Manage workspace checkpoints separately, for example with Git.

**Boundaries:** project trust controls loading local resources and extensions,
not subsequent tool authority. Extensions share process privileges. Session
trees support continuation, forks, and navigation, but branching conversation
history does not roll back files or external writes.

**Verification exercise:** terminate during a controlled edit in a disposable
container, resume the session, and compare its history to filesystem/process
state. Fork the history and confirm files have not silently reverted. Test OS
containment separately from project trust.

**Official sources:** [Project and layers](https://github.com/earendil-works/pi),
[SDK](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/sdk.md),
[security](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/security.md),
[sessions](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/sessions.md).

For the corresponding comparisons, see the
[grouped evaluation](harness-evaluation.md#at-a-glance) and the other
[implementation chapters](../README.md#implementation-chapters).
