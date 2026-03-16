---
tracker:
  kind: linear
  api_key: $LINEAR_API_TOKEN
  project_slug: c3c0717a652a
  active_states:
    - Todo
    - In Progress
  terminal_states:
    - Done
    - Canceled
    - Cancelled

codex:
  command: codex app-server
  approval_policy: auto-edit
  thread_sandbox: none

workspace:
  root: /tmp/symphony_workspaces

hooks:
  after_create: |
    cp -r /home/james/source/todo-app/. .
    git init || true
    git add -A && git commit -m "Initial scaffold" || true

server:
  port: 8080

agent:
  max_concurrent_agents: 2
  max_turns: 10
---

You are an AI coding agent working on a software task from Linear.

**{{issue.identifier}}: {{issue.title}}**

{{issue.description}}

## Instructions

1. Understand the task described above.
2. Make the necessary code changes in the workspace.
3. Write or update tests as appropriate.
4. Commit your changes with a clear message referencing the issue identifier.

Work carefully and follow the existing code style.
