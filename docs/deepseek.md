# DeepSeek Harness: a plugin-based agent runtime

[Back to the guide](../README.md) · [Execution contracts](execution.md#task-and-execution-contracts)

Source review: **2026-09-22**. The official repository describes a rapidly
changing **developer preview**, with compatibility-breaking changes expected.

## What the implementation provides

[DeepSeek Harness (`dsh`)](https://www.deepseek.com/harness/en/) is an open-source,
MIT-licensed agent harness, not merely DeepSeek's Chat Completions API. Its
[official repository](https://github.com/deepseek-ai/deepseek-harness) describes
an architecture built on Cordis in which model adapters, tools, session logging,
and the agent loop itself are replaceable plugins.

| Layer | Role | Boundary |
| --- | --- | --- |
| Cordis plugins | Supply services, typed events, and lifecycle-managed registrations | Loading a plugin extends the trusted execution surface; review its code and configuration |
| Profiles and bundles | Compose plugins for Web UI, headless runs, SDK, and ACP integrations | The effective composition determines capabilities; profiles do not all supply the same controls |
| Agent loop and tool pipeline | Assemble model context, invoke tools, and record results | Authorization and isolation depend on the mounted policy and execution providers |
| Session log and persistence | Derive model history and support transcripts, resume, and forks | Logged state is not rollback of filesystem or external-service changes |

The [architecture documentation](https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/architecture.md)
lists `web`, `headless`, `sdk`, `sdk-minimal`, and `acp` profiles. Most use the
shared base bundle for tools, persistence, sandbox and approval policy;
`sdk-minimal` deliberately has its own composition rather than that base.
TypeScript and Python SDKs integrate through the profile-based runtime.
Model support depends on the configured adapter; do not infer that every model
or tool protocol is interchangeable.

## Mapping the harness

For a patch-only repository maintenance task:

1. Select a runtime profile and review its resolved plugin configuration and
   overrides. Include only the model adapter, tools, and integrations needed.
2. Use a disposable environment with a scoped checkout, no production
   credentials, and independently enforced network and filesystem limits.
3. Configure the selected sandbox and approval providers. Check effective
   behavior rather than assuming that a profile name guarantees isolation.
4. Retain session events and verification evidence. Review the resulting diff
   and run independent acceptance checks before reporting completion.
5. Keep publication behind a separate authorization decision. Permission to
   edit a checkout must not implicitly authorize pushing or deployment.

## Persistence and authority boundaries

The runtime derives model-visible history from an append-only session event log.
Durable session events and live streaming events serve different purposes:
the architecture explicitly notes that a hard process loss before stream
settlement leaves no durable record of that unfinished attempt. Do not equate a
visible UI update with a completed, persisted action.

The official [safety notice](https://github.com/deepseek-ai/deepseek-harness/blob/master/SAFETY.md)
states that the preview **has not undergone a security audit and must not be
treated as secure or production-ready**. It can execute model-generated code,
load third-party plugins, and access exposed host resources. Sandboxes and
approval prompts reduce risk but are not guarantees; do not use the harness as
the sole security control for untrusted workloads.

On resume, reconcile workspace and external state and revalidate approvals.
Protect session logs as potentially sensitive data. Plugin unload cleanup and
session forks cannot undo a tool's external side effects.

## Direct API integration is a separate option

DeepSeek's documented Chat Completions tool-calling flow still leaves function
execution to the caller. If using that API without `dsh`, the application owns
the [bounded tool loop](execution.md#bounded-controller-sketch), tool validation,
authorization, approvals, execution, budgets, and durable state. The documented
multi-round API flow is stateless: the caller retains and resends message history.

Correlate each result with its tool-call identifier and reject unknown tools,
invalid arguments, or unauthorized actions before execution. Schema conformance
is not authorization, and OpenAI-compatible request formatting does not provide
another harness's security or persistence semantics.

## Verification exercise

In a disposable environment, record the runtime revision, profile, plugins, model,
and policy configuration. Attempt a denied write and network request and check
independently that neither succeeds. Restart a session and compare the recovered
history with the real workspace. Simulate interruption around a mock external
write and verify reconciliation rather than duplicate execution.

For evaluations, the repository's
[benchmark instructions](https://github.com/deepseek-ai/deepseek-harness/blob/master/BENCHMARK.md)
point to the Python SDK minimal variant and require separate workspaces and
session IDs for independent tasks. That file does not publish a score table.
Do not treat unsourced benchmark numbers as model-only performance: comparisons
must identify the dataset, model, harness revision, tools, budgets, and run setup.

## Official sources

- [DeepSeek Harness announcement](https://www.deepseek.com/harness/en/)
- [Repository and developer-preview status](https://github.com/deepseek-ai/deepseek-harness)
- [Architecture, profiles, and session semantics](https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/architecture.md)
- [Safety notice](https://github.com/deepseek-ai/deepseek-harness/blob/master/SAFETY.md)
- [Benchmark instructions](https://github.com/deepseek-ai/deepseek-harness/blob/master/BENCHMARK.md)
- [DeepSeek tool calls](https://api-docs.deepseek.com/guides/tool_calls/)
- [DeepSeek multi-round conversation](https://api-docs.deepseek.com/guides/multi_round_chat/)

The announcement was located through online search; runtime and safety details
above were checked against the official repository because direct retrieval of
the announcement page was unavailable. Recheck the selected release before use.
