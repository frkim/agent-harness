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
  }
  section.lead {
    text-align: center;
  }
  section.lead h1 {
    font-size: 54px;
  }
  section.dense {
    font-size: 22px;
  }
  table {
    font-size: 22px;
  }
  section.dense table {
    font-size: 20px;
  }
  code {
    font-size: 0.85em;
  }
  footer {
    font-size: 16px;
  }
footer: "Agent Harness design guide · github.com/frkim/agent-harness"
---

<!-- _class: lead -->
<!-- _paginate: false -->

# Designing an Agent Harness

The control plane around an AI model

45 minutes · design guide walkthrough

<!--
Timing plan for the whole deck (45 minutes):
- Opening and framing: 4 min
- Part 1 Foundations: 9 min
- Part 2 Execution: 11 min
- Part 3 Safety and verification: 9 min
- Part 4 Operations and rollout: 7 min
- Part 5 Worked example and close: 5 min
Leave the last 3-5 minutes of the slot for questions.
-->

---

## What you get from this talk

- A shared definition of a **harness** and the concerns it owns.
- A bounded execution loop you can implement in one service.
- Controls that hold when the model behaves unexpectedly.
- Evidence and reporting rules that make results auditable.
- A checklist to scope a first release.

This is a design guide, not a framework: no SDK, no installation step.
Examples are illustrative contracts and pseudocode.

<!--
Set expectations: attendees leave with vocabulary and a checklist,
not with a library to import.
-->

---

## Agenda

| Part | Topic | Time |
| --- | --- | --- |
| 1 | Foundations: why a harness, core model, lifecycle | 9 min |
| 2 | Execution: task contracts, controller, tools, context | 11 min |
| 3 | Safety and verification | 9 min |
| 4 | Operations: reports, observability, rollout | 7 min |
| 5 | Worked example, implementations, checklist | 5 min |
| — | Questions | remaining time |

<!--
Announce that every part maps to one chapter in docs/, so the audience
can follow up after the session.
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

```text
User -> Task intake (acceptance criteria) -> Controller <-> Model
                        Context selector ->     |
                                    Policy and budget gate
                  denied |               | allowed      | approval required
                   Stop  |        Tool dispatcher       Human approval
                         |   (isolated runtime)              |
           read-only | scoped write | verification    recheck scope/state
                         |
           Structured results -> Controller -> audit events + final report
```

- The **controller** owns task state and scheduling.
- The **dispatcher** checks identity, arguments, authorization, and limits on
  every invocation.
- The **runtime** enforces filesystem, process, and network boundaries even if
  the model or a tool misbehaves.

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

```text
Received -> Orienting -> Planning -> Acting -> Verifying -> Completed
                            |          |           |
                         Waiting <- clarification or approval needed
                            |          |           |
        Stopped: rejected, expired, denied, cancelled, budget exhausted
```

- Cancellation and deadlines apply in **every** nonterminal state.
- A stopped run may keep useful artifacts, but must never be reported as
  completed.
- A verifier failure leads to a bounded repair attempt or a clear blocker —
  not an indefinite retry loop.

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
About thirteen minutes elapsed at the end of Part 1.
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

<!--
About twenty-four minutes elapsed at the end of Part 2.
-->

---

<!-- _class: lead -->

# Part 3 — Safety and verification

Defense in depth, then evidence

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
About thirty-three minutes elapsed at the end of Part 3.
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
About forty minutes elapsed at the end of Part 4.
-->

---

<!-- _class: lead -->

# Part 5 — Worked example and next steps

---

## Worked example: one bounded request

> Reject negative quantities in the total calculator, preserve zero-quantity
> behavior, and prepare a patch for review. Do not publish or deploy anything.

1. **Receive** — record both requirements and the no-publication constraint.
2. **Orient** — inspect the calculator, tests, and working-tree changes.
3. **Plan** — regression test, smallest guard, relevant checks only.
4. **Act** — edit only the two authorized files in an isolated checkout.
5. **Verify** — the new test fails on the baseline and passes on the candidate.
6. **Report** — patch, checked revision, actual results, limitations.

Publication would be a **separately authorized** action with its own scoped,
expiring approval.

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

---

<!-- _class: lead -->

# Questions

Guide and chapters: [github.com/frkim/agent-harness](https://github.com/frkim/agent-harness)

Foundations · Execution · Safety and verification · Examples · Operations

<!--
Keep the checklist slide handy for follow-up questions about scoping a
first release.
-->
