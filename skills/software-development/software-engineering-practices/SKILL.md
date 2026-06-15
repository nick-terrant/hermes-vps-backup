---
name: software-engineering-practices
description: "Software engineering workflows: planning, debugging, TDD, subagent-driven development, kanban orchestration, spike experiments."
version: 1.0.0
author: Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [engineering, planning, debugging, TDD, testing, development, kanban, subagent, spike, workflow]
    related_skills: [writing-plans, systematic-debugging, test-driven-development, subagent-driven-development, kanban-orchestrator, spike]
---

# Software Engineering Practices

Umbrella for software engineering workflows and methodologies. Each section below covers a distinct practice — load the relevant reference for the task at hand.

## Practices

### 1. Writing Implementation Plans

Structured approach to breaking down complex tasks into bite-sized, executable steps. Use when the user asks for a plan, or when a task is too large to tackle in one pass.

See `references/writing-plans.md` for the full methodology — task decomposition, dependency ordering, bite-sized task format, and plan templates.

### 2. Systematic Debugging

4-phase root cause debugging: understand the bug before fixing it. Use when encountering unexpected behavior, test failures, or production issues.

See `references/systematic-debugging.md` for the full methodology — reproduce, hypothesize, isolate, fix, and verify.

### 3. Test-Driven Development

RED-GREEN-REFACTOR cycle: write tests before code. Use when the user asks for TDD or when building new features that benefit from test-first development.

See `references/test-driven-development.md` for the full methodology — test patterns, refactoring steps, and when TDD is/isn't appropriate.

### 4. Subagent-Driven Development

Execute implementation plans via `delegate_task` subagents with a 2-stage review pattern (worker → reviewer). Use for parallelizable development work.

See `references/subagent-driven-development.md` and its sub-references (`references/subagent-context-budget.md`, `references/subagent-gates-taxonomy.md`).

### 5. Kanban Orchestration

Decomposition playbook and anti-temptation rules for orchestrating work across multiple agent profiles via the Hermes kanban system.

See `references/kanban-orchestrator.md` for the full playbook.

### 6. Spike Experiments

Throwaway experiments to validate an idea before committing to a build. Use when exploring unfamiliar technologies, testing feasibility, or prototyping.

See `references/spike.md` for the spike methodology.

## When to Use Which Practice

| Situation | Practice |
|-----------|----------|
| Large task, need to break it down | Writing Plans (#1) |
| Bug or unexpected behavior | Systematic Debugging (#2) |
| Building new feature with tests | TDD (#3) |
| Parallelizable dev work | Subagent-Driven Dev (#4) |
| Multi-agent coordination | Kanban Orchestration (#5) |
| Exploring unfamiliar territory | Spike (#6) |
