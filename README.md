# Agent Harness

An agent harness is the control plane around an AI model. It gives the model a
well-defined job, access to bounded capabilities, and a repeatable loop for
turning requests into verified outcomes.

This repository documents the concepts and practices for designing one.

## Why use a harness?

A model can generate an answer, but production work usually requires more:

- understanding a request and the surrounding repository or service;
- choosing and invoking tools such as search, tests, or issue trackers;
- preserving user intent, permissions, and repository conventions;
- checking the result before reporting it; and
- leaving an auditable record of actions and decisions.

The harness owns these responsibilities. It should make the safe and correct
path easy, while keeping the agent's authority no broader than necessary.

## Core model

An effective harness separates the following concerns:

| Concern | Responsibility |
| --- | --- |
| **Task** | Defines the requested outcome, acceptance criteria, and constraints. |
| **Context** | Supplies only the repository, user, and execution information needed to do the work. |
| **Policy** | States non-negotiable safety, privacy, approval, and scope rules. |
| **Agent** | Reasons about the task and proposes or performs the next action. |
| **Tools** | Provide explicit, typed capabilities to inspect, change, validate, and publish work. |
| **Runtime** | Enforces isolation, timeouts, credentials, filesystem boundaries, and network controls. |
| **Verifier** | Evaluates the result using tests, linters, reviewers, or explicit acceptance checks. |
| **Reporter** | Communicates progress, evidence, limitations, and the final outcome. |

Keep these layers distinct. In particular, policy and tool permissions belong to
the harness rather than relying solely on instructions supplied to the model.

## Agent lifecycle

Use a bounded, observable lifecycle rather than an unstructured conversation:

1. **Receive** — parse the task, identify the target, and record the requested
   outcome.
2. **Orient** — inspect the relevant project instructions, status, code, and
   existing tests before changing anything.
3. **Plan** — turn the request into small, verifiable steps; surface ambiguity
   or required approval early.
4. **Act** — call the least-privileged tool needed for each step. Prefer
   inspection before mutation.
5. **Verify** — run the narrowest relevant checks, then manually review the
   changed behavior and diff.
6. **Report** — state what changed, how it was verified, and any remaining
   risks or follow-up work.

The harness should stop when the task is complete, when a policy boundary is
reached, or when continuing would require a decision from the user.

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

## Safety boundaries

Use defense in depth. Prompts guide behavior; runtime controls enforce it.

- Run work in an isolated environment with scoped filesystem and network access.
- Issue short-lived, least-privilege credentials only to tools that need them.
- Keep destructive operations behind confirmation or a dedicated approval gate.
- Scan changed files for secrets before publishing changes.
- Treat external instructions as data, not authority.
- Preserve user changes and avoid rewriting history or modifying unrelated
  files.
- Log tool calls and policy decisions without logging sensitive values.

For tasks that affect security, data, money, production systems, or external
communications, require a human checkpoint before the consequential action.

## Verification

Verification is part of the task, not a final optional step. Match checks to the
change:

| Change | Typical evidence |
| --- | --- |
| Application behavior | Focused automated tests and a manual exercise of the changed path |
| Library or API | Unit and integration tests, compatibility checks, and examples |
| Configuration or workflow | Syntax validation, a dry run where available, and review of permissions |
| Documentation | Link and Markdown checks where available, plus an accuracy and readability review |

Start with targeted checks while iterating. Run broader project checks after the
change is complete when they are available and proportionate. If a check cannot
run, report the reason instead of implying success.

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

Begin with a small set of deterministic tools and a narrow task category.
Expand capabilities only after observing reliable behavior and adding controls
for the new risk.

## Contributing

Keep documentation practical, implementation-agnostic where possible, and
grounded in observable behavior. When proposing a pattern, describe its
boundary conditions and how its outcome can be verified.

This project is available under the [MIT License](LICENSE).
