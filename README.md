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

## Prompt templates

The prompt in `WORKFLOW.md` is a [Liquid](https://shopify.github.io/liquid/) template rendered by the [`liquid`](https://github.com/jamesc/beamtalk-liquid) package. Two variables are available: `issue` (the Linear issue's fields, e.g. `{{ issue.identifier }}`, `{{ issue.labels }}`) and `attempt` (`0` on the first dispatch, `1`+ on retries, `nil` if unknown). See the package README for the supported tags, filters and divergences from Liquid.

### Migrating an existing WORKFLOW.md (BT-4918)

The old renderer rewrote substrings; the new one parses the template. Behaviour changes:

- **Truthiness:** only `nil` and `false` are falsy. `0` and `""` are now truthy, so `{% if attempt %}` is true on the first dispatch (`attempt` is `0`). Write `{% if attempt > 0 %}` instead.
- **Strict variables:** a variable that does not exist (`{{ issue.titel }}`) is now an error that fails the dispatch, instead of rendering as empty text. A field that exists but is `nil` (e.g. an issue with no description) still renders empty.
- **Parse errors:** unclosed or unknown tags, unknown filters and a bare `|` are errors instead of passing through silently.
- **Limits:** templates nest at most 100 levels deep (error kind `nesting`), and a single `{{ }}` or `{% %}` may hold at most 1000 tokens (`limit`); exceeding either is a parse error.
- **Values are literal:** issue text containing `{{ ... }}` or `{% ... %}` is emitted as-is and never expanded.
- **New:** `if`/`for` blocks nest; `elsif`, `else`, `unless`, comparisons (`== != < > <= >= contains and or`), filters (`{{ issue.title | upcase }}`), `comment`/`raw` and `{%-`/`-%}` whitespace control are supported.

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
