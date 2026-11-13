---
inclusion: always
---

# Local Development

The PoC runs the agent, tools, API and UI on a developer machine against real AWS services. Nothing is emulated locally except in offline unit and property tests.

## Toolchain

- **Python:** 3.12+, managed with `uv`. One `pyproject.toml` at the repo root holds runtime dependencies plus `dev` and `infra` dependency groups. Commit `uv.lock`.
- **Node:** current LTS, with `npm` in `ui/`. Commit `package-lock.json`.
- **AWS CDK CLI:** installed globally or run with `npx cdk`.
- Do not add a package manager, task runner or container setup without asking.

## First-time setup

1. `uv sync --all-groups` creates `.venv` and installs everything.
2. `cd ui && npm ci`
3. Sign in with the standard credential chain, for example `aws sso login --profile <profile>` and then `export AWS_PROFILE=<profile>`.
4. `cd infra && cdk deploy`, then copy the stack outputs into `.env`.
5. `uv run python seed.py --reset`

## Configuration files

- `.env` holds real values. It is git-ignored and must never be committed, printed or logged.
- `.env.example` is committed. It lists every required key with placeholder values only. Update it whenever a key is added.
- Code reads configuration only through `core/config.py`, which fails fast on missing or invalid values.
- Never hard-code a profile, account ID or region. They come from the environment.

## Long-running processes

The assistant must not start these. Give the user the command to run in their own terminal.

- Demo API: `uv run uvicorn demo_api.app:app --host 127.0.0.1 --port 8000 --reload`
- Demo UI: `cd ui && npm run dev` (Vite on `http://localhost:5173`, which must match `DEMO_UI_ORIGIN`)

## Commands the assistant may run

These finish on their own and do not write to AWS.

- Format: `uv run ruff format .`
- Lint: `uv run ruff check --fix .`
- Type-check: `uv run mypy`
- Offline tests: `uv run pytest -m "not integration"`
- UI checks: `cd ui && npm run lint && npx tsc --noEmit`
- UI tests: `cd ui && npx vitest --run`
- CDK synth and diff: `cd infra && cdk synth`, `cdk diff`

## Commands that need confirmation first

These write to AWS or cost money. Before running any of them, confirm the target account and region with the user.

- `cd infra && cdk deploy` and `cdk destroy`
- `uv run python seed.py --reset`
- `uv run pytest -m integration` (also calls Bedrock)

## Rules

- Run each command once. No watch modes.
- If a command fails because credentials have expired, tell the user to sign in again. Do not try to work around it.
