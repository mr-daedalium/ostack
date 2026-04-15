# PROJECT.md
# Agent Companies Standard — Project definitions and workspace bindings.
# This file maps logical project names to their workspace directories and
# describes project-level configuration for agent routing and task scoping.

projects:
  - name: default
    description: Primary project workspace
    workspace: .
    agents:
      - example-agent
    skills:
      - ostack

workspaces:
  - name: root
    path: .
    project: default
