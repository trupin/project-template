---
description: Run all linters and formatters for the project
user_invocable: true
---

Run all linting and formatting checks for the project.

Check if the user passed `--fix` in their arguments. If so, apply auto-fixes.

{{LINT_COMMANDS}}

**This skill will be configured during `/setup` with the actual lint commands for your tech stack.**

Until configured, detect available tools by checking for config files:
- `pyproject.toml` or `ruff.toml` -> `ruff check .` / `ruff format .`
- `package.json` with eslint -> `npm run lint` or `npx eslint .`
- `Cargo.toml` -> `cargo clippy`
- `.golangci.yml` -> `golangci-lint run`
- `tsconfig.json` -> `npx tsc --noEmit`

Report a summary: which checks passed, which failed, and any issues found.
