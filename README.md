# Kiro steering: AWS chatbot / agent PoCs

Reusable Kiro steering files for proof-of-concept chatbot and agent projects on AWS. The blueprint is Amazon Bedrock (agents, AgentCore, Knowledge Bases), a Python core of deterministic tools, and a throwaway React demo UI. The stance throughout is: build the PoC the way production would be built, at a smaller scale, and take no hacks.

## Using these files in a new project

1. Copy `.kiro/steering/` into the new repository.
2. Fill in `product.md` completely. It is a template. Delete sections that do not apply.
3. Review `tech.md`: edit the component mapping table and add the project's configuration keys.
4. Review `structure.md` and replace the tree with the project's actual layout once it settles.
5. Delete files for parts the project does not have (for example `react-ui.md` if there is no UI, `knowledge-bases.md` if there are no knowledge bases).
6. Create specs under `.kiro/specs/` with numbered requirements. The steering files refer to requirement numbers in docstrings and test names when a spec exists.

## Files

| File | Inclusion | Purpose |
|---|---|---|
| `product.md` | always | Project template: purpose, users, flow, non-negotiable rules, scope, risks |
| `tech.md` | always | PoC-to-production stance, component mapping, libraries, configuration |
| `structure.md` | always | Layout and dependency rules |
| `coding-standards.md` | always | Python and TypeScript standards, tooling |
| `testing.md` | always | How tests are written: unit, property, integration, conversation, UI |
| `local-dev.md` | always | Toolchain, setup, safe versus confirm-first commands |
| `security.md` | always | Secrets, IAM, untrusted content, data handling, guardrails |
| `git-and-ci.md` | always | Git rules, CI baseline, definition of done |
| `data-access.md` | file match | Adapters, DynamoDB, Knowledge Base access code, seed and reset |
| `agents.md` | file match | System prompt, tool design, Strands, AgentCore |
| `knowledge-bases.md` | file match | KB design, chunking, metadata, retrieval, evaluation |
| `aws-cdk.md` | file match | CDK structure, tagging, IAM, outputs, deploy workflow |
| `react-ui.md` | file match | Demo UI rules |

File-match patterns assume the layout in `structure.md`. If the project uses different folder names, update the `fileMatchPattern` lists, or the files will not load.

## Keeping these files current

Bedrock, AgentCore and Strands change quickly. The steering files tell the agent to check current documentation before using API names, so avoid adding specific API signatures here. Record project-specific decisions in the project's own `tech.md` decisions log.
