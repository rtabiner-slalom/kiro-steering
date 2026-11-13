---
inclusion: always
---

# Product: <Project name>

<!--
Template. Fill in every section before starting work, then delete this comment.
Keep it short. The spec holds the detail; this file holds what the agent must never forget.
Remove any section that does not apply rather than leaving placeholders behind.
-->

## Purpose

<One paragraph: what the PoC proves, who it is for, and what "success" looks like. Say whether the tools/backend or the demo UI is the real deliverable.>

## Users

- **<User type>:** <what they do and what they need from the system>
- **<Demo presenter>:** <resets data, runs known scenarios>
- **<Operator / reviewer>:** <audits tool calls and the sources behind each answer>

## Core flow

<Numbered steps of the main conversation or workflow, including the decision points and what happens on each branch.>

1. <Collect inputs. Ask only for what is missing.>
2. <Look up / retrieve.>
3. <Decide with deterministic code.>
4. <Tell the user the result before offering any action.>
5. <Act only after explicit confirmation.>

## Non-negotiable rules

<!-- Examples to adapt or delete:
- Calculations and rule decisions are deterministic code. The model never does the arithmetic.
- Factual statements about policy, limits or eligibility come only from retrieved content, with citations. Never from model knowledge.
- If a knowledge base or backend system is unavailable, say the check could not be completed. Never guess.
- Never take an irreversible action without explicit user confirmation of the exact values.
- Log every tool call with request ID, inputs, outputs and the sources used.
- The UI always shows the banner: "<demo / synthetic data banner text>".
-->

- <Rule>

## Scope

- **In scope:** <list>
- **Stretch, built last:** <list, or remove>
- **Out of scope for the PoC:** <list. Note which of these are covered as a production pathway in the design.>

## Known risks and open questions

- <Risk or unknown, and the mitigation or the decision still needed.>
