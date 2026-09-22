# Operations

[Back to the guide](../README.md)

## Progress and final reports

Progress updates should be short and factual. A useful update records completed
work, the next verification step, and any blocker. The final report should
include:

- the user-visible outcome;
- the files or systems changed;
- validation performed and its result;
- items intentionally not changed; and
- unresolved risks, limitations, or follow-up actions.

Do not claim that a command, test, review, or deployment succeeded unless the
harness has recorded its successful result.

An illustrative report for the [order-service task](examples.md#worked-example-a-repository-maintenance-agent) would be:

```text
Status: completed (patch only)
Changed: src/orders.py and tests/test_orders.py
Evidence: orders-unit-tests passed on candidate-007; diff reviewed
Scope: negative quantities rejected; zero/positive cases covered
Not performed: publication, deployment, or production access
Limitations: no end-to-end checkout-system test was run
```

Only emit this status and evidence if they were actually observed. If a required
check cannot run, report `blocked` or `partial` with the reason and preserved
artifacts instead.

## Operating a harness

### Observability and cost

Record structured events with task ID, tool-call ID, timestamp, artifact
revision, policy decision, duration, status, and usage. Keep sensitive payloads
out of routine telemetry; apply access controls and retention limits to
diagnostic artifacts. Collect decision summaries and observable evidence rather
than private model reasoning.

| Metric | What it reveals | Interpretation caution |
| --- | --- | --- |
| Verified task completion rate | Useful outcomes against acceptance criteria | A model's self-reported completion is not verification |
| Unauthorized effects and blocked attempts | Enforcement failures and attempted boundary crossings | Zero observed incidents does not prove safety |
| Human intervention rate | Ambiguity, missing tools, or appropriate escalation | Lower is not always better for high-impact work |
| Cost per verified completion | Efficiency including failed runs and retries | Cheap failed runs can make per-run cost misleading |
| Median and tail latency | User experience and slow external dependencies | Separate execution time from approval waiting time |
| Retry and repeated-action rate | Flaky tools or unproductive loops | Inspect duplicates and timeouts, not only averages |

Set per-task and per-tenant budgets. Reserve capacity for verification and
reporting rather than spending the entire budget on generation. Queue work and
cap concurrency so agents cannot overwhelm downstream services.

### Rollout and architecture choices

Start with an offline evaluation set, then run in a read-only or draft-only mode.
Introduce scoped writes only after the relevant controls have been tested.
Retain a kill switch that stops new dispatches and a recovery procedure for
in-flight operations; stopping the model cannot undo an already accepted write.

Prefer one agent and deterministic orchestration until measured evidence
justifies more complexity. When adding specialist agents, give each explicit
inputs, outputs, permissions, and budget. Use isolated workspaces for parallel
edits and one owner for integration. Delegation must not expand the parent's
authority, and aggregate budgets must include all child runs.

| Common pitfall | Better default |
| --- | --- |
| A long prompt is the only safety control | Enforce policy in dispatchers, credentials, and runtime boundaries |
| Arbitrary shell access is the default tool | Start with narrow typed tools; isolate any necessary shell execution |
| Every failure is retried | Classify failures and reconcile uncertain writes |
| More context or more agents is assumed to help | Measure outcomes and add only task-relevant information or specialization |
| Framework selection is treated as the whole architecture | Explicitly assign ownership of state, authorization, verification, and operations |

## Implementation checklist

When building a harness, define the following before expanding agent autonomy:

- [ ] Task schema and acceptance criteria
- [ ] System policy, escalation rules, and stop conditions
- [ ] Tool contracts, permission model, and approval gates
- [ ] Sandbox, credential, network, and filesystem boundaries
- [ ] Context selection, redaction, and retention strategy
- [ ] Execution limits, retries, cancellation, and recovery behavior
- [ ] Structured event logs and audit trail
- [ ] Verification strategy and failure reporting
- [ ] Metrics for task success, safety violations, cost, latency, and retries
- [ ] Representative evaluations, including adversarial and failure cases
- [ ] Tenant isolation, approval expiry, and safe resume semantics
- [ ] Rollout stages, kill switch, and external-effect recovery procedures

Begin with a small set of deterministic tools and a narrow task category.
Expand capabilities only after observing reliable behavior and adding controls
for the new risk.

A practical first milestone is one task type, one isolated workspace, a small
tool allowlist, one independent verification path, and a report that a human can
audit. Do not expand authority just because the model can request more tools.
