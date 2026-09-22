# Claude: Claude Code and Agent SDK

[Back to the guide](../README.md) · [Repository maintenance example](examples.md#worked-example-a-repository-maintenance-agent)

Source review: **2026-09-22**, using official SDK documentation and examples.

## What the implementation provides

Claude is a model family; Claude Code is a coding-oriented agent harness.
The Claude Agent SDK exposes the Claude Code execution machinery for applications
rather than requiring them to assemble every model/tool turn themselves. The
Python SDK offers `query(...)` for one-shot interactions and `ClaudeSDKClient`
for interactive conversations.

This is a different integration choice from making a direct model API call.
Use Claude Code for interactive repository work, or the Agent SDK when an
application needs to initiate runs, consume results, and control tool use.

## Mapping the harness

For the repository maintenance task:

1. Supply a bounded task and an isolated checkout with no production credentials.
   Select the tools needed to inspect files, make the patch, and run checks.
2. Configure permissions explicitly. The SDK's `can_use_tool` callback can
   return `PermissionResultAllow` or `PermissionResultDeny` when the runtime
   requests a permission decision. Evaluate the tool's arguments, not merely
   its name.
3. Capture execution results and run independent checks against the resulting
   diff. A natural-language completion message is not regression evidence.
4. Return a patch and report. Publishing or deploying requires separate
   authorization and must not follow merely from permission to edit files.

Permission behavior depends on configured modes and rules. Do not assume a
callback prompts for every invocation, and do not use bypass modes when the
application relies on approval gates.

Configure filesystem settings explicitly as well as permissions. In the Python
SDK, `setting_sources=[]` disables filesystem setting sources; an explicit list
selects the intended sources. Do not assume omitted settings behave identically
across SDK/CLI versions or match an interactive setup. Review any project-local
configuration before allowing it to influence execution.

The SDK also exposes `PreToolUse` hooks for argument inspection and explicit
tool denial. These complement permission decisions through `can_use_tool`.
`PostToolUse` hooks can inspect or record outcomes but cannot retroactively
prevent an action that already occurred. Hooks do not replace runtime isolation.

## Persistence and authority boundaries

Session resumption depends on retained session data, not just knowledge of a
session identifier. Preserve the required transcripts through the supported
storage mechanism for the installed SDK and control access to them.

Resuming a conversation does not restore the working tree or undo an external
tool action. Reconcile the workspace and revalidate pending actions on resume.
Tool permissions are also not a substitute for operating-system isolation:
restrict filesystem access, network access, and credentials in the execution
environment. A shell tool can reach more capabilities than its name suggests.

## Verification exercise

In a disposable workspace, configure a denied write and confirm independently
that the file did not change. Exercise an allowed write separately. Resume a
saved session in a fresh process and check both conversational continuity and
the actual workspace state. Confirm a failed required check produces a blocked
or partial result rather than a success report.
Test a hook-denied action when the tool otherwise has an allow rule, and verify
that only the intended filesystem setting sources affect the run.

## Official sources

- [Claude Agent SDK Python reference](https://platform.claude.com/docs/en/agent-sdk/python)
- [Tool permission callback example](https://github.com/anthropics/claude-agent-sdk-python/blob/main/examples/tool_permission_callback.py)
- [Session restoration implementation](https://github.com/anthropics/claude-agent-sdk-python/blob/main/src/claude_agent_sdk/_internal/session_resume.py)
- [Explicit filesystem setting sources](https://github.com/anthropics/claude-agent-sdk-python/blob/main/examples/setting_sources.py)
- [PreToolUse and PostToolUse hook examples](https://github.com/anthropics/claude-agent-sdk-python/blob/main/examples/hooks.py)

Check session-storage and permission support against the SDK version you deploy;
source on the default branch may differ from an installed release.
