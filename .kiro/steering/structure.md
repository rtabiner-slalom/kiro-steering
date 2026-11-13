---
inclusion: always
---

# Project Structure

This is the default layout. Adapt it to the project, then record the actual layout here and keep it current.

```
.
├── core/                    # Business logic: shared, framework-free Python
│   ├── <domain modules>.py  # Pure functions for rules and calculations
│   ├── results.py           # Typed outcome objects (Citation, ValidationIssue, Unavailable, ...)
│   ├── contracts.py         # Pydantic tool input/output models and schema loader
│   ├── tool_runner.py       # run_tool: request IDs, schema checks, structured log entries
│   ├── kb/                  # Knowledge base client, record models, parser/printer
│   ├── adapters/            # Interfaces (Protocols) plus AWS-backed implementations
│   ├── config.py            # Load and validate configuration, fail fast
│   └── logging.py           # Structured JSON logger with request IDs
├── contracts/
│   ├── tools/               # JSON Schemas: input and output for each tool
│   └── <system>-api.yaml    # OpenAPI contracts for backend systems
├── handlers/                # Thin adapters only, no business logic
│   ├── local.py             # Direct Python calls / Strands tool wrappers
│   ├── lambda_gateway.py    # Lambda handler for AgentCore Gateway (MCP)
│   └── action_group.py      # Bedrock Agent action group handler (only if needed)
├── agent/                   # Agent (Strands)
│   ├── prompt.md            # System prompt, versioned in git
│   ├── agentcore_app.py     # AgentCore Runtime entrypoint wrapper (when used)
│   └── recorder.py          # Per-turn recorder of tool calls, arguments and results
├── demo_api/                # FastAPI app
├── ui/                      # React + Vite + TypeScript demo UI
├── infra/                   # AWS CDK (Python) app
├── scripts/                 # One-off helpers (for example threshold calibration)
├── seed/
│   ├── data/                # Synthetic documents and sidecars
│   └── scenarios.md         # Documented demo and test scenarios
├── seed.py                  # Seed script, supports --reset
├── docs/                    # Schemas and notes needed for the production pathway
├── tests/
│   ├── unit/                # Offline
│   ├── property/            # Offline, hypothesis
│   ├── integration/         # Real AWS
│   ├── conversations/       # Scripted agent conversations, real AWS
│   └── evals/               # Retrieval and agent quality sets (when used)
├── .kiro/
│   ├── steering/
│   └── specs/
└── .env.example
```

## Rules

- Business rules live only in `core/`. Handlers translate request and response formats and nothing else.
- `core/` must not import Strands, FastAPI, boto3 clients outside `adapters/` and `kb/`, or Lambda-specific modules. This keeps the tools portable across local, Gateway and action group hosting.
- Backend systems are reached only through the adapter interfaces in `core/adapters/`.
- The tool JSON Schemas in `contracts/tools/` are the single source of truth. The agent, Gateway and action group definitions all load them.
- Deterministic calculations are pure functions with no I/O.
- Never hard-code resource names, model IDs or the region. Read them from `core/config.py`.
- Put the system prompt in `agent/prompt.md`, not in a Python string.
