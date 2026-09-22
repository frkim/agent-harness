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
| [Examples](docs/examples.md) | Repository maintenance walkthrough, product ideas, and dedicated framework/coding-harness implementation coverage |
| [Operations](docs/operations.md) | Progress reports, observability, budgets, rollout, and implementation checklist |

## Implementation chapters

These chapters map the design guide to specific tools and platforms. They are
not dependencies or runnable integrations of this repository. Product behavior
varies by version and deployment; consult the official sources in each chapter.
The implementation chapters were reviewed against official online sources on
**2026-09-22**; default-branch documentation may be ahead of installed releases.

| Implementation | Layer covered |
| --- | --- |
| [Microsoft: Foundry and Agent Framework](docs/microsoft.md) | Managed agent service and code-first orchestration framework |
| [Claude: Claude Code and Agent SDK](docs/claude.md) | Coding harness and programmable agent runtime |
| [OpenAI Codex](docs/codex.md) | Coding agent, CLI/SDK automation, and App Server integration |
| [DeepSeek Harness](docs/deepseek.md) | Plugin-based agent runtime (developer preview), distinct from the model API |
| [LangGraph](docs/langgraph.md) | Stateful workflow orchestration, persistence, and human checkpoints |

Dedicated implementation sections in the examples chapter extend these five
chapters; a mention in a project list is not a substitute for implementation
coverage.

| Application-agent framework | Layer covered |
| --- | --- |
| [LangChain Deep Agents](docs/examples.md#langchain-deep-agents) | Packaged harness built on LangGraph, not the same abstraction as LangGraph |
| [Google ADK](docs/examples.md#google-adk) | Tool-using agents, multi-agent workflows, sessions, and artifacts |
| [OpenAI Agents SDK](docs/examples.md#openai-agents-sdk) | Application-agent orchestration, separate from Codex |
| [Strands Agents](docs/examples.md#strands-agents) | Model-driven tool loop and multi-agent patterns |
| [Pydantic AI / Pydantic AI Harness](docs/examples.md#pydantic-ai-and-pydantic-ai-harness) | Typed agent foundation versus a fuller harness |
| [CrewAI](docs/examples.md#crewai) | Role-based crews and explicitly controlled flows |

| Coding runtime / toolkit | Layer covered |
| --- | --- |
| [OpenHands Software Agent SDK](docs/examples.md#openhands-software-agent-sdk) | Programmable software-development agents and execution workspaces |
| [Gemini CLI](docs/examples.md#gemini-cli) | Coding CLI, headless operation, hooks, and sandbox configuration |
| [OpenCode](docs/examples.md#opencode) | Multi-provider coding runtime with client/server SDK integration |
| [Pi](docs/examples.md#pi) | Composable agent toolkit and coding CLI |

Managed hosting remains a separate choice: see
[Foundry Agent Service](docs/microsoft.md), rather than assuming an SDK includes
a hosted execution environment.

## Harness evaluation

The [harness evaluation page](docs/harness-evaluation.md) groups coding runtimes,
application-agent frameworks, and managed hosting services separately. It
includes a weighted rubric, editorial scores, implementation trade-offs,
verification exercises, and needs-based selection guidance. Scores are dated
editorial assessments, not benchmark results or a cross-layer leaderboard.
Read it directly on GitHub as a rendered Markdown page.

## Presentation

A 45-minute [Marp](https://marp.app/) deck walks through the whole guide:
[`docs/presentations/agent-harness-45min.md`](docs/presentations/agent-harness-45min.md).
The [presentations README](docs/presentations/README.md) explains how to
preview and export it locally. Pushes to `main` publish the rendered HTML and
PDF to GitHub Pages at <https://frkim.github.io/agent-harness/>.

## Contributing

Keep documentation practical, implementation-agnostic where possible, and
grounded in observable behavior. When proposing a pattern, describe its
boundary conditions and how its outcome can be verified.

This project is available under the [MIT License](LICENSE).
