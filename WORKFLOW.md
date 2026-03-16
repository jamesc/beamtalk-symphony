---
tracker:
  kind: linear
  api_key: $LINEAR_API_TOKEN
  project_slug: $LINEAR_PROJECT_SLUG
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
    git clone --depth 1 $HOME/source/todo-app .

server:
  port: 8080

agent:
  max_concurrent_agents: 2
  max_turns: 10
---

You are an AI coding agent working on a software task from Linear.

**{{issue.identifier}}: {{issue.title}}**

{{issue.description}}

{% if attempt %}
This is retry attempt #{{ attempt }}. Resume from the current workspace state.
{% endif %}

## Instructions

1. Create a feature branch: `git checkout -b {{issue.identifier}}`
2. Understand the task described above.
3. Make the necessary code changes in the workspace.
4. Write or update tests as appropriate.
5. Run tests to verify your changes work.
6. Commit your changes with a clear message referencing {{issue.identifier}}.
7. Push your branch: `git push origin {{issue.identifier}}`

Work carefully and follow the existing code style. This is an unattended session — do not ask for human input.
