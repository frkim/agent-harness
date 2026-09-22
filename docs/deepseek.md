# DeepSeek: supplying the model, building the harness

[Back to the guide](../README.md) · [Execution contracts](execution.md#task-and-execution-contracts)

## What the implementation provides

This chapter covers DeepSeek's Chat Completions and tool-calling API, not a
complete coding harness. The model can propose function calls, but the calling
application executes those functions and returns their results. The API alone
does not supply a shell executor, an approval system, or a durable task runtime.

Use this integration when you want to supply the model to a harness you build or
select. OpenAI-compatible request formatting does not by itself make a model
compatible with every coding agent's tool, persistence, or security assumptions.

## Mapping the harness

A minimal application-owned tool loop follows the
[bounded controller design](execution.md#bounded-controller-sketch):

1. Authenticate and scope the task, then send relevant messages and the schemas
   of permitted tools.
2. If the response proposes tool calls, validate each tool name and arguments
   against a trusted registry. Reject unknown functions and malformed input.
3. Authorize each action and obtain any required approval before execution.
   Run the tool with scoped credentials, isolation, and a deadline.
4. Add the assistant's tool-call message and the corresponding tool results to
   the conversation, preserving each call/result identifier, then continue.
5. Stop on verified completion, cancellation, denial, or budget exhaustion.
   Persist evidence and report the outcome separately from the model's answer.

Schema conformance is not authorization. Even syntactically valid arguments can
refer to another tenant's data or a destructive operation. Apply the same checks
to every call, including multiple calls in a single response.

## Persistence and authority boundaries

The documented Chat Completions multi-round flow is stateless: the caller
accumulates messages and resends the relevant history. The harness therefore
owns durable task state, conversation retention, tool outcomes, and approvals.
Do not assume a successful API request checkpoints an external operation.

Record write intent before consequential actions and reconcile uncertain
results after timeouts. Model-call retries must not blindly re-execute earlier
tool calls. Keep service credentials outside model context and redact tool
results before returning them.

## Verification exercise

Start with a deterministic mock lookup tool. Check that the harness associates
the result with the correct tool-call identifier and continues the conversation.
Inject unknown functions, invalid arguments, and unauthorized resource requests;
none should reach the executor. Restart the application and reconstruct its
conversation state, then simulate a write timeout and verify reconciliation
rather than duplicate execution.

## Official sources

- [DeepSeek tool calls](https://api-docs.deepseek.com/guides/tool_calls/)
- [DeepSeek multi-round conversation](https://api-docs.deepseek.com/guides/multi_round_chat/)

Confirm tool-calling and message requirements for your selected model and API
mode. The application responsibilities above are design recommendations, not
claims that the API supplies those controls.
