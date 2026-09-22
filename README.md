# Agent Harness

An agent harness is the control plane around an AI model. It gives the model a
well-defined job, access to bounded capabilities, and a repeatable loop for
turning requests into verified outcomes.

This repository documents the concepts and practices for designing one. It is
a design guide, not a runnable framework: there is no SDK, application, or
installation step here. The examples in the guide are illustrative contracts and
pseudocode, not APIs implemented by this repository.

The guide is for engineers building agents and technical leaders deciding how
much autonomy a product can safely support. Start with the
[core model](docs/foundations.md#core-model), then use the
[worked example](docs/examples.md#worked-example-a-repository-maintenance-agent)
and [implementation checklist](docs/operations.md#implementation-checklist) to
scope a first release.

## Design guide

The original guide is organized into focused chapters in [`docs/`](docs/).
Read them in order, or jump to the topic you need:

| Chapter | Topics |
| --- | --- |
| [Foundations](docs/foundations.md) | Why use a harness, core model, reference architecture, lifecycle, and recovery |
| [Execution](docs/execution.md) | Task contracts, bounded controller, tool design, and context management |
| [Safety and verification](docs/safety-and-verification.md) | Trust boundaries, approvals, threat exercises, and independent checks |
| [Examples](docs/examples.md) | Repository maintenance walkthrough, product ideas, and projects to study |
| [Operations](docs/operations.md) | Progress reports, observability, budgets, rollout, and implementation checklist |

## Implementation chapters

These chapters map the design guide to specific tools and platforms. They are
not dependencies or runnable integrations of this repository. Product behavior
varies by version and deployment; consult the official sources in each chapter.

| Implementation | Layer covered |
| --- | --- |
| [Microsoft: Foundry and Agent Framework](docs/microsoft.md) | Managed agent service and code-first orchestration framework |
| [Claude: Claude Code and Agent SDK](docs/claude.md) | Coding harness and programmable agent runtime |
| [OpenAI Codex](docs/codex.md) | Coding agent, CLI automation, and SDK integration |
| [DeepSeek](docs/deepseek.md) | Model API and tool-calling loop for a harness you supply |
| [LangGraph](docs/langgraph.md) | Stateful workflow orchestration, persistence, and human checkpoints |

## Contributing

Keep documentation practical, implementation-agnostic where possible, and
grounded in observable behavior. When proposing a pattern, describe its
boundary conditions and how its outcome can be verified.

This project is available under the [MIT License](LICENSE).
