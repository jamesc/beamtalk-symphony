---
tracker:
  kind: linear
  api_key: $LINEAR_API_TOKEN
  project_slug: c3c0717a652a
  active_states: "Todo"
  terminal_states: "Done, Canceled"

codex:
  command: codex
  approval_policy: auto-edit
  thread_sandbox: none

workspace:
  root: /tmp/symphony_workspaces

server:
  port: 8080
---

You are an AI coding agent working on a software task from Linear.

**{{issue.identifier}}: {{issue.title}}**

{{issue.description}}

## Instructions

1. Understand the task described above.
2. Make the necessary code changes in the workspace.
3. Write or update tests as appropriate.
4. Commit your changes with a clear message referencing the issue identifier.
5. Open a pull request for review.

Work carefully and follow the existing code style.
