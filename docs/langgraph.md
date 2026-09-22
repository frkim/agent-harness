# LangGraph: stateful orchestration

[Back to the guide](../README.md) · [Recovery and retries](foundations.md#recovery-and-retries)

## What the implementation provides

LangGraph is a low-level orchestration framework and runtime for stateful
workflows. Applications define state, nodes, and edges, then compile the graph.
Nodes can perform deterministic work, call models, or invoke tools.

It is useful when a harness needs explicit routing, resumable state, or human
checkpoints. It is not a ready-made coding CLI or an authorization system: the
application still supplies tools, execution isolation, and acceptance checks.

## Mapping the harness

For a workflow that prepares a change and waits for approval:

1. Put the task, candidate revision, and verification evidence in graph state.
   Use separate nodes for proposing work, checking it, and executing approved
   external actions.
2. Configure a checkpointer and supply a task-scoped `thread_id` when invoking
   the graph. Bind that identifier to the authenticated caller in the application.
3. Use `interrupt(...)` before the protected action to return the proposed
   payload for review. The application authenticates the reviewer and records
   the scoped decision.
4. Resume with `Command(resume=...)` and the same thread ID. Branch explicitly
   on approval or rejection and revalidate the action before executing it.

An interrupt is a pause, not an approval policy. The application must decide
what a resume value means and prevent an unauthorized caller from resuming a
different user's task.

## Persistence and authority boundaries

Checkpointers retain thread-scoped graph state. Stores can hold application
data across threads; neither automatically supplies tenant authorization.
An in-memory checkpointer does not survive a process restart, so choose durable
storage when recovery is required.

On resume, an interrupted node starts again from its beginning. Code before
the interrupt may execute again. Keep consequential side effects behind the
approval branch and design replay-sensitive operations for idempotency and
reconciliation. Checkpoints do not roll back writes in external systems.

The graph controls orchestration, not operating-system access. Apply the
[tool and runtime boundaries](execution.md#tool-design) even when routing and
persistence are handled by LangGraph.

## Verification exercise

Pause before a mock publication action, restart with a durable checkpointer,
and resume the same thread. Reject the action and confirm no publication occurs.
Repeat with approval and count external effects. Simulate a crash after a write
but before the next checkpoint, checking that replay does not duplicate it.
Also confirm that a different caller cannot inspect or resume that thread.

## Official sources

- [LangGraph overview](https://docs.langchain.com/oss/python/langgraph/overview)
- [Persistence](https://docs.langchain.com/oss/python/langgraph/persistence)
- [Interrupts and replay behavior](https://docs.langchain.com/oss/python/langgraph/interrupts)

Choose the persistence backend and durability settings deliberately, and test
recovery with the same configuration used in deployment.
