---
inclusion: fileMatch
fileMatchPattern: ['infra/**', 'cdk.json', 'cdk.context.json']
---

# Infrastructure as Code (AWS CDK, Python)

## Structure

- One CDK app in `infra/`. Start with one stack. Split into stacks only when lifecycles differ (for example data versus compute).
- Use constructs for repeated patterns (for example "knowledge base with bucket and vector index"). Prefer L2 constructs. Use L1 (`Cfn*`) only when no L2 exists, with a comment.
- Pin the CDK library version through `uv.lock`.
- Read account and region from the environment (`CDK_DEFAULT_ACCOUNT`, `CDK_DEFAULT_REGION`). Do not hard-code them.

## Resources and naming

- Let CDK generate physical names where possible, to avoid clashes. Where a fixed name is needed, derive it from a project prefix in config.
- Tag every resource with `Project`, `Environment` (`poc`) and `Owner`. Apply tags at the app level. The seed script's `--reset` safety check relies on these tags.
- Data stores for synthetic data use on-demand billing and `RemovalPolicy.DESTROY` with auto-delete where applicable. Production would retain.
- Enable encryption at rest with AWS-managed keys as a minimum, block public access on every bucket, and enforce TLS.

## IAM

- Follow `security.md`. Grant with the L2 `grant_*` methods where they exist, and check the resulting policy for wildcards.
- Use `cdk diff` to review IAM changes before deploying.
- Run `cdk-nag` (AwsSolutions pack) in the app. Fix findings, or suppress them with a specific rule ID and a reason. Never suppress broadly.

## Outputs

- Export every value that `.env` needs as a `CfnOutput`: knowledge base IDs, data source IDs, bucket names, table names, function ARNs, and model or profile IDs where deployed.
- Keep `.env.example` in sync with the outputs.

## Workflow

- Run `cdk synth` and `cdk diff` freely. Confirm the target account and region with the user before `cdk deploy` or `cdk destroy`.
- Before deploying, confirm that the Bedrock models, S3 Vectors and AgentCore are available in the target region.
- Bedrock model access and some account-level settings are not managed by CDK. Document these manual steps in the README.
- Tear down demo environments when finished, and tell the user which resources incur cost while they exist.
