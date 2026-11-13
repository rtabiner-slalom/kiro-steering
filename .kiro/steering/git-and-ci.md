---
inclusion: always
---

# Git and CI

## Git

- Never run destructive commands (`reset --hard`, `clean -fd`, force push, branch delete) unless the user asks.
- Stage files by name. Do not use `git add -A` or `git add .`.
- Do not commit unless the user asks. When asked, write commits in imperative mood with a short subject line, and a body that explains why when it is not obvious.
- One logical change per commit. Do not mix formatting-only changes with behaviour changes.
- Never commit `.env`, credentials, `cdk.out/`, `.venv/`, `node_modules/`, `.hypothesis/`, or build output.
- Check that `.gitignore` covers these before the first commit.
- Never modify git config or skip hooks.

## Repository files

- `README.md` covers what the project is, prerequisites, first-time setup, how to run the demo, how to run tests, and how to tear down. Keep it in step with `local-dev.md`.
- `.env.example` lists every required key with placeholders.
- `docs/` holds schemas and notes needed for the production pathway.

## CI

Add a CI pipeline once the first code is committed. Keep it small and offline.

- Run on every pull request: `ruff format --check`, `ruff check`, `mypy`, `pytest -m "not integration"`, `npm run lint`, `tsc --noEmit`, `vitest --run`.
- Run `pip-audit` and `npm audit` on a schedule and report the results.
- Run `cdk synth` to catch infrastructure errors.
- Integration and conversation tests call real AWS, so they run manually or on a protected branch, with credentials from OIDC. Never store long-lived keys in CI.
- Use the `ci` hypothesis profile.

## Definition of done

A task is done when:

- the code meets every acceptance criterion in the spec task,
- format, lint, type and offline test checks pass,
- tests exist for new behaviour, as the spec task asks,
- no secrets, resource IDs or regions are hard-coded,
- the README, `.env.example` and steering files that describe the change are updated,
- any deliberate PoC shortcut is recorded with its production fix.
