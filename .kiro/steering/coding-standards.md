---
inclusion: always
---

# Coding Standards

## Tooling

Code must pass these checks before a task counts as done. Fix the problems the tools report. Don't silence them.

| Area | Format | Lint | Types |
|---|---|---|---|
| Python | `ruff format` | `ruff check` (rule sets `E`, `F`, `I`, `B`, `UP`, `SIM`, `S`, `DTZ`) | `mypy --strict` on `core/`, `handlers/`, `demo_api/`, `agent/` |
| TypeScript | Prettier | ESLint with `typescript-eslint` | `tsc --noEmit` with `strict: true` |

- Put ruff and mypy settings in `pyproject.toml`. Use ruff's default line length.
- Allow `assert` in tests with a per-file ignore: `[tool.ruff.lint.per-file-ignores]` with `"tests/**" = ["S101"]`. Do not disable `S` globally.
- Use `# noqa` or `# type: ignore` only with a specific code and a short reason on the same line.

## Python

- Add type hints to every function signature. Use `X | None`, not `Optional[X]`.
- Model records, tool inputs and tool outputs as Pydantic v2 models with `frozen=True`. The tool JSON Schemas in `contracts/tools/` stay the source of truth, and tests check that the models match them.
- Quantities where exactness matters (money, units, limits, allocations) are `Decimal`. Never use `float` for them, and never build a `Decimal` from a float. Use `Decimal(str(value))` or parse from text.
- Use `StrEnum` for fixed value sets such as statuses and outcomes.
- Timestamps are timezone-aware UTC in ISO 8601. Calendar dates are `datetime.date`.
- Outcomes that the contracts define (not found, validation error, conflict, unavailable, and so on) are returned as typed result objects. Do not raise them as exceptions across the tool boundary.
- Catch `botocore` errors only in the adapter layer and map them to an "unavailable" result. Never let a raw AWS error reach the agent, and never fall back to a guessed value.
- No bare `except:` and no `except Exception: pass`. Catch what you can handle, log with context, and let the rest propagate.
- Use the structured logger from `core/logging.py`. Do not use `print`. Never log credentials, full `.env` contents, personal data beyond what the audit trail requires, or raw AWS responses that might contain them.
- Keep functions small and pure where possible. Inject I/O dependencies (KB client, adapters, clock, ID generator) so `core/` can be tested offline.
- Write docstrings on public functions that state what the function does. When the project has a spec, also name the requirement, for example `Implements Requirement 6.3`.
- No commented-out code. A `TODO` needs a reason and an owner or ticket.

## TypeScript (UI)

See `react-ui.md` for UI rules. The type and lint checks above apply to all TypeScript.

- Do not use `any`. Type the API request and response explicitly.

## Naming

- Python: `snake_case` modules and functions, `PascalCase` classes, `UPPER_SNAKE_CASE` constants.
- TypeScript: `PascalCase` components and their files, `camelCase` everything else.
- Use the spec's domain terms for identifiers, and keep one term per concept. Record the glossary in the design so code, prompts, schemas and tests agree.
