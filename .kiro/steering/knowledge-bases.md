---
inclusion: fileMatch
fileMatchPattern: ['**/kb/**', 'seed/**', 'seed.py', 'docs/kb-schema.md', 'scripts/**', 'tests/evals/**', 'tests/integration/**']
---

# Knowledge Bases

Guidance for Amazon Bedrock Knowledge Bases backed by Amazon S3 Vectors. `data-access.md` covers the adapter code. Service limits change, so check the current docs for S3 Vectors and Bedrock Knowledge Bases before relying on a number here.

## Design

- Decide the number of knowledge bases from the data, not by habit. Separate KBs suit data with different owners, update cycles, or access rules. Records of one kind with a `doc_type` field fit one KB.
- Treat each KB as structured reference data. One short document per record, a documented field schema, and a `.metadata.json` sidecar so retrieval can be filtered.
- Record the schema in `docs/kb-schema.md`. It is the contract for any later ingestion pipeline.
- Documents carry a `source` field so every answer can cite where the fact came from.
- Never mix facts that change on different cycles in one document.

## Chunking and size

- For record-style documents, use chunking `NONE`, so each chunk is one whole record. Keep documents small enough to stay within the embedding model's input limit.
- For prose documents (policies, manuals), use a chunking strategy suited to the content, and keep headings with their text. Record the choice in the decisions log in `tech.md`.

## Metadata

- Sidecar fields are the filter keys. Include `doc_type` and the identifying key of the record.
- S3 Vectors treats metadata as filterable by default. Filterable metadata has a smaller size limit than total metadata per vector, and Bedrock stores its own metadata alongside yours. Check the current limits before adding fields, and keep sidecars minimal.
- Retrieval always passes a metadata filter. Do not rely on semantic similarity alone to pick the right record.

## Retrieval

- Fixed `numberOfResults`, a metadata filter, and a score threshold set in config.
- Near-identical names (for example model numbers differing by one digit) are the main failure mode. Mitigate with filters, a threshold, and having the agent read the matched record back to the user for confirmation.
- Calibrate the threshold with real queries. Keep a script in `scripts/` that reports the score gap between correct and incorrect matches, and rerun it whenever documents or the embedding model change.
- Parse retrieved chunks into typed records. Invalid records are failures, not partial answers.

## Ingestion and seed

- Documents live under `seed/data/` for the PoC, uploaded to the KB's S3 bucket by the seed script.
- After any change to documents, start an ingestion job and wait for it to complete. Fail the seed run if ingestion fails.
- Keep the seed data complete enough to exercise each scenario in `seed/scenarios.md`, including not found, ambiguous and unavailable cases.

## Evaluation

- Keep a small evaluation set in `tests/evals/`: query, expected record, and expected outcome when nothing should match.
- Report hit rate and the threshold gap. Run it after any change to documents, chunking, metadata or the embedding model.
- Include near-duplicate and out-of-vocabulary queries.

## Production path

- Real documents (often PDFs) need an ingestion pipeline that extracts the same schema, for example with Bedrock Data Automation or LLM extraction. Plan it as a separate piece of work with its own validation, because the structure of real documents is the biggest unknown.
- Real rules may be more complex than the PoC schema (per-item caps, per-compartment limits, conditions). Raise this in the design as a risk.
