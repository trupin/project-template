---
description: Run the test suite with optional module or test name filter
user_invocable: true
---

Run the project's test suite.

Parse the user's arguments:
- No args -> run all tests
- Module/domain name (e.g., "api", "core", "ui") -> run tests for that module only
- Test name (e.g., "test_parser") -> run a single test by name
- Both (e.g., "core test_parser") -> run specific test in specific module

{{TEST_COMMANDS}}

**This skill will be configured during `/setup` with the actual test commands for your tech stack.**

Until configured, detect the test framework by checking for:
- `pyproject.toml` with pytest -> `uv run pytest` or `python -m pytest`
- `package.json` with vitest/jest -> `npm test`
- `Cargo.toml` -> `cargo test`
- `go.mod` -> `go test ./...`

Add `-v` or equivalent verbose flag so test names are visible.

Report the results: number of tests passed/failed, and show any failure details.
