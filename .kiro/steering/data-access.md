---
inclusion: fileMatch
fileMatchPattern: ['**/adapters/**', '**/kb/**', 'seed.py', 'seed/**', 'infra/**', 'tests/integration/**']
---

# Data Access

How code talks to AWS data stores and backend systems. `tech.md` records which stores the project uses.

## General

- Do not use an ORM or object mapper. Call `boto3` directly inside the adapters. Everything else goes through the adapter interface.
- Create one `boto3` client per service with `botocore.config.Config`, setting `retries={"max_attempts": 3, "mode": "standard"}`, `connect_timeout` and `read_timeout` explicitly. Use 10 s for data stores and a longer read timeout for Bedrock generation calls. Take the region from `core/config.py`.
- Adapters return Pydantic models, never raw `boto3` dicts.
- Each adapter interface is a `typing.Protocol`. Offline tests use an in-memory fake that implements the same protocol. Do not mock `boto3` in unit tests.
- Map every `botocore` error to a typed `unavailable` result inside the adapter. Include which dependency failed, not the raw response.
- Generate identifiers (record references, request IDs, idempotency keys) in code. Never let the model generate them.

## DynamoDB

- Use the low-level client with `TypeSerializer` and `TypeDeserializer` so numbers round-trip as `Decimal`.
- Use `ConsistentRead=True` for reads that feed a decision or a conditional write.
- Never use `Scan` in tool code paths. It is allowed only in the seed script and in test assertions.
- Model uniqueness and limits (for example "total must not exceed a cap") with conditional writes and `TransactWriteItems`, not read-then-write. DynamoDB has no unique secondary indexes, so enforce uniqueness with a guard item written with `attribute_not_exists` in the same transaction.
- Condition expressions cannot do arithmetic. Compute thresholds in code and pass them as values.
- On `TransactionCanceledException`, map `CancellationReasons` back to the transaction items by index, and translate each reason into a typed outcome.
- Make writes idempotent with a caller-supplied key. Store a fingerprint of the request with the guard item. A repeated key with the same fingerprint returns the original result. A repeated key with a different fingerprint is a conflict error.
- Anything that lists records must filter out guard items by an `item_type` attribute.
- Store numbers as DynamoDB numbers and handle them as `Decimal`. Never use float.

## Bedrock Knowledge Bases

See `knowledge-bases.md` for document design, retrieval and evaluation. In adapter code:

- Retrieve with the `bedrock-agent-runtime` `retrieve` API, with a fixed `numberOfResults` and a metadata filter that restricts `doc_type` and any identifying field.
- Parse every retrieved chunk into a typed record. Records that fail to parse are invalid results. Never pass raw chunk text to the agent as fact.
- Build citations from the parsed record and the result's S3 location.
- Apply the score threshold in code, not in a prompt.
- Changing KB documents requires a re-sync with `start_ingestion_job`. The seed script starts it and waits for it to finish before returning.

## Seed and reset

- The seed script is idempotent. `--reset` clears only data this project owns, identified by table and bucket names from config, then reloads the seed data.
- `--reset` refuses to run if the configured resources are not tagged as belonging to this project.
- Seed data is synthetic. Never load real or personal data.
