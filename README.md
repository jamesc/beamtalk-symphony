# Symphony

An AI agent orchestrator written in [Beamtalk](https://beamtalk.dev). Symphony polls Linear for issues, dispatches coding agents via [Codex](https://openai.com/codex), and manages the full lifecycle: workspace creation, multi-turn sessions, approval handling, and cleanup.

Inspired by [openai/symphony](https://github.com/openai/symphony).

## How it works

1. **Orchestrator** polls a Linear project for issues in active states (Todo, In Progress)
2. For each eligible issue, an **AgentRunner** creates an isolated workspace (git clone) and starts a **CodexClient** subprocess
3. The codex agent receives the issue description as a prompt, writes code, runs tests, commits, and pushes to a feature branch
4. **HttpServer** provides a dashboard at `http://localhost:8080` for observability

## Prerequisites

- [Beamtalk](https://beamtalk.dev) runtime
- [Codex CLI](https://openai.com/codex) (`codex app-server`)
- A Linear project with issues to work on
- A target repository for the agents to work in

## Setup

1. Create a `.env` file:
```
LINEAR_API_TOKEN=lin_api_...
LINEAR_PROJECT_SLUG=your-project-slug
```

2. Configure `WORKFLOW.md` with your project settings — tracker config, workspace root, hooks, agent prompt, and codex command.

3. Build and run:
```bash
beamtalk build
beamtalk run .
```

Or use the REPL for development:
```bash
beamtalk repl
> Main run
```

## Architecture

| Component | Role |
|-----------|------|
| `SymphonySupervisor` | OTP supervision tree |
| `Orchestrator` | Poll loop, dispatch, reconciliation, retry queue |
| `AgentRunner` | Per-issue worker lifecycle |
| `CodexClient` | Codex app-server subprocess over stdio (JSON-RPC) |
| `LinearClient` | GraphQL client for Linear API |
| `WorkspaceManager` | Per-issue workspace directories with hooks |
| `HttpServer` | Dashboard and JSON API |
| `Config` | Typed config with `$VAR` env resolution |
| `CandidateFilter` | Priority sorting and concurrency control |

## License

Apache-2.0 — see [LICENSE](LICENSE).
