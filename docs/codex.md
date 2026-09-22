# OpenAI Codex

[Back to the guide](../README.md) · [Safety and verification](safety-and-verification.md)

## What the implementation provides

Codex is a coding agent with a CLI and programmatic integration options. Its
TypeScript SDK drives the Codex executable through the noninteractive `exec`
path and consumes structured events. It is not the same library as the
general-purpose OpenAI Agents SDK.

The SDK exposes threads: `startThread()` begins a conversation, `run()` collects
the result, and `runStreamed()` exposes events as execution proceeds. Use the
CLI for repository tasks and the SDK when an application needs to coordinate
runs and consume their outcomes.

## Mapping the harness

For a patch-only maintenance job:

1. Prepare an isolated checkout and specify the working directory and task
   constraints. Avoid exposing credentials or unrelated repositories.
2. Configure sandbox mode and approval policy separately. Also review additional
   writable directories, network access, and web-search settings rather than
   treating any one option as the complete security policy.
3. Consume run events to track actions, failures, and usage. Apply application
   deadlines and budgets; do not infer success from the final text alone.
4. Inspect the diff and run independent acceptance checks on the exact candidate.
   Leave publication to a separately authorized step.

Noninteractive execution needs an explicit blocked-action policy. Confirm how
the selected CLI version handles approval requests; accepting an approval-policy
option in the SDK does not establish that the embedding application has an
interactive approval callback.

## Persistence and authority boundaries

`resumeThread(id)` refers to persisted conversation data. The SDK documents
local thread storage under `~/.codex/sessions`; retain the relevant state when
moving between processes or ephemeral workers, and protect it as task data.
A thread identifier alone does not recreate missing session files.

Conversation recovery is not workspace recovery. Re-read the checkout and
external resources before acting on resumed instructions. Sandbox enforcement
depends on the execution environment and configuration, while approval governs
whether an action may proceed. Neither removes the need for least-privilege
credentials and tool-side authorization.

## Verification exercise

Use a disposable repository and capture the event stream. Confirm that an
attempted out-of-workspace write and disallowed network request fail under the
chosen configuration. Restart and resume using the saved thread ID, then verify
the actual diff and tests. Check that an unavailable test or denied action is
reported accurately instead of being treated as completion.

## Official sources

- [Codex TypeScript SDK](https://github.com/openai/codex/tree/main/sdk/typescript)
- [SDK CLI execution adapter](https://github.com/openai/codex/blob/main/sdk/typescript/src/exec.ts)
- [Codex security: sandbox and approvals](https://developers.openai.com/codex/security)

Use documentation matching the installed CLI and SDK; available settings and
their effective enforcement can differ between environments.
