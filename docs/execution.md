# Execution

[Back to the guide](../README.md) · Next: [Safety and verification](safety-and-verification.md)

## Task and execution contracts

Make the requested outcome machine-readable before giving the model tools.
The following JSON describes a hypothetical task in another repository; its
paths and test target are examples, not files or commands in this project.

```json
{
  "task_id": "task-042",
  "objective": "Reject a negative quantity when calculating an order total",
  "workspace": "/workspace/order-service",
  "acceptance_criteria": [
    "Negative quantities raise ValueError",
    "Zero and positive quantities keep their existing behavior"
  ],
  "allowed_write_paths": ["src/orders.py", "tests/test_orders.py"],
  "allowed_tools": ["read_file", "apply_patch", "run_check"],
  "verification_targets": ["orders-unit-tests"],
  "limits": {
    "max_model_turns": 12,
    "max_tool_calls": 30,
    "wall_time_seconds": 600,
    "max_output_bytes_per_call": 32768
  },
  "on_limit": "stop_and_report"
}
```

These values are starting points, not production defaults. Authenticate the
caller and intersect requested permissions with server-side policy; never let
a task grant itself access. Resolve check identifiers to trusted configuration
outside agent-writable files. Add model token/cost limits and per-tool deadlines
appropriate to the deployment.

### Bounded controller sketch

This pseudocode shows responsibility boundaries, not a security-complete
implementation. Each helper requires a concrete implementation and tests.

```text
task = authenticate_validate_and_scope(request, server_policy)
state = create_or_resume_task(task)

while not state.terminal:
    enforce_deadline_cancellation_and_remaining_budget(state)
    context = select_and_redact_context(task, state)
    proposal = ask_model(context, permitted_tool_schemas(task))
    record_model_usage(state, proposal.usage)

    if proposal.kind == "finish":
        evidence = run_required_checks_on_current_artifact(task, state)
        if acceptance_criteria_met(task, evidence):
            state.complete(evidence)
        else:
            state.record_failure_or_stop(evidence)
        continue

    call = validate_tool_name_and_arguments(proposal)
    decision = authorize(task, call, current_resource_versions())
    if decision.denied:
        state.stop(decision.reason)
        continue
    if decision.requires_approval:
        approval = await_scoped_approval(call, deadline=state.deadline)
        require_valid_approval_and_reauthorize(approval, task, call)

    reserve_tool_budget_or_stop(state, call)
    result = execute_in_sandbox(call, timeout=remaining_deadline(state))
    state.record(redact(result))
    checkpoint(state)

emit_report(state.status, state.artifacts, state.evidence, state.limitations)
```

Deadline, denial, cancellation, and execution errors must transition to an
explicit stopped or recoverable state and still produce a report. Enforce
limits inside model calls, verification, and tools as well as at loop
boundaries. Persist write intent before an external mutation and reconcile
uncertain results on recovery; checkpointing after a call alone cannot ensure
exactly-once execution.

## Tool design

Tools are the agent's real capabilities. Design them as narrow interfaces with
clear inputs, outputs, and side effects.

- Split read-only tools from tools that mutate files, repositories, or external
  systems.
- Require explicit arguments instead of allowing arbitrary shell or network
  access where a focused operation is possible.
- Validate arguments and canonicalize paths before executing an operation.
- Return structured results, including command status, changed resources, and
  concise diagnostic output.
- Set time, output, retry, and concurrency limits.
- Mark irreversible or externally visible operations—such as publishing,
  deploying, deleting, or sending messages—for explicit approval.

Tool descriptions should explain both what a tool can do and what it cannot.
The model must not be able to obtain additional authority merely by composing
instructions.

### Example tool contract

Prefer a named check to a model-supplied shell command. For the hypothetical
order service, `run_check` could accept:

```json
{
  "name": "run_check",
  "arguments": {
    "check_id": "orders-unit-tests",
    "artifact_revision": "candidate-007"
  }
}
```

The harness maps `orders-unit-tests` to a reviewed command in trusted
configuration. It rejects unknown identifiers and extra arguments, confirms
that `candidate-007` is the current immutable candidate, and records a result:

```json
{
  "status": "passed",
  "exit_code": 0,
  "check_id": "orders-unit-tests",
  "artifact_revision": "candidate-007",
  "duration_ms": 842,
  "summary": "8 tests passed",
  "output_truncated": false
}
```

This is sample output, not a test result from this repository. Define distinct
statuses for `failed`, `timed_out`, `cancelled`, and `unavailable`; none implies
success. Store redacted diagnostic artifacts when output is truncated.

Even a fixed test command executes repository code. Run it without production
credentials and with restricted network access. For file tools, enforce path
boundaries after canonicalization, account for symlinks, and prevent races
between checking a path and opening it.

## Context management

Context quality matters as much as context volume. Supply authoritative,
task-relevant material first:

1. task description and acceptance criteria;
2. repository-specific instructions and current working state;
3. relevant source, configuration, and tests;
4. tool results and verification evidence.

Summarize large or repetitive output, but preserve exact errors, commands, and
file locations needed to make decisions. Clearly label untrusted content—such
as issue comments, web pages, logs, and generated files—so it cannot override
the harness policy.

Avoid placing secrets, access tokens, private customer data, or unrelated
repository content in model context. Redact sensitive values in logs and tool
output.

### Separate context by lifetime and trust

| Context class | Example | Handling |
| --- | --- | --- |
| Trusted configuration | Tool allowlist and approval rules | Loaded by the harness; not editable by the agent |
| Task-local state | Plan, current diff, failed check | Checkpoint with task ID and artifact revision |
| Retrieved evidence | Source snippets, issue text, web content | Label origin and treat embedded instructions as untrusted |
| Durable memory | Reviewed project convention | Store only when permitted, with provenance and expiry/review rules |

When summarizing, retain unresolved questions, constraints, exact failure
evidence, and references back to originals. Do not turn a model's inference
into a stored fact. Re-read mutable resources before acting on an old summary.
Partition retrieval, caches, and memory by tenant and authorization scope to
avoid cross-user information leakage.
