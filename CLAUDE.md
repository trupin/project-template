# CLAUDE.md

This file provides guidance to Claude Code when working with this repository. It serves as the **orchestrator agent guide** for a multi-agent workflow.

## First-Run Detection

**Check for first run by reading `.claude/project.configured`. If the file does not exist (read errors or is missing), this project has not been set up yet.**

On first run, before doing anything else:
1. Tell the user: "This project hasn't been configured yet. Let me walk you through setup."
2. Run the `/setup` skill to begin the interactive onboarding wizard.
3. Do NOT proceed with any implementation work until setup is complete.

After setup completes, all `{{PLACEHOLDER}}` sections below will be filled in and `.claude/project.configured` will exist.

---

## Project Overview

{{PROJECT_NAME}} — {{PROJECT_DESCRIPTION}}

**Tech Stack:** {{TECH_STACK}}
**Package Manager:** {{PACKAGE_MANAGER}}

**If `specs.md` exists, it is the source of truth for all behavior** — read the relevant section before implementing any feature. If the spec is ambiguous, clarify before coding — don't guess. If no `specs.md` exists, the issue files and this CLAUDE.md serve as the specification.

## Repository Structure

{{REPO_STRUCTURE}}

**Dependency direction: {{DEPENDENCY_DIRECTION}}. Never import upstream.**

## Multi-Agent Workflow

This project uses a multi-domain architecture with an orchestrator pattern.

### Agents

| Agent | Domain | Description |
|-------|--------|-------------|
| **Orchestrator** (you) | root | Reads the plan, finds ready issues, spawns domain agents, tracks progress |
{{AGENT_TABLE}}

Agent definitions live in `.claude/agents/`. Each agent file has YAML frontmatter (`name`, `description`) and markdown instructions covering responsibilities, workflow, escalation rules, and code organization.

### Optional Specialized Agents

These agents are activated during `/setup` for projects that benefit from them. They are NOT domain agents — they serve cross-cutting concerns.

| Agent | Purpose | When to Use |
|-------|---------|-------------|
| **spec-writer** | Converts vague prompts into structured behavioral specs | Greenfield projects, "build me an app" scenarios |
| **evaluator** | Tests the running app like a skeptical user (never reads source code) | Any project with user-facing behavior |
| **sprint-planner** | Defines testable "done" criteria before each implementation batch | Projects with 5+ issues, pairs with evaluator |
| **context-manager** | Creates handoff artifacts for fresh-context restarts | Multi-session projects, 10+ issues |

If activated, these agents add steps to the orchestration loop (see below).

### Orchestration Loop

1. **Restore context** *(if context-manager active)*: Check `.claude/handoffs/` for recent handoff files. If found, spawn context-manager to restore session state.
2. **Read `issues/PLAN.md`** to understand current state.
3. **Find ready issues**: Scan the phase table for issues with status `todo` whose dependencies are all `done`.
4. **Group by domain**: Collect ready issues by domain.
5. **Sprint contract** *(if evaluator active + batch > 1)*: Spawn sprint-planner to produce a sprint contract defining testable "done" criteria for this batch.
6. **Handle shared issues** (SHARED-*) yourself — they produce artifacts all domains consume.
7. **Spawn domain agents**: Use the Agent tool to start the appropriate domain agent for each group. Pass the sprint contract path (if one exists) alongside issue files. Domain agents run in parallel when multiple have ready work.
8. **Verify completion**: When a domain agent reports done, verify by running the appropriate check skills (`/test`, `/lint`).
9. **Evaluate** *(if evaluator active)*: Spawn the evaluator agent to test the running application against specs and sprint contract. If FAIL, send the eval verdict back to the domain agent. Loop up to 3 times. If still failing, escalate to user.
10. **Audit** *(for qualifying issues)*: Run `/audit` for P0 issues, cross-domain changes, large changes, and security-sensitive code.
11. **Mark done**: Update the issue's Status field to `done` in both the issue file and `issues/PLAN.md`.
12. **Checkpoint** *(if context-manager active)*: Every 3-5 completed issues, spawn context-manager to create a handoff artifact.
13. **Repeat** until all issues are done or no more issues are ready.

### When to Spawn Which Agent

{{AGENT_DISPATCH_RULES}}

### Maximizing Parallelism

**Actively look for opportunities to split work across more agents running in parallel.** The domain agents are a starting point, not a ceiling.

- **Split within a domain**: If a domain has multiple independent issues ready, spawn separate agents for each — one per issue — rather than one agent doing them sequentially. Use `isolation: "worktree"` when agents touch overlapping files.
- **Split within an issue**: If an issue has clearly independent sub-tasks, consider splitting them across agents that each handle a subset, then merge.
- **Define new agents when needed**: If you notice a body of work that doesn't fit existing agents, create a new agent definition in `.claude/agents/` and use it.
- **Run verification in parallel**: Lint, test, and type-check can often run concurrently.
- **Background non-blocking work**: Use `run_in_background` for agents whose results you don't need immediately.

The goal is to minimize wall-clock time by keeping as many agents busy as possible. Sequential execution should be the exception (when there's a real data dependency), not the default.

### Escalation Protocol

1. **Domain agent handles**: Implementation, testing, basic refactoring, lint/type fixes within its domain.
2. **Escalate to orchestrator**: Cross-domain issues, ambiguous requirements, dependency conflicts, shared contract changes.
3. **Escalate to user**: Architecture decisions not covered by specs, unclear acceptance criteria, issues that require design judgment beyond the spec.

When a subagent reports a problem it cannot resolve, evaluate whether the problem is:
- **Within another domain** — spawn the other domain agent to address it.
- **A spec gap** — check `specs.md` for clarification. If still unclear, ask the user.
- **A design decision** — ask the user.

## Issue-First Workflow

**Always create issue files before implementing.** When planning a new feature, fixing a bug, or doing any non-trivial work:

1. **Create an issue file** in the appropriate `issues/<domain>/` directory using `issues/TEMPLATE.md` as the format.
2. **Add the issue to `issues/PLAN.md`** in the appropriate phase table.
3. **Then implement** — domain agents read the issue file for context.

This applies even when the user describes the work inline. Capture it as a structured issue first so it's tracked, discoverable, and provides context for domain agents.

## SDLC (Software Development Lifecycle)

Every issue follows this cycle, enforced by domain agents and the orchestrator:

1. **Implement** — Write code per the issue's Technical Design and Acceptance Criteria. Read the sprint contract (if one exists in `issues/sprints/`) to understand what the evaluator will verify.
2. **Test** — Run tests per the issue's Testing Strategy. Write new tests for new code.
3. **Check** — Run linters and type checkers (`/lint`).
4. **Audit** — Self-audit: check for spec compliance, missing tests, code quality issues.
5. **Refactor** — Fix any issues found in review. Re-run tests to confirm no regressions.
6. **Evaluate** *(if evaluator active)* — Orchestrator runs the evaluator agent against the running application. If FAIL, domain agent receives the eval verdict and fixes. Loop up to 3 times.
7. **Surface** — If something cannot be resolved, escalate (see Escalation Protocol).
8. **Report** — Mark issue as done and report to orchestrator.

### Definition of Done

An issue is done when:
- All acceptance criteria are met.
- Tests pass with no regressions.
- Type checks pass.
- Lint passes.
- Evaluator verdict is PASS (if evaluator is active).
- Code follows project conventions.

## Available Skills

| Skill | Purpose |
|-------|---------|
| `/setup` | Run the project onboarding wizard (first-run only) |
| `/dashboard` | Quick project dashboard: issues, git state, what's actionable now |
| `/decompose` | Decompose a feature into phased issues across domains |
| `/implement` | Pick ready issues from plan and implement via domain agents |
| `/issue` | Manage issues: create, close, implement, plan, refine, list, show |
| `/evaluate` | Run the behavioral evaluator against completed issues |
| `/audit` | Audit recent changes for defects, missing tests, spec drift |
| `/lint` | Run all linters and formatters |
| `/test` | Run the test suite with optional filters |
| `/pr` | Create a pull request from the current branch |

## Git Workflow

**Only the orchestrator agent (you) creates commits.** Domain agents write code and run tests but never commit or push. This ensures atomic, well-described commits and prevents conflicts between parallel agents.

### Rules

1. **All work happens on `main`** unless the user asks for a branch.
2. **Commit before starting new work** — never start a new task with uncommitted changes from a previous task.
3. **Every commit must have an issue number** — commit messages must start with `[ISSUE-ID]`.
4. **Commit after verification** — only commit once the domain agent reports success AND you have verified via check skills.
5. **One commit per issue** (or per logical batch of related issues). Never mix unrelated changes.
6. **Commit message format**:
   ```
   [ISSUE-ID] Short imperative description

   - Key change 1
   - Key change 2
   ```
7. **Stage specifically** — use `git add <path>` for the files relevant to the issue. Never `git add -A` or `git add .` blindly.
8. **Never force push, amend published commits, or reset --hard** unless the user explicitly asks.
9. **Never skip hooks** (`--no-verify`, `--no-gpg-sign`).

### When to Commit

- **After each completed issue**: Once domain agent reports done, you verify, and checks pass — commit all files for that issue.
- **After a batch of related issues**: If multiple issues in the same phase complete together, you may commit them together if they form a logical unit.
- **After spec/plan updates**: Documentation-only changes (specs.md edits, new issue files, plan updates) can be committed separately.
- **Never commit broken code** — if checks fail, fix first.

### What Domain Agents Must NOT Do

Domain agents must never run `git commit`, `git push`, `git checkout`, `git reset`, `git stash`, or any other state-changing git command. They only write files and run tests.

## Build & Dev Commands

{{BUILD_COMMANDS}}

## Testing Conventions

{{TESTING_CONVENTIONS}}

## Lint Discipline

**Never fix lint warnings by disabling rules.** Always fix the underlying code first. Only add an inline suppression comment as a last resort when no code fix is possible — and include a comment explaining why.

## Code Organization

**Colocate code by component/feature, not by class type.** Multiple agents work in parallel on different features. If a feature's code is spread across many directories, agents working on different features will conflict on the same files.

- Group all code for a component (implementation, types, helpers, tests) in the same directory.
- Ask: "Can two agents work on two different features simultaneously without touching the same files?" If not, restructure.
- Keep related types and constants next to the code that uses them, not in shared utility files.

## Code Style

{{CODE_STYLE}}
