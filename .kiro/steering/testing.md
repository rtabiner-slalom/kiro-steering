---
inclusion: always
---

# Testing

`tech.md` and `local-dev.md` list the tools and commands. When the project has a spec, its requirements define the cases each suite must contain. This file covers how tests are written.

## Layout and markers

- `tests/unit/`, `tests/property/`, `tests/integration/`, `tests/conversations/` and `tests/evals/` follow `structure.md`.
- Register markers in `pyproject.toml` and run pytest with `--strict-markers`. Mark every integration and conversation test `@pytest.mark.integration`. Also mark conversation tests `@pytest.mark.conversation`.
- Offline suites must not reach the network. Use `pytest-socket` with `--disable-socket` for runs with `-m "not integration"`.
- Name tests after the behaviour. When the project has a spec, reference the requirement or acceptance criterion in the test docstring so coverage can be traced.

## Unit tests

- Test `core/` through its public functions, with in-memory fakes that implement the adapter Protocols. Do not mock `boto3`.
- Write exact quantities as `Decimal("99.99")` strings, never as float literals.
- Assert exact values, not just success.
- Use one `pytest.mark.parametrize` table per boundary set, with readable `ids`.
- Validate tool inputs and outputs against the JSON Schemas in `contracts/tools/` with `jsonschema`.
- Cover every typed outcome a tool can return, including `unavailable`.

## Property tests (hypothesis)

- Use `@settings(max_examples=100)` or more, or the spec minimum if higher.
- Generate bounded values explicitly, for example `st.decimals(..., places=2, allow_nan=False, allow_infinity=False)` within the spec ranges.
- Use them for invariants, determinism, and round-trips: print-then-parse of KB documents, and JSON encode-then-decode of tool responses.
- Register a `ci` profile with `derandomize=True` so CI failures can be reproduced.

## Integration tests (real AWS)

- A session-scoped fixture seeds data once (`seed.py --reset`) before any test that needs it.
- Read all resource identifiers from `core/config.py`. Never hard-code them.
- Use a new `uuid4` for every idempotency key and test record, unless the test is about idempotency.
- Concurrency tests use `concurrent.futures.ThreadPoolExecutor` with at least 10 workers. Assert the final state from a consistent read.
- Tests that call Bedrock cost money and vary between runs. Keep them few, and keep the model ID in config.

## Conversation and agent tests

- Drive the agent with scripted user turns against seeded scenarios from `seed/scenarios.md`.
- Assert on the recorded tool calls and their arguments and results first. Check reply text only for the values the spec requires, compared with the tool output.
- Run each conversation 3 times with `@pytest.mark.parametrize("run", range(3))`. A test passes only when all runs pass.
- Add negative conversations: missing fields, prompt-injection attempts in user text, unavailable backends, and a user who tries to skip confirmation.
- Retrieval quality sets (query to expected document) live in `tests/evals/`. See `knowledge-bases.md`.

## UI tests

- Use Vitest in run mode (`npx vitest --run`) with React Testing Library for the main flow and the error states. Do not test styling.

## Rules

- Run tests once, never in watch mode.
- A failing test means the code or the test is wrong. Do not weaken assertions, add retries or skip tests to make a suite pass.
- Do not chase a coverage number. Cover every acceptance criterion and every typed outcome.
- Write or update tests in the same task as the code they cover, when the spec task asks for them.
