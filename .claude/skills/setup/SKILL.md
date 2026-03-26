---
description: "Interactive project onboarding wizard. Walks through tech stack, architecture, domains, agents, and skills to configure the full Claude multi-agent setup."
user_invocable: true
---

Run the project onboarding wizard. This is a step-by-step interactive process to configure the multi-agent development environment.

**Goal:** By the end of this wizard, CLAUDE.md will be fully configured, domain agents will be generated, skills will be tailored to the tech stack, and the issue tracking system will be ready.

---

## Phase 0: Pre-Check

Before starting, check for conflicts:

1. If `.claude/` already contains files not from this template (e.g., an existing `settings.local.json` with permissions, custom skills, or agent definitions), **warn the user** and ask how to proceed:
   - "I found existing Claude configuration files. Should I merge with them, or start fresh?"
   - If merging, preserve existing permissions and skills, only adding new ones.
2. If a `CLAUDE.md` already exists at the project root that is NOT the template version (no `{{PLACEHOLDER}}` markers), warn: "There's already a configured CLAUDE.md. Running setup will overwrite it."

---

## Phase 1: Project Identity

Ask the user the following questions **one group at a time** (don't dump all questions at once). Wait for answers before proceeding.

**Questions:**
1. What is the project name?
2. In one sentence, what does this project do?
3. Is there existing code already, or are we starting from scratch?

If there's existing code, use the Explore agent to scan the repository structure, detect the tech stack, and pre-fill as many answers as possible for the remaining phases. Present your findings to the user for confirmation before proceeding.

---

## Phase 2: Tech Stack & Tooling

**Questions:**
1. What language(s) and version(s)? (e.g., Python 3.12, TypeScript 5.x, Rust, Go)
2. What frameworks? (e.g., FastAPI, React, Next.js, Axum, none)
3. What package manager? (e.g., uv, npm, pnpm, cargo, go mod)
4. What linter(s) and formatter(s)? (e.g., ruff+pyright, eslint+prettier, clippy)
5. What testing framework? (e.g., pytest, vitest, jest, cargo test)
6. Any build tools or task runners? (e.g., make, just, turbo, nx)

**Based on answers, generate:**
- The `Build & Dev Commands` section for CLAUDE.md
- The `Code Style` section for CLAUDE.md
- The `/lint` skill (with actual commands)
- The `/test` skill (with actual commands)

---

## Phase 3: Architecture & Domains

This is the most important phase — it determines the agent structure.

**Questions:**
1. What are the major modules/domains of this project? (e.g., "parser, database, API server, frontend", or "core library, CLI, web UI")
2. What is the dependency direction between them? (e.g., "parser <- database <- API <- frontend")
3. Are there any shared contracts or interfaces between domains? (e.g., shared types, API schemas, protobuf definitions)
4. For each domain: what directory does its source live in, and where are its tests? (e.g., "src/api/ with tests in tests/api/", or "colocated __tests__/ directories")

**Help the user think through this:**
- If the user isn't sure, suggest a decomposition based on the tech stack and project description.
- Propose a dependency graph and ask the user to validate or adjust.
- Identify which domains can be worked on in parallel (no dependency between them).
- Ask: "Is there a frontend and backend? Are they in the same repo?"

**Based on answers:**
- Fill in the `Repository Structure` section in CLAUDE.md
- Fill in the `Dependency direction` line
- Determine how many domain agents we need

---

## Phase 4: Agent Generation

For each domain identified in Phase 3, generate an agent file at `.claude/agents/<domain>-dev.md`.

**Use `.claude/AGENT_TEMPLATE.md` as the base.** For each domain, copy the template and substitute all `{{PLACEHOLDER}}` values:
- `{{DOMAIN}}` — lowercase domain name (e.g., `api`)
- `{{DOMAIN_PREFIX}}` — uppercase issue prefix (e.g., `API`)
- `{{DOMAIN_DESCRIPTION}}` — human-readable name (e.g., "REST API server")
- `{{DOMAIN_DIRECTORY}}` — source directory (e.g., `src/api/`)
- `{{TEST_DIRECTORY}}` — test directory (e.g., `tests/api/`)
- `{{PROJECT_NAME}}` — from Phase 1
- `{{LINT_COMMAND}}`, `{{TEST_COMMAND}}`, `{{TYPECHECK_COMMAND}}` — from Phase 2

**After generating agents:**
- Fill in the `Agents` table in CLAUDE.md
- Fill in the `Agent Dispatch Rules` section
- Create `issues/<domain>/` directories for each domain

Present the generated agent list to the user for review. Ask:
- "Does this agent breakdown look right?"
- "Should any domain be split further or merged?"

### Specialized Agents

After the domain agents are confirmed, offer the optional specialized agents:

```
In addition to your domain agents, this template includes optional specialized agents
for long-running projects:

1. **Spec Writer** — Converts vague requirements ("build me an app that does X") into
   structured behavioral specs. Recommended for: greenfield projects, major new features.

2. **Evaluator** — Tests the running application like a skeptical real user. Never reads
   source code. Catches behavioral bugs that unit tests miss.
   Recommended for: any project with user-facing behavior (web, CLI, API).

3. **Sprint Planner** — Before each implementation batch, defines exactly what "done"
   looks like in testable terms. Pairs with the evaluator.
   Recommended for: projects with 5+ issues.

4. **Context Manager** — Creates handoff documents for fresh-context restarts. Prevents
   information loss across long or multi-session work.
   Recommended for: multi-session projects, 10+ issues.

Which of these would you like to activate? (e.g., "all", "1,2", or "none")
```

For each activated agent:
1. The agent file already exists at `.claude/agents/<name>.md` (pre-configured). No generation needed — just verify it's present.
2. Add the agent to the Agents table in CLAUDE.md (in the "Optional Specialized Agents" section).
3. If **evaluator** is activated:
   - Create `issues/evals/` directory.
   - The `/evaluate` skill already exists at `.claude/skills/evaluate/SKILL.md`.
4. If **sprint-planner** is activated:
   - Create `issues/sprints/` directory.
5. If **context-manager** is activated:
   - Create `.claude/handoffs/` directory.
6. Update the Orchestration Loop in CLAUDE.md to note which optional steps are active (the loop already has conditional steps marked with "if X active").

---

## Phase 5: Issue Domains & Plan Setup

**Based on the domains from Phase 4:**

1. Create `issues/<domain>/` directory for each domain, plus `issues/shared/` for cross-domain issues.
2. Generate the initial `issues/PLAN.md` with:
   - A Phase 0 section for project setup tasks
   - Empty phase tables ready for future issues
   - The correct domain prefixes (e.g., `API-001`, `UI-001`, `CORE-001`)

**Ask the user:**
- "Do you already have a rough plan or list of features to build? I can file them as issues now."
- If yes, run through the issue creation flow for each one (using the `/issue create` logic).

---

## Phase 6: Specs Foundation

Ask the user:
1. "Do you have existing requirements, PRDs, or design docs I should reference?"
2. "Would you like me to create a `specs.md` skeleton based on the domains we identified?"

If yes to specs.md:
- Create a `specs.md` with section headings for each domain
- Add a "Shared Contracts" section for cross-domain interfaces
- Leave sections as stubs with `[TBD]` markers for the user to fill in

---

## Phase 7: Skills Refinement

Present the current skills to the user and ask if they want to customize or add any:

**Current skills:**
- `/dashboard` — Quick project dashboard: issues, git state, what's actionable
- `/decompose` — Decompose a feature into phased issues across domains
- `/implement` — Orchestrate issue implementation via domain agents
- `/issue` — Create, close, list, refine, show issues
- `/evaluate` — Run behavioral evaluator against completed issues (if evaluator active)
- `/audit` — Audit code for defects, missing tests, spec drift
- `/lint` — Run linters (configured in Phase 2)
- `/test` — Run tests (configured in Phase 2)
- `/pr` — Create a pull request from the current branch

**Ask:**
- "Are there any project-specific workflows you'd like as skills? For example: `/deploy`, `/migrate`, `/seed`, `/check`, `/build`, `/e2e`"
- For each requested skill, ask what it should do, then generate a SKILL.md file.

---

## Phase 8: Finalize

1. **Fill in all remaining `{{PLACEHOLDER}}` sections** in CLAUDE.md with the collected information.
2. **Rewrite `/lint` and `/test` skills entirely** — replace the full file content with the actual commands from Phase 2. Remove the fallback detection logic; it's no longer needed after configuration.
3. **Configure permissions** in `.claude/settings.local.json` based on the tooling from Phase 2. Suggest permissions for common tool invocations (e.g., `Bash(uv run:*)`, `Bash(npm:*)`, `Bash(cargo:*)`). Present the list to the user for approval before writing.
4. **Write `.claude/project.configured`** with the current date to mark setup as complete:
   ```
   configured: YYYY-MM-DD
   project: {{PROJECT_NAME}}
   ```
5. **Present a summary** to the user:
   - Project: name and description
   - Domains: list with agent names
   - Skills: list of available skills
   - Issue prefixes: e.g., `CORE-001`, `API-001`, `UI-001`, `SHARED-001`
   - Next steps: "You can now use `/issue create <description>` to file your first issue, or `/implement` to start working."

6. **Ask:** "Would you like to start filing issues for your initial features now?"

---

## Rules

- Ask questions one group at a time. Don't overwhelm the user with everything at once.
- If existing code is detected, pre-fill as much as possible and ask for confirmation rather than making the user type everything.
- Always show the user what you're about to generate and get approval before writing files.
- If the user seems unsure about architecture or domains, offer concrete suggestions based on the tech stack and project type.
- Keep the generated CLAUDE.md concise. Don't add flowstate-specific content — only what's relevant to this project.
- The onboarding should feel like a conversation, not a form.
