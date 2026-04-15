# AGENTS.md
# Agent Companies Standard — Agent definition template.
# This file describes an agent's identity, capabilities, allowed tools,
# and behavioral guidelines for use within the Agent Companies framework.

name: example-agent
version: v0.1.0
description: |
  A generic agent template demonstrating the Agent Companies agent definition format.
  Replace this description with the agent's specific purpose and capabilities.

allowed-tools:
  - Read
  - Bash
  - AskUserQuestion

behaviors:
  - Always read relevant files before modifying them
  - Ask before taking irreversible actions
  - Prefer the simplest approach that satisfies the task

scope:
  projects:
    - default
  workspaces:
    - root
