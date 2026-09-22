# Microsoft: Foundry Agent Service and Agent Framework

[Back to the guide](../README.md) · [Core model](foundations.md#core-model)

## What each layer provides

Microsoft Foundry Agent Service and Microsoft Agent Framework are related but
distinct:

| Layer | Role | What you still own |
| --- | --- | --- |
| Foundry Agent Service | Managed service for agents, including configuration-based prompt agents and custom-code hosted agents | Application authorization, tool credentials, data access, and acceptance checks |
| Microsoft Agent Framework | Code-first framework for agents, tools, sessions, and workflows | Deployment, persistence configuration, runtime isolation, and business policy |

A hosted agent can use Agent Framework, but Foundry is not another name for that
framework. Choose the managed service when you need hosted agent capabilities;
choose the framework when you need to express application orchestration in code.
They can be used together.

## Mapping the harness

For a support assistant that drafts an answer but cannot send it:

1. Define the task and acceptance criteria in the application. Give the agent
   only tenant-scoped ticket lookup and approved knowledge-retrieval tools.
2. In Agent Framework, configure an agent with a model client, tools, and a
   session. Reuse the session for related turns. A Foundry-backed client is one
   integration option.
3. If sending is later authorized, expose it as a separate tool with an approval
   requirement. Agent Framework's function-approval flow surfaces a request to
   the application, which supplies an approve or reject response.
4. Independently check policy citations and sensitive data before releasing the
   draft. Capture the tool outcomes and the review decision, not just the final
   model response.

Foundry's MCP integration has its own approval request/response flow. Configure
approval requirements explicitly; do not assume that a local framework function
approval also governs every remote MCP tool.

## Persistence and authority boundaries

Foundry conversation state and Agent Framework sessions are different storage
boundaries. An in-process session is not automatically durable across restarts;
select and test the relevant storage or checkpoint mechanism. Persist pending
approvals with the exact action, caller, resource, and expiry.

Managed hosting does not make a tool's external credentials least-privileged.
The application must authenticate the approver, recheck authorization before
execution, and reconcile uncertain writes rather than assuming a checkpoint
undoes external effects. Keep examples from older threads/runs APIs separate
from conversation/Responses API examples.

## Verification exercise

Use a harmless mock send tool. Reject an approval and confirm that it is never
called; approve a specific payload and confirm that only that payload is sent.
Change the payload or tenant and require a new decision. Separately restart the
application to check conversation recovery and pending approval handling.

## Official sources

- [Foundry Agent Service overview](https://github.com/MicrosoftDocs/azure-ai-docs/blob/main/articles/foundry/agents/overview.md)
- [Foundry MCP tools and approvals](https://github.com/MicrosoftDocs/azure-ai-docs/blob/main/articles/foundry/agents/how-to/tools/model-context-protocol.md)
- [Agent Framework function approval and sessions example](https://github.com/microsoft/agent-framework/blob/main/python/samples/02-agents/tools/function_tool_with_approval_and_sessions.py)

The example APIs evolve independently of the design principles. Check the
documentation for your selected service API and framework version.
