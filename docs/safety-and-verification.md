# Safety and verification

[Back to the guide](../README.md) · Next: [Examples](examples.md)

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

| Threat or failure | Enforcement | Verification exercise |
| --- | --- | --- |
| A retrieved page asks the agent to upload credentials | No credentials in context; restricted egress and tool authorization | Supply adversarial retrieved text and confirm no upload occurs |
| A proposed file path escapes the workspace | Canonical path checks and sandbox boundaries | Test traversal, absolute paths, and symlink escapes |
| The model invents an administrative tool | Registry allowlist and strict argument validation | Reject unknown tools and unexpected fields before execution |
| A retry creates a duplicate external action | Idempotency keys and destination reconciliation | Simulate a timeout after the destination accepts a write |
| An approval is reused for a different payload | Bind approval to actor, action, resource, payload, revision, and expiry | Change the payload after approval and require a new decision |
| An agent weakens tests to obtain a green result | Independent checks and review of test changes | Confirm removal of a required assertion cannot satisfy acceptance |

Approval is not blanket authority for the rest of a session. Show the reviewer
the exact diff or external action, expected effects, and available evidence.
Reject stale approvals and recheck authorization immediately before execution.
Treat model output as untrusted input to downstream systems as well: validate
structured data and escape rendered content.

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

### Verify the harness as well as its artifacts

Test deterministic enforcement independently of the model: denied actions must
never reach adapters, timeouts must release resources, and cancellation must
stop dispatching new work. Use fake adapters to exercise failure handling
without performing real external writes.

Maintain a representative evaluation set containing successful tasks, ambiguous
requests, unavailable tools, adversarial content, stale approvals, and budget
exhaustion. For nondeterministic runs, compare repeated trials on task success,
unsafe attempted actions, actual unauthorized effects, latency, and cost.
Keep evaluation checks outside agent-writable scope.

A passing command is only one piece of evidence. Associate results with the
exact artifact revision; edits after a check invalidate affected evidence.
Combine automated checks with acceptance review, especially when requirements
involve usability or business policy that tests do not fully capture.
