# Examples

[Back to the guide](../README.md) · Next: [Operations](operations.md)

## Worked example: a repository maintenance agent

Consider an order-service team receiving this request:

> Reject negative quantities in the total calculator, preserve zero-quantity
> behavior, and prepare a patch for review. Do not publish or deploy anything.

1. **Receive:** record the two behavior requirements and the no-publication
   constraint in the [task contract](execution.md#task-and-execution-contracts).
2. **Orient:** inspect the existing calculator, tests, repository instructions,
   and working-tree changes. Clarify the exception type if it is not established
   by the API contract.
3. **Plan:** add a negative-input regression test, add the smallest guard, and
   run the relevant checks. Leave unrelated pricing behavior untouched.
4. **Act:** edit only the two authorized files in an isolated checkout. A
   hypothetical Python implementation might contain this guard:

   ```python
   def total_price(unit_price, quantity):
       if quantity < 0:
           raise ValueError("quantity must be non-negative")
       return unit_price * quantity
   ```

   This assumes the application already validates numeric types and represents
   currency appropriately; it is not a complete order or money-handling API.
5. **Verify:** confirm the negative-input test fails on the baseline and passes
   on the candidate. Check zero and positive quantities, inspect the diff, and
   run applicable broader checks. If dependencies are unavailable, report that
   verification is blocked rather than reporting a verified fix.
6. **Report:** return the patch, checked revision, actual check results, and
   limitations. Publication is not authorized by this task.

If the product later adds pull-request creation, make it a separately
authorized action:

```mermaid
sequenceDiagram
    actor User
    participant Harness
    participant Model
    participant Sandbox
    participant Reviewer
    participant Git as Repository service
    User->>Harness: Request a patch with acceptance criteria
    Harness->>Model: Scoped context and permitted tools
    Model-->>Harness: Proposed edits
    Harness->>Harness: Validate permissions and limits
    Harness->>Sandbox: Apply edits and run required checks
    Sandbox-->>Harness: Candidate revision, diff, and evidence
    Harness-->>User: Patch and verification report
    opt Separately requested pull-request publication
        Harness->>Reviewer: Exact candidate and publication action
        alt Approved
            Reviewer-->>Harness: Scoped, expiring approval
            Harness->>Harness: Revalidate authorization and revision
            Harness->>Git: Publish approved candidate if still valid
            Git-->>Harness: Confirmed result
            Harness-->>User: Publication result
        else Rejected or expired
            Harness-->>User: Patch retained; nothing published
        end
    end
```

## Project and product examples

### Products you could build

| Product | Bounded agent task | Tools and context | Verification and human boundary |
| --- | --- | --- | --- |
| Repository maintenance assistant | Prepare a narrowly scoped bug-fix patch | Source search, file edits, sandboxed checks | Regression evidence and diff review; separate approval to publish |
| Customer-support copilot | Draft a response using approved help content | Tenant-scoped ticket lookup and knowledge retrieval | Check cited policy and redact personal data; human approves sending or refunds |
| Analytics assistant | Answer a business question from approved datasets | Catalog lookup and read-only, resource-limited queries | Validate aggregates and freshness; restrict sensitive rows and exports |
| Incident triage assistant | Summarize symptoms and propose a runbook step | Redacted logs, metrics, and read-only service status | Link claims to telemetry; operator authorizes production mutations |

Read-only does not mean risk-free: retrieval can expose private data and queries
can exhaust resources. Apply authorization, data minimization, and resource
limits even when no writes are allowed.

### Existing projects to study

These examples illustrate different parts of a harness, not interchangeable
solutions or dependencies of this repository. Consult their official
documentation for current behavior and configuration.

| Project or product | Relevant capabilities | Design lesson and boundary |
| --- | --- | --- |
| [LangGraph](https://docs.langchain.com/oss/python/langgraph/overview) | Stateful orchestration, durable execution, and human-in-the-loop workflows | Resumable approval checkpoints need configured persistence and thread identity; orchestration is not an authorization system. |
| [DeepSeek Harness](deepseek.md) | Plugin-based agent loop, tools, runtime profiles, and session persistence | An unaudited developer preview; plugin composition and approval prompts are not substitutes for independent isolation. |
| [OpenAI Agents SDK](https://openai.github.io/openai-agents-python/) | Agent/tool orchestration, handoffs, guardrails, and tracing | Agent-level input/output guardrails do not inspect every intermediate tool call; enforce authorization at tool execution boundaries. |
| [OpenHands](https://docs.openhands.dev/sdk) | Software-development tools, Docker workspaces, and configurable action confirmation | Runtime isolation and approval policies are separate controls; confirmation behavior depends on configuration. |
| [GitHub Copilot cloud agent](https://docs.github.com/en/copilot/concepts/agents/cloud-agent/about-cloud-agent) | Repository-oriented work in an ephemeral environment, test/lint execution, and reviewable changes | A reviewable pull request is an artifact, not proof of correctness; humans still review changes and verification evidence. |

For details behind these boundaries, see LangGraph's
[interrupts](https://docs.langchain.com/oss/python/langgraph/interrupts),
the Agents SDK's [guardrails](https://openai.github.io/openai-agents-python/guardrails/),
OpenHands' [security guide](https://docs.openhands.dev/sdk/guides/security),
and GitHub's [responsible-use guidance](https://docs.github.com/en/copilot/responsible-use/agents).

Choose based on the missing layer: orchestration for resumable workflows,
an SDK for tool-driven application logic, or a coding-oriented environment for
repository tasks. In every case, decide which controls the product supplies
and which remain your application's responsibility.

For deeper comparisons, see the [implementation chapters](../README.md#implementation-chapters).
