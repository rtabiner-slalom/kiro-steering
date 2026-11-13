---
inclusion: always
---

# Technology Stack

Specs stay technology-neutral. This file records the concrete choices they map to, and the engineering stance for a PoC that is meant to grow into production.

## PoC to production

Build the PoC the way production would be built, at a smaller scale. Take shortcuts on scale and polish, never on correctness or safety.

**Acceptable PoC shortcuts**

- Runs locally against real AWS services, single account and region, no multi-environment pipeline.
- Synthetic data only, re-seedable, with `RemovalPolicy.DESTROY` on data stores.
- No production auth, WAF, custom domains or autoscaling tuning.
- Minimal demo UI.
- Manual deploys from a developer machine.
- Fewer edge-case tests than production, but every spec acceptance criterion is still covered.

**Not acceptable, even in a PoC**

- Hard-coded secrets, resource IDs, model IDs or regions.
- Business rules or arithmetic done by the model, or duplicated inside handlers.
- Bypassing the adapter interfaces to reach a data store.
- Skipping validation at trust boundaries.
- Swallowed errors, or silent fallbacks to a guessed value.
- Disabled, weakened or skipped tests.
- Commented-out code, or `TODO` without a reason and an owner.

Record every deliberate shortcut in the design under "PoC shortcuts", with the production fix next to it. A shortcut with no recorded fix is a hack.

## Platform

- **Cloud:** AWS.
- **Region:** set in configuration. Before deploying, confirm that the chosen Bedrock models (or inference profiles), the vector store and AgentCore are available in the target region.
- **Credentials:** the standard AWS credential provider chain only. Never put keys in code, committed config or logs.

## Component mapping

Edit this table for the project. The defaults below are the usual blueprint.

| Spec term | PoC technology | Production path |
|---|---|---|
| Agent | Local Python agent on the Strands Agents SDK, Claude model on Amazon Bedrock | Same code hosted on Amazon Bedrock AgentCore Runtime |
| Tools / Core_Logic | Python package with thin handlers | AWS Lambda behind AgentCore Gateway (MCP), or a Bedrock Agent action group |
| Knowledge base(s) | Amazon Bedrock Knowledge Base(s) with S3 data source and Amazon S3 Vectors | Same, fed by a real ingestion pipeline |
| Backend system(s) | `<mock: DynamoDB / S3 / stub>` behind an adapter interface | Adapter to the real system |
| Conversation memory | In-process, per session ID | AgentCore Memory |
| Demo API | Local FastAPI service, CORS for the Vite dev server only | API Gateway + Lambda, AgentCore Runtime invocation, or the existing chatbot channel |
| Demo UI | React + Vite + TypeScript, plain CSS | React build on S3 + CloudFront, or Amplify |
| Infrastructure | AWS CDK (Python) | Same |
| Security and ops | Not in PoC | Cognito or IAM auth, Bedrock Guardrails, AgentCore Identity, CloudWatch and AgentCore Observability, X-Ray |

Where an existing chatbot is being extended, name it here and note how the tools reach it.

## Languages and libraries

- **Python 3.12+** for tools, agent, API, seed script and CDK.
  - `boto3` for AWS calls. Data-store clients use 3 attempts and 10 s connect/read timeouts. Bedrock model calls use a longer read timeout (start at 60 s) because generation is slow. Always set both explicitly.
  - Pydantic v2 for models. JSON Schemas in `contracts/` are the source of truth for tool contracts.
  - `strands-agents` for the agent.
  - `fastapi` and `uvicorn` for the demo API, listening on the loopback interface only.
  - Structured JSON logging to stdout in a CloudWatch-compatible format.
- **TypeScript** for the UI: React and Vite, with plain CSS.
- Pin dependency versions through lock files (`uv.lock`, `package-lock.json`). AgentCore and Strands move quickly, so check the current AWS and Strands docs for API names before using them, and do not rely on memory.

## Testing

See `testing.md` for how tests are written. Unit and property tests run offline. Integration and conversation tests run against real AWS and are marked `integration`.

## Configuration

Read every resource identifier from environment variables, or from a `.env` file that is never committed.

- Always required: `AWS_REGION`, `AGENT_MODEL_ID`, `DEMO_UI_ORIGIN` (URL with scheme and host).
- Per resource, as the design lists them: knowledge base IDs, data source IDs, bucket names, table names, thresholds and feature flags.
- `.env.example` lists every key with placeholder values. The design's Configuration table is the source for the key list.
- Take resource values from the CDK stack outputs.
- Fail fast at start-up if any value is missing or invalid.

## Common commands

See `local-dev.md`.

## Decisions log

Record choices that are not obvious from the code. One row each.

| Decision | Choice | Why | Revisit when |
|---|---|---|---|
| <e.g. number of knowledge bases> | <choice> | <reason> | <trigger> |
