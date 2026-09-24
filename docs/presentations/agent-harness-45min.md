---
marp: true
theme: default
paginate: true
title: "Designing an Agent Harness"
description: "A 45-minute walkthrough of the agent harness design guide"
author: "Agent Harness contributors"
style: |
  section {
    font-size: 26px;
    color: #101a35;
    background:
      linear-gradient(90deg, #4338ca 0%, #7c3aed 50%, #06b6d4 100%) top left / 100% 12px no-repeat,
      linear-gradient(135deg, #ffffff 0%, #f3f6ff 55%, #ecfeff 100%);
  }
  section h1,
  section h2 {
    color: #312e81;
  }
  section h2 {
    padding-bottom: 10px;
    background-image: linear-gradient(90deg, #7c3aed, #06b6d4);
    background-repeat: no-repeat;
    background-size: 160px 6px;
    background-position: 0 100%;
  }
  section strong {
    color: #be123c;
  }
  section blockquote {
    border-left: 8px solid #7c3aed;
    background: #f5f3ff;
    border-radius: 0 12px 12px 0;
    padding: 12px 18px;
  }
  section pre {
    border-left: 8px solid #06b6d4;
    border-radius: 10px;
  }
  section.lead {
    text-align: center;
    color: #ffffff;
    background:
      linear-gradient(90deg, #22d3ee 0%, #f472b6 50%, #fbbf24 100%) top left / 100% 12px no-repeat,
      linear-gradient(135deg, #312e81 0%, #6d28d9 50%, #0e7490 100%);
  }
  section.lead h1 {
    font-size: 54px;
    color: #ffffff;
  }
  section.lead h2,
  section.lead h3 {
    color: #ffffff;
    background-image: none;
  }
  section.lead a {
    color: #67e8f9;
  }
  section.lead strong {
    color: #fde68a;
  }
  section.lead::after,
  section.lead footer {
    color: #ddd6fe;
  }
  section.dense {
    font-size: 22px;
  }
  table {
    font-size: 22px;
    border-collapse: collapse;
  }
  th {
    background: linear-gradient(90deg, #4338ca, #7c3aed);
    color: #ffffff;
    border-color: #c7d2fe;
  }
  td {
    border-color: #d8dff2;
  }
  tr:nth-child(even) td {
    background: #eef2ff;
  }
  section.dense table {
    font-size: 20px;
  }
  code {
    font-size: 0.85em;
  }
  section > p > code,
  section li > code {
    background: #eef2ff;
    color: #3730a3;
    border-radius: 6px;
  }
  footer {
    font-size: 16px;
    color: #475569;
  }
  .row {
    display: flex;
    gap: 12px;
    align-items: stretch;
    justify-content: center;
    margin: 8px 0;
  }
  .col {
    display: flex;
    flex: 1;
    flex-direction: column;
    gap: 12px;
  }
  .node {
    flex: 1;
    border-radius: 14px;
    padding: 12px 14px;
    color: #ffffff;
    font-size: 19px;
    font-weight: 700;
    line-height: 1.25;
    text-align: center;
    box-shadow: 0 8px 18px rgba(17, 24, 60, 0.18);
  }
  .node span {
    display: block;
    margin-top: 4px;
    font-size: 15px;
    font-weight: 400;
    opacity: 0.93;
  }
  .indigo { background: linear-gradient(135deg, #4338ca, #6366f1); }
  .violet { background: linear-gradient(135deg, #7c3aed, #a855f7); }
  .cyan   { background: linear-gradient(135deg, #0e7490, #22d3ee); }
  .green  { background: linear-gradient(135deg, #047857, #34d399); }
  .amber  { background: linear-gradient(135deg, #b45309, #fbbf24); }
  .rose   { background: linear-gradient(135deg, #be123c, #fb7185); }
  .slate  { background: linear-gradient(135deg, #334155, #64748b); }
  .tight {
    padding: 7px 14px;
  }
  .ghost {
    background: #ffffff;
    color: #312e81;
    border: 3px dashed #a5b4fc;
    box-shadow: none;
  }
  .arrow {
    flex: 0 0 auto;
    align-self: center;
    color: #6366f1;
    font-size: 28px;
    font-weight: 700;
  }
  .caption {
    margin: 0;
    font-size: 17px;
    color: #475569;
    text-align: center;
  }
  .bar {
    border-radius: 14px;
    padding: 3px 12px;
    color: #ffffff;
    font-size: 17px;
    font-weight: 700;
    box-sizing: border-box;
  }
  .w25 { width: 100%; }
  .w20 { width: 80%; }
  .w15 { width: 60%; }
footer: "Agent Harness design guide · github.com/frkim/agent-harness"
---

<!-- _class: lead -->
<!-- _paginate: false -->

# Designing an Agent Harness

The control plane around an AI model

45 minutes · design guide walkthrough

<!--
Timing plan for the whole deck (45 minutes):
- Opening and framing: 3 min
- Part 1 Foundations: 7 min
- Part 2 Execution: 9 min
- Part 3 Safety and verification: 8 min
- Part 4 Operations and rollout: 5 min
- Part 5 Worked example: 4 min
- Part 6 Choosing a harness and first release: 5 min
- Questions and close: 4 min
-->

---

## What you get from this talk

- A shared definition of a **harness** and the concerns it owns.
- A bounded execution loop you can implement in one service.
- Controls that hold when the model behaves unexpectedly.
- Evidence and reporting rules that make results auditable.
- A rubric for comparing existing harnesses at the layer you need.
- A checklist to scope a first release.

This is a design guide, not a framework: no SDK, no installation step.
Examples are illustrative contracts and pseudocode.

<!--
Set expectations: attendees leave with vocabulary, a rubric, and a checklist,
not with a library to import.
-->

---

## Agenda

| Part | Topic | Time |
| --- | --- | --- |
| 1 | Foundations: why a harness, core model, lifecycle | 7 min |
| 2 | Execution: task contracts, controller, tools, context, memory | 9 min |
| 3 | Safety and verification | 8 min |
| 4 | Operations: reports, observability, rollout | 5 min |
| 5 | Worked example: retry-safe webhook ingestion | 4 min |
| 6 | Choosing a harness: layers, rubric, adoption gates, checklist | 5 min |
| — | Questions | 4 min |

<!--
Announce that every part maps to one chapter in docs/: foundations, execution,
safety-and-verification, operations, examples, and harness-evaluation.
-->

---

<!-- _class: lead -->

# Part 1 — Foundations

Why a harness, the core model, and the lifecycle

---

## A model answers; production work needs more

Production work usually requires:

- understanding a request and the surrounding repository or service;
- choosing and invoking tools such as search, tests, or issue trackers;
- preserving user intent, permissions, and repository conventions;
- checking the result before reporting it; and
- leaving an auditable record of actions and decisions.

The harness owns these responsibilities. It makes the safe and correct path
easy while keeping the agent's authority no broader than necessary.

<!--
Emphasize the last sentence: authority is the variable we are controlling
throughout the talk.
-->

---

## Concrete difference

A model can **suggest** a bug fix.

A harness can:

1. restrict the work to one checkout,
2. collect the relevant code,
3. apply a proposed patch,
4. run tests in a sandbox, and
5. present the diff for review.

The useful outcome is a **verified artifact**, not merely a plausible response.

<!--
This is the anchor example for the rest of the talk; the worked example in
Part 5 returns to it.
-->

---

## Choose the simplest suitable approach

| Approach | Best fit | Boundary |
| --- | --- | --- |
| Direct model call | Summarizing text, drafting an answer | The app still owns input handling and output validation |
| Deterministic workflow | Known steps: extract, validate, route | Prefer ordinary code when the next action is predictable |
| Agent with a harness | Adaptive search, tool selection, iteration | Flexibility requires budgets, permissions, verification |
| Multiple agents | Work that benefits from independent specialists | Coordination, shared state, verification get harder |

<!--
Ask the room which row their current project actually needs.
Many "agent" projects are the second row.
-->

---

## What RAG and MCP do not give you

- **Retrieval-augmented generation** supplies relevant information.
  It does not by itself authorize actions.
- **A tool protocol** such as the Model Context Protocol standardizes
  integration. It does not replace application policy, identity checks,
  or runtime isolation.

Integration is not authorization.

<!--
Common misconception slide; keep it short but do not skip it.
-->

---

## Core model: eight separable concerns

| Concern | Responsibility |
| --- | --- |
| **Task** | Requested outcome, acceptance criteria, constraints |
| **Context** | Only the repository, user, and execution info needed |
| **Policy** | Non-negotiable safety, privacy, approval, and scope rules |
| **Agent** | Reasons about the task, proposes the next action |
| **Tools** | Explicit, typed capabilities to inspect, change, validate, publish |
| **Runtime** | Isolation, timeouts, credentials, filesystem and network limits |
| **Verifier** | Tests, linters, reviewers, explicit acceptance checks |
| **Reporter** | Progress, evidence, limitations, final outcome |

<!--
Keep the layers distinct: policy and tool permissions belong to the harness,
not to instructions supplied to the model.
-->

---

## Reference architecture

<div class="row">
<div class="node indigo">Task intake<span>objective and acceptance criteria</span></div>
<div class="arrow">→</div>
<div class="node violet">Controller<span>task state, budget, scheduling</span></div>
<div class="arrow">↔</div>
<div class="node cyan">Model<span>proposes the next action</span></div>
</div>

<div class="row">
<div class="node amber">Policy and budget gate<span>identity · arguments · authorization · limits</span></div>
</div>

<div class="row">
<div class="node rose">Denied<span>stop and explain</span></div>
<div class="node slate">Approval required<span>human decides; scope rechecked</span></div>
<div class="node green">Allowed<span>tool dispatcher</span></div>
</div>

<div class="row">
<div class="node ghost">Isolated runtime<span>read-only · scoped writes · verification runner</span></div>
<div class="arrow">→</div>
<div class="node indigo">Structured results<span>audit events and final report</span></div>
</div>

The **controller** owns task state; the **dispatcher** checks every invocation;
the **runtime** enforces filesystem, process, and network boundaries even if the
model or a tool misbehaves.

<!--
The rendered Mermaid version of this diagram is in docs/foundations.md.
-->

---

## Start as modules, not microservices

- For a first version, these can be modules in one service.
- **Separate the trust boundaries before separating deployments.**
- Store durable task state so a restart does not silently lose approvals or
  repeat consequential actions.
- External systems still need their own scoped credentials: a sandbox alone
  cannot prevent misuse of a permitted API.

<!--
Counter the instinct to start with a distributed system.
-->

---

## Agent lifecycle: six bounded steps

1. **Receive** — parse the task, identify the target, record the outcome.
2. **Orient** — inspect instructions, status, code, and tests before changing.
3. **Plan** — small verifiable steps; surface ambiguity and approvals early.
4. **Act** — call the least-privileged tool; inspect before mutating.
5. **Verify** — run the narrowest relevant checks, review the diff.
6. **Report** — what changed, how it was verified, what risk remains.

Stop when the task is complete, when a policy boundary is reached, or when a
user decision is required.

<!--
Contrast with an unstructured chat loop: each step has an exit condition.
-->

---

## States, not conversation turns

<div class="row">
<div class="node indigo">Received</div>
<div class="arrow">→</div>
<div class="node indigo">Orienting</div>
<div class="arrow">→</div>
<div class="node violet">Planning</div>
<div class="arrow">→</div>
<div class="node cyan">Acting</div>
<div class="arrow">→</div>
<div class="node green">Verifying</div>
<div class="arrow">→</div>
<div class="node green">Completed</div>
</div>

<p class="caption">Verifying → Acting is a bounded repair loop, never an open-ended retry.</p>

<div class="row">
<div class="node amber">Waiting<span>clarification or approval; revalidate before resuming</span></div>
<div class="node rose">Stopped<span>rejected · expired · denied · cancelled · budget exhausted</span></div>
</div>

- Cancellation and deadlines apply in **every** nonterminal state.
- A stopped run may keep useful artifacts, but is never reported as completed.

---

## Recovery and retries

- Retry transient **read** failures with bounded backoff; never retry a policy
  denial or an invalid argument unchanged.
- Before retrying a **write** after a timeout, query the destination.
  A missing response does not mean the action failed.
- Use idempotency keys where supported; otherwise reconcile or ask a human.
- Checkpoint plan, artifact revision, completed tool calls, pending approvals.
  Revalidate permissions and resource versions on resume.
- Do not assume external effects can be rolled back: define a compensating
  action or a manual recovery procedure.

<!--
About ten minutes elapsed at the end of Part 1.
-->

---

<!-- _class: lead -->

# Part 2 — Execution

Task contracts, the bounded controller, tools, and context

---

## Make the outcome machine-readable

```json
{
  "task_id": "task-042",
  "objective": "Reject a negative quantity when calculating an order total",
  "acceptance_criteria": [
    "Negative quantities raise ValueError",
    "Zero and positive quantities keep their existing behavior"
  ],
  "allowed_write_paths": ["src/orders.py", "tests/test_orders.py"],
  "allowed_tools": ["read_file", "apply_patch", "run_check"],
  "verification_targets": ["orders-unit-tests"],
  "limits": { "max_model_turns": 12, "max_tool_calls": 30,
              "wall_time_seconds": 600, "max_output_bytes_per_call": 32768 },
  "on_limit": "stop_and_report"
}
```

<!--
Say clearly that these paths belong to a hypothetical order service,
not to this repository.
-->

---

## Rules behind the contract

- Authenticate the caller and **intersect** requested permissions with
  server-side policy.
- Never let a task grant itself access.
- Resolve check identifiers against trusted configuration, outside
  agent-writable files.
- Add model token/cost limits and per-tool deadlines for your deployment.

The numbers on the previous slide are starting points, not production defaults.

---

<!-- _class: dense -->

## Bounded controller (responsibility sketch)

```text
task  = authenticate_validate_and_scope(request, server_policy)
state = create_or_resume_task(task)

while not state.terminal:
    enforce_deadline_cancellation_and_remaining_budget(state)
    context  = select_and_redact_context(task, state)
    proposal = ask_model(context, permitted_tool_schemas(task))

    if proposal.kind == "finish":     # verify before believing
        evidence = run_required_checks_on_current_artifact(task, state)
        ...

    call     = validate_tool_name_and_arguments(proposal)
    decision = authorize(task, call, current_resource_versions())
    if decision.requires_approval: await_scoped_approval(call, state.deadline)
    result = execute_in_sandbox(call, timeout=remaining_deadline(state))
    state.record(redact(result)); checkpoint(state)

emit_report(state.status, state.artifacts, state.evidence, state.limitations)
```

<!--
Point out the three gates: validate arguments, authorize, then execute in the
sandbox. This is pseudocode, not a security-complete implementation.
-->

---

## What the loop guarantees

- Deadline, denial, cancellation, and execution errors move to an explicit
  stopped or recoverable state **and still produce a report**.
- Limits are enforced inside model calls, verification, and tools — not only
  at loop boundaries.
- Write intent is persisted before an external mutation; uncertain results are
  reconciled on recovery.

Checkpointing after a call alone cannot ensure exactly-once execution.

---

## Tool design: narrow interfaces

- Split **read-only** tools from tools that mutate files, repositories, or
  external systems.
- Require explicit arguments instead of arbitrary shell or network access.
- Validate arguments and canonicalize paths before executing.
- Return structured results: status, changed resources, concise diagnostics.
- Set time, output, retry, and concurrency limits.
- Gate irreversible or externally visible operations — publish, deploy,
  delete, send — behind explicit approval.

Tool descriptions should say what a tool **cannot** do. Composing instructions
must not create new authority.

---

## Example tool contract

Request — a named check, not a model-supplied shell command:

```json
{ "name": "run_check",
  "arguments": { "check_id": "orders-unit-tests",
                 "artifact_revision": "candidate-007" } }
```

Result — structured, with an explicit status:

```json
{ "status": "passed", "exit_code": 0, "check_id": "orders-unit-tests",
  "artifact_revision": "candidate-007", "duration_ms": 842,
  "summary": "8 tests passed", "output_truncated": false }
```

Define distinct statuses for `failed`, `timed_out`, `cancelled`, and
`unavailable`. None of them implies success.

<!--
Sample output, not a test result from this repository.
-->

---

## Even a fixed command runs repository code

- Run checks without production credentials and with restricted network access.
- Enforce path boundaries **after** canonicalization.
- Account for symlinks.
- Prevent races between checking a path and opening it.

The harness maps `orders-unit-tests` to a reviewed command in trusted
configuration, rejects unknown identifiers and extra arguments, and confirms
the candidate revision is current.

---

## Context management: order matters

Supply authoritative, task-relevant material first:

1. task description and acceptance criteria;
2. repository instructions and current working state;
3. relevant source, configuration, and tests;
4. tool results and verification evidence.

Summarize large output, but preserve exact errors, commands, and file
locations. Label untrusted content — issue comments, web pages, logs,
generated files — so it cannot override harness policy.

Keep secrets, tokens, private customer data, and unrelated repository content
out of model context.

---

## Separate context by lifetime and trust

| Context class | Example | Handling |
| --- | --- | --- |
| Trusted configuration | Tool allowlist, approval rules | Loaded by the harness; not agent-editable |
| Task-local state | Plan, current diff, failed check | Checkpoint with task ID and revision |
| Retrieved evidence | Source snippets, issue text, web content | Label origin; embedded instructions are untrusted |
| Durable memory | Reviewed project convention | Store with provenance and expiry/review rules |

Do not turn a model's inference into a stored fact. Re-read mutable resources
before acting on an old summary. Partition retrieval, caches, and memory by
tenant and authorization scope.

---

## Memory and skills are scoped, not authority

<div class="row">
<div class="node indigo">Task-local state<span>plan, revisions, failed checks, remaining budget</span></div>
<div class="node violet">Durable memory<span>reviewed convention with provenance and expiry</span></div>
<div class="node cyan">Retrieved evidence<span>code, issues, traces — untrusted instructions</span></div>
<div class="node green">Reviewed skills<span>versioned procedure, references, expected output</span></div>
</div>

- Memory saves rediscovery; a stale convention must be **checked against current
  code**, flagged, and corrected rather than followed.
- A skill is a reusable procedure, **not an extra permission**: its scripts and
  tool calls pass the same authorization and sandbox controls.
- Never store customer payloads, credentials, or one-time approval tokens.

<!--
Example from docs/examples.md: an old memory recommends an in-process duplicate
cache; the current multi-worker design contradicts it.
About nineteen minutes elapsed at the end of Part 2.
-->

---

<!-- _class: lead -->

# Part 3 — Safety and verification

Defense in depth, then evidence

---

## Defense in depth: six layers

<div class="row"><div class="node tight ghost">Prompt guidance<span>helpful, but never the control</span></div></div>
<div class="row"><div class="node tight indigo">Task contract and policy<span>scope, limits, stop conditions</span></div></div>
<div class="row"><div class="node tight violet">Dispatcher<span>allowlist, argument validation, authorization on every call</span></div></div>
<div class="row"><div class="node tight amber">Approval gate<span>bound to actor, action, payload, revision, expiry</span></div></div>
<div class="row"><div class="node tight cyan">Runtime and credentials<span>isolation, short-lived scoped secrets, restricted egress</span></div></div>
<div class="row"><div class="node tight green">Independent verification and audit<span>trusted checks outside agent-writable scope</span></div></div>

<p class="caption">Every layer must hold when the layer above it fails.</p>

---

## Prompts guide; runtime controls enforce

- Run work in an isolated environment with scoped filesystem and network access.
- Issue short-lived, least-privilege credentials only to tools that need them.
- Keep destructive operations behind a dedicated approval gate.
- Scan changed files for secrets before publishing changes.
- Treat external instructions as **data, not authority**.
- Preserve user changes; do not rewrite history or touch unrelated files.
- Log tool calls and policy decisions without logging sensitive values.

For security, data, money, production, or external communication: require a
human checkpoint before the consequential action.

---

## Threat exercises, not just intentions

| Threat | Enforcement | Exercise |
| --- | --- | --- |
| Retrieved page asks for credentials | No credentials in context; restricted egress | Supply adversarial text; confirm no upload |
| Path escapes the workspace | Canonical path checks, sandbox | Test traversal, absolute paths, symlinks |
| Model invents an admin tool | Registry allowlist, strict validation | Reject unknown tools and fields |
| Retry duplicates an external action | Idempotency keys, reconciliation | Time out after the destination accepts |
| Approval reused for another payload | Bind to actor, action, resource, payload, revision, expiry | Change payload; require a new decision |
| Agent weakens tests for a green result | Independent checks, review of test changes | Remove an assertion; acceptance must fail |

---

## Approval is scoped, not a session-wide grant

- Show the reviewer the **exact** diff or external action, expected effects,
  and available evidence.
- Reject stale approvals; recheck authorization immediately before execution.
- Treat model output as untrusted input to downstream systems: validate
  structured data and escape rendered content.

<!--
Tie back to the "recheck scope and current state" step in the architecture.
-->

---

## Verification is part of the task

| Change | Typical evidence |
| --- | --- |
| Application behavior | Focused tests plus a manual exercise of the changed path |
| Library or API | Unit and integration tests, compatibility checks, examples |
| Configuration or workflow | Syntax validation, dry run, review of permissions |
| Documentation | Link and Markdown checks, accuracy and readability review |

Start with targeted checks while iterating; run broader checks once the change
is complete. **If a check cannot run, report the reason instead of implying
success.**

---

## Verify the harness, not only its artifacts

Test deterministic enforcement independently of the model:

- denied actions never reach adapters;
- timeouts release resources;
- cancellation stops dispatching new work.

Use fake adapters to exercise failure handling without real external writes.

Maintain an evaluation set: successful tasks, ambiguous requests, unavailable
tools, adversarial content, stale approvals, budget exhaustion. Keep evaluation
checks outside agent-writable scope.

---

## Evidence has a revision

- Associate every result with the **exact artifact revision**.
- Edits after a check invalidate the affected evidence.
- For nondeterministic runs, compare repeated trials on task success, unsafe
  attempted actions, actual unauthorized effects, latency, and cost.
- Combine automated checks with acceptance review, especially for usability or
  business policy that tests do not capture.

A passing command is one piece of evidence, not a conclusion.

<!--
About twenty-seven minutes elapsed at the end of Part 3.
-->

---

<!-- _class: lead -->

# Part 4 — Operations

Reports, observability, budgets, and rollout

---

## Final report template

```text
Status: completed (patch only)
Changed: src/orders.py and tests/test_orders.py
Evidence: orders-unit-tests passed on candidate-007; diff reviewed
Scope: negative quantities rejected; zero/positive cases covered
Not performed: publication, deployment, or production access
Limitations: no end-to-end checkout-system test was run
```

Report the user-visible outcome, what changed, validation and its result,
what was intentionally left alone, and unresolved risks.

Only emit this status if it was actually observed; otherwise report `blocked`
or `partial` with the reason and preserved artifacts.

---

<!-- _class: dense -->

## Metrics worth collecting

| Metric | What it reveals | Caution |
| --- | --- | --- |
| Verified completion rate | Useful outcomes vs. acceptance criteria | Self-reported completion is not verification |
| Unauthorized effects, blocked attempts | Enforcement failures, boundary probing | Zero incidents does not prove safety |
| Human intervention rate | Ambiguity, missing tools, escalation | Lower is not always better |
| Cost per verified completion | Efficiency including retries | Cheap failed runs mislead per-run cost |
| Median and tail latency | Experience, slow dependencies | Separate execution from approval waiting |
| Retry and repeated-action rate | Flaky tools, unproductive loops | Inspect duplicates, not only averages |

Log task ID, tool-call ID, revision, decision, duration, status, and usage.

<!--
Keep sensitive payloads out of routine telemetry; apply access controls and
retention limits to diagnostic artifacts. Collect decision summaries and
observable evidence rather than private model reasoning.
-->

---

## Budgets and rollout

- Set per-task and per-tenant budgets; reserve capacity for verification and
  reporting, not only generation.
- Queue work and cap concurrency so agents cannot overwhelm dependencies.
- Start with an offline evaluation set, then a read-only or draft-only mode.
- Add scoped writes only after the relevant controls have been tested.
- Keep a kill switch plus a recovery procedure for in-flight operations:
  stopping the model cannot undo an already accepted write.

---

## Common pitfalls and better defaults

| Pitfall | Better default |
| --- | --- |
| A long prompt is the only safety control | Enforce policy in dispatchers, credentials, runtime boundaries |
| Arbitrary shell access is the default tool | Start with narrow typed tools; isolate necessary shell use |
| Every failure is retried | Classify failures; reconcile uncertain writes |
| More context or more agents must help | Measure outcomes; add only relevant information |
| Framework choice is the whole architecture | Assign ownership of state, authorization, verification, operations |

Prefer one agent and deterministic orchestration until measured evidence
justifies more complexity.

<!--
About thirty-two minutes elapsed at the end of Part 4.
-->

---

<!-- _class: lead -->

# Part 5 — Worked example

Retry-safe webhook ingestion, from request to approved draft PR

---

<!-- _class: dense -->

## One bounded request

> Make payment webhook ingestion safe to retry, including concurrent deliveries
> and a process restart. Keep signature validation. Add a migration, regression
> tests, and operator documentation. Publish a draft pull request only after I
> approve the exact candidate. No production access, merge, or deploy.

| Contract element | Agreed scope |
| --- | --- |
| Workspace | Isolated checkout, recorded baseline revision, existing user changes preserved |
| Acceptance | Deduplicate by `(tenant_id, provider, event_id)`; repeated or concurrent deliveries create one local job |
| Allowed edits | Handler, migration, regression tests, operator documentation — four files |
| Required checks | Webhook regressions, tenant isolation, migration compatibility, diff review, secret scan |
| External effects | Draft-PR publication needs a separate, scoped approval; no merge or deploy |

<!--
A hypothetical order service. Ambiguity about conflicting payloads went back to
the owner instead of being invented by the agent.
-->

---

## Act, verify, repair — until the evidence holds

<div class="row">
<div class="node indigo">Reproduce<span>duplicate and concurrent deliveries fail on the baseline</span></div>
<div class="arrow">→</div>
<div class="node amber">Candidate 1<span>“look up the receipt, then insert” still races</span></div>
<div class="arrow">→</div>
<div class="node violet">Repair<span>unique key plus one receipt/job transaction</span></div>
<div class="arrow">→</div>
<div class="node green">Candidate 2<span>required checks pass on a frozen revision</span></div>
</div>

<div class="row">
<div class="node rose">Waiting for approval<span>candidate frozen, evidence retained, nothing published</span></div>
</div>

- The racy candidate is deliberate: the harness keeps the failure **visible**
  and blocks completion instead of accepting a plausible patch.
- Editing after a check **invalidates** the affected evidence.
- Passing agent-written tests is not acceptance: independent checks, diff
  review, migration risk, and secret scanning all apply.

---

## Restart, approve, reconcile

- The checkpoint holds task and candidate IDs, evidence references, the approval
  request, and consumed budget — not a prose conversation summary.
- On resume the harness rechecks authorization, deadline, workspace revision,
  and evidence freshness. No fresh budget, no reused approval.
- Approval binds actor, action, repository and branch, candidate revision, diff
  payload, and expiry. **Approval to publish is not approval to merge or deploy.**
- If publication times out, reconcile the stored operation ID before retrying;
  if the outcome cannot be established, report it as unknown.

<!--
A checkpoint alone does not guarantee exactly-once external effects; the adapter
must deduplicate or reconcile.
-->

---

<!-- _class: dense -->

## What the harness added

| Without these controls | Contribution of the harness |
| --- | --- |
| Trust stale advice or rediscover project practice | Scoped, provenance-bearing memory checked against current evidence |
| Improvise the same debugging and migration steps | Reviewed skills supply procedures without extra authority |
| Accept a plausible patch after one green test | A bounded act–verify–repair loop exposes the concurrency failure |
| Treat retrieved instructions as trusted | Runtime guardrails deny unauthorized effects before execution |
| Lose progress or blindly retry publication | Durable task state and external-effect reconciliation |
| Report success without saying what ran | Revision-bound evidence and a human checkpoint |

It does not eliminate model mistakes; it makes them detectable, bounds
authority, and keeps unverified outcomes explicit.

---

## Where the boundary usually sits

| Product | Bounded task | Human boundary |
| --- | --- | --- |
| Repository maintenance | Narrowly scoped bug-fix patch | Diff review; separate approval to publish |
| Support copilot | Draft from approved help content | Human approves sending or refunds |
| Analytics assistant | Answer from approved datasets | Restrict sensitive rows and exports |
| Incident triage | Summarize and propose a runbook step | Operator authorizes production mutations |

Read-only is not risk-free: retrieval can expose private data and queries can
exhaust resources.

<!--
About thirty-six minutes elapsed at the end of Part 5.
-->

---

<!-- _class: lead -->

# Part 6 — Choosing a harness

Layers, rubric, decision tree, and the first release

---

## Compare within the layer you need

<div class="row">
<div class="node cyan">Coding runtimes and toolkits<span>Claude Code · Codex · Copilot cloud agent · Gemini CLI · OpenCode · OpenHands SDK · Pi · DeepSeek (preview)</span></div>
<div class="node violet">Application-agent frameworks<span>LangGraph · Deep Agents · Google ADK · OpenAI Agents SDK · Strands · Pydantic AI · CrewAI</span></div>
<div class="node green">Managed hosting<span>Foundry Agent Service · Agent Framework is a separate, code-first choice</span></div>
</div>

A framework's extensibility does not replace a ready-made coding environment,
and a managed service does not replace application-owned authorization.

Five implementation chapters plus dedicated framework and coding-runtime
sections back this map; the evaluation page scores them layer by layer.

---

## The rubric: weights before winners

<div class="row"><div class="col">
<div class="bar indigo w25">Control 25% — scoped tools, approvals, execution boundaries</div>
<div class="bar violet w25">Recovery 25% — durable state, resume, checkpoints, replay</div>
<div class="bar cyan w20">Extensibility 20% — programmable tools, adapters, replaceable parts</div>
<div class="bar green w15">Visibility 15% — inspectable events, logs, reviewable artifacts</div>
<div class="bar amber w15">Setup ease 15% — less application and infrastructure work</div>
</div></div>

**Final / 100 = (25C + 25R + 20E + 15V + 15S) / 5**, on an anchored 1–5 scale;
stars = final / 20. Unknown capabilities are "not assessed", never scored as
absent.

These are **dated editorial judgments about supplied mechanisms**, not benchmark
results. Reweight them for your own requirements.

---

<!-- _class: dense -->

## Scorecard highlights

| Layer | Option | C / R / E / V / S | Final |
| --- | --- | --- | --- |
| Coding runtime | OpenHands Software Agent SDK | 4 / 3 / 5 / 4 / 3 | 76/100 |
| Coding runtime | Claude Code, Codex, Gemini CLI | 4 / 3 / 4 / 4 / 4 | 75/100 |
| Hosted coding agent | GitHub Copilot cloud agent | 4 / 2 / 3 / 4 / 5 | 69/100 |
| Framework | OpenAI Agents SDK | 4 / 3 / 5 / 5 / 4 | 82/100 |
| Framework | LangChain Deep Agents | 4 / 4 / 5 / 4 / 3 | 81/100 |
| Framework | LangGraph | 3 / 5 / 5 / 4 / 2 | 78/100 |
| Managed hosting | Microsoft Foundry Agent Service | 4 / 4 / 3 / 4 / 3 | 73/100 |

Cost, model quality, latency, licensing, and data residency are excluded —
treat them as adoption gates. **A lower total can still be the better fit.**

<!--
Full scorecard, per-dimension rationale, and the capability quadrant are in
docs/harness-evaluation.md.
-->

---

## Start with your requirements

<div class="row"><div class="node indigo">Is the primary output a repository change?</div></div>

<div class="row">
<div class="node cyan">Yes<span>Delegated inside GitHub → Copilot cloud agent. Local or embedded loop → Claude Code, Codex, Gemini CLI, OpenCode, OpenHands SDK, Pi</span></div>
<div class="node violet">No, but Azure-managed hosting is required<span>Foundry Agent Service; confirm region, data handling, feature maturity</span></div>
</div>

<div class="row">
<div class="node green">Otherwise choose the abstraction<span>durable branching and checkpoints → LangGraph · packaged harness → Deep Agents · hierarchies and artifacts → Google ADK · handoffs and tracing → Agents SDK · typed tools → Pydantic AI · roles and flows → CrewAI</span></div>
</div>

<div class="row">
<div class="node rose">None fits<span>start from a small bounded tool loop; do not adopt a larger harness because it scores well</span></div>
</div>

---

<!-- _class: dense -->

## Turn a shortlist into evidence

1. **Pin the setup** — runtime revision, model, tools, policies, storage,
   deployment, budget.
2. **Run representative tasks** — success, ambiguity, denied tool use,
   malicious retrieved instructions, budget exhaustion.
3. **Exercise recovery** — restart during a pending approval and around a mock
   external write. Session resume is not rollback.
4. **Compare outcomes** — accepted-task rate, unauthorized effects, human
   intervention, cost per accepted task, latency.
5. **Reweight and decide** — pilot with limited permissions; expand only after
   the controls work.

Hard requirements — residency, licensing, identity, network — override every
recommendation.

---

<!-- _class: dense -->

## Implementation checklist

- [ ] Task schema and acceptance criteria
- [ ] System policy, escalation rules, stop conditions
- [ ] Tool contracts, permission model, approval gates
- [ ] Sandbox, credential, network, filesystem boundaries
- [ ] Context selection, redaction, retention
- [ ] Limits, retries, cancellation, recovery
- [ ] Structured event logs and audit trail
- [ ] Verification strategy and failure reporting
- [ ] Metrics: success, safety, cost, latency, retries
- [ ] Representative evaluations, including adversarial cases
- [ ] Tenant isolation, approval expiry, safe resume
- [ ] Rollout stages, kill switch, external-effect recovery

---

## A practical first milestone

- one task type,
- one isolated workspace,
- a small tool allowlist,
- one independent verification path, and
- a report a human can audit.

Do not expand authority just because the model can request more tools.

<!--
About forty-one minutes elapsed at the end of Part 6.
-->

---

<!-- _class: lead -->

# Questions

Guide and chapters: [github.com/frkim/agent-harness](https://github.com/frkim/agent-harness)

Foundations · Execution · Safety and verification · Examples · Operations
Implementation chapters · Harness evaluation

<!--
Keep the checklist and scorecard slides handy for follow-up questions about
scoping a first release.
-->
