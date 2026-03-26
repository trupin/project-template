# Project Template

A multi-agent orchestration template for [Claude Code](https://docs.anthropic.com/en/docs/claude-code). Clone it, run Claude Code, and it walks you through setup — then you can say "build me an app that does X" and it handles the rest.

## What this is

This is a project scaffold that turns Claude Code into a team of specialized agents. Instead of one agent doing everything in a single long session, work is decomposed into issues, assigned to domain-specific agents running in parallel, and verified by a dedicated evaluator before committing.

The architecture is inspired by Anthropic's research on [harness design for long-running applications](https://www.anthropic.com/engineering/harness-design-long-running-apps), which found that separating generation from evaluation — and giving each role to a different agent — significantly outperforms single-agent systems.

## How it works

```
You: "Build me a project management app with real-time updates"
                    |
            [spec-writer agent]
          Asks clarifying questions,
        produces behavioral specs.md
                    |
              [/decompose]
         Breaks spec into 15 issues
          across 4 domains in 3 phases
                    |
              [/implement all]
                    |
     +--------------+--------------+
     |              |              |
 [api-dev]    [ui-dev]     [core-dev]     <- domain agents (parallel)
     |              |              |
     +--------------+--------------+
                    |
            [/test + /lint]
                    |
           [evaluator agent]              <- tests the running app like a user
                    |
          PASS? -> commit
          FAIL? -> send back to domain agent (up to 3x)
                    |
           [context-manager]              <- periodic checkpoint for session continuity
                    |
                 [repeat]
```

## The agents

### Domain agents (generated during setup)

One per architectural domain (e.g., `api-dev`, `ui-dev`, `core-dev`). These are your builders — they read issue files, implement code, write tests, and report back. They never touch git. The orchestrator handles all commits.

### Specialized agents (optional, activated during setup)

These come pre-configured and address specific failure modes of long-running AI coding sessions:

| Agent | Problem it solves |
|-------|-------------------|
| **spec-writer** | Vague prompts lead to vague implementations. This agent converts "build me X" into structured behavioral specs before any code is written. Describes WHAT, never HOW — avoiding implementation-detail errors that cascade downstream. |
| **evaluator** | Agents are bad at evaluating their own work. This agent tests the running application through its public interfaces (HTTP, CLI, browser) like a skeptical QA engineer. It never reads source code — this prevents it from rationalizing bugs. |
| **sprint-planner** | Without explicit "done" criteria, generators and evaluators talk past each other. This agent produces Given/When/Then contracts before each implementation batch so everyone agrees on what success looks like. |
| **context-manager** | Long sessions degrade — models lose coherence and start wrapping up prematurely. This agent creates checkpoint documents that capture ephemeral session state, enabling clean restarts without information loss. |

## Getting started

1. Clone this repo as your project starting point:
   ```bash
   git clone https://github.com/trupin/project-template my-project
   cd my-project
   rm -rf .git && git init
   ```

2. Open Claude Code:
   ```bash
   claude
   ```

3. It detects the unconfigured template and runs the `/setup` wizard, which walks you through:
   - Project name and description
   - Tech stack and tooling
   - Architecture and domain decomposition
   - Agent generation and activation
   - Issue tracking setup

4. Start building:
   ```
   > Build me a CLI tool that converts markdown files to PDF with custom templates
   ```

   Or use the skills directly:
   ```
   > /decompose Add user authentication with OAuth and email/password
   > /implement all
   ```

## Skills

| Skill | What it does |
|-------|-------------|
| `/setup` | Interactive onboarding wizard (first-run only) |
| `/dashboard` | Quick status: issue counts, git state, what's actionable |
| `/decompose` | Break a feature into phased issues across domains |
| `/implement` | Pick ready issues and run the full orchestration loop |
| `/issue` | Create, close, list, refine, show issues |
| `/evaluate` | Run the behavioral evaluator against completed work |
| `/audit` | Static code review for defects, missing tests, spec drift |
| `/lint` | Run linters and formatters |
| `/test` | Run tests with optional filters |
| `/pr` | Create a pull request from current branch |

## Project structure

```
.claude/
  agents/              # Agent definitions (domain + specialized)
  skills/              # Skill definitions (slash commands)
  AGENT_TEMPLATE.md    # Template for generating domain agents
issues/
  PLAN.md              # Master issue tracker with phases and dependencies
  TEMPLATE.md          # Issue file format
  shared/              # Cross-domain issues
  <domain>/            # Per-domain issue directories (created during setup)
  evals/               # Evaluator verdict files (if evaluator active)
  sprints/             # Sprint contract files (if sprint-planner active)
CLAUDE.md              # Orchestrator guide — the brain of the system
specs.md               # Behavioral specification (source of truth)
```

## Design principles

**Issue-first**: All work flows through structured issue files with acceptance criteria, dependencies, and technical design. No ad-hoc coding.

**Parallel by default**: The orchestration loop spawns as many agents as possible concurrently. Domain agents work in parallel. Verification steps run in parallel. Sequential execution only happens when there's a real data dependency.

**Separate generation from evaluation**: Agents are bad at judging their own work. The evaluator agent is a distinct role with adversarial framing — it assumes failure until proven otherwise and never reads source code.

**Spec-driven**: `specs.md` is the source of truth for behavior. Agents read it before implementing. The evaluator tests against it. Ambiguities are flagged, not guessed at.

**Context-aware**: Long sessions degrade. The context-manager creates periodic checkpoints so work can resume cleanly in a new session.

**Only the orchestrator commits**: Domain agents write code but never touch git. This prevents conflicts between parallel agents and ensures atomic, well-described commits tied to issue IDs.

## Key insight from the research

> "When asked to evaluate work they've produced, agents tend to respond by confidently praising the work — even when, to a human observer, the quality is obviously mediocre."

The evaluator agent counters this with explicit anti-self-approval measures:
- **Assumes failure** until the behavior is proven correct
- **Never reads source code** — only interacts through public interfaces
- **No benefit of the doubt** — any spec/behavior mismatch is a FAIL
- **Grading rubrics** for subjective quality (UI design, UX) with concrete 1-5 scales

This is the single highest-leverage change you can make to AI-driven development workflows.
