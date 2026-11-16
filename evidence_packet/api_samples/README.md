# API and Data Exchange Samples

## Current exchange mechanism
The agreed initial exchange mechanism is file-based JSON.

## Input
The Drug Safety Evidence Integration workstream prepares normalized evidence records in JSON format.

Reference input:
- ../outputs/normalized_evidence_input.json

## Output
The producing workstream generates an integrated evidence JSON output.

Reference output:
- ../outputs/integrated_evidence_output.json

## Integration status
- JSON file-based exchange: Agreed in principle.
- Shared schema and final field mapping: Pending.
- Receiving Biomedical Evidence Engine import adapter: Pending.
- Structured QC output from receiving engine: Pending.
- Live API integration: Not implemented or verified.

## Responsibility boundary
The Drug Safety workstream prepares and exports normalized evidence with provenance.
The receiving Biomedical Evidence Engine is responsible for its import adapter, compatibility checks, structured QC output, and receiving-side error handling.

Uncertain records must remain subject to human review.

## Important limitation
These files demonstrate the producing workstream's local input and output. They do not prove successful ingestion by the receiving Biomedical Evidence Engine.
