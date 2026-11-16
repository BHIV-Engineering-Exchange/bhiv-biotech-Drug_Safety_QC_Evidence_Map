# Biomedical Evidence Engine Integration Boundary

## BHIV Drug Safety Evidence Integration x Biomedical Evidence Engine

**Organization:** BHIV Biotech  
**Producing Workstream:** Drug Safety Evidence Integration  
**Receiving Workstream:** Biomedical Evidence Engine  
**Integration Contact:** Surendra  
**Document Version:** 1.0  
**Status:** Agreed - Implementation Pending
**Date:** November 2026

---

## 1. Purpose

This document establishes the proposed technical and operational boundary between the Drug Safety Evidence Integration workstream and the Biomedical Evidence Engine.

The objective is to define compatible data exchange, schema mapping, provenance requirements, normalization ownership, QC responsibilities, error handling, and human-review boundaries.

The integration will reuse existing capabilities and avoid unnecessary duplication of evidence normalization and processing.

This document distinguishes existing functionality from proposed and pending integration work.

## 2. Existing Workstream Capabilities

### 2.1 Drug Safety Evidence Integration

The producing workstream currently uses:

- Schema name: BHIV_Drug_Safety_Evidence
- Schema version: 1.0.0
- Input file: normalized_evidence.json
- Output file: integration/integrated_evidence_output.json

Existing capabilities include:

1. Evidence ingestion.
2. Source-specific normalization.
3. Provenance preservation.
4. Drug/product matching.
5. Evidence validation.
6. Duplicate detection.
7. Conflict detection.
8. Human-review flagging.
9. Structured evidence output generation.

The current input uses a JSON wrapper containing schema_name, schema_version, record_count, and records.

### 2.2 Biomedical Evidence Engine

Based on the receiving workstream's implementation review, it currently uses:

- Schema name: EvidenceRecord
- Schema version: 1.1

Reported existing capabilities include:

1. Required-field validation.
2. Validity flag generation.
3. Validation error reporting.
4. OpenFDA/FAERS-specific normalization.

The following capabilities require further implementation or verification:

- Import adapter for the shared input format.
- Structured QC output document.
- Invalid-record preservation.
- Explicit duplicate-versus-conflict reporting.
- Improved human-review routing.

## 3. Proposed Integration Architecture

The integration will initially use file-based JSON exchange.

### 3.1 Data Flow

1. Source evidence is received by the Drug Safety Evidence Integration workstream.
2. Source-specific normalization and standardization are performed.
3. Drug/product matching and provenance preservation are performed.
4. Records are exported in the agreed shared JSON format.
5. The Biomedical Evidence Engine receives the records through an import adapter.
6. The receiving engine performs QC validation.
7. Validation results and identified data-quality issues are reported.
8. Invalid, conflicting, and uncertain records are preserved or flagged for review.
9. Structured QC results are returned as JSON.

### 3.2 Exchange Mechanism

Initial exchange:

- JSON file input.
- JSON file output.
- No direct API dependency.

API integration may be considered in a future phase.

## 4. Schema Compatibility

The two workstreams currently use different schema definitions.

| Attribute | Producing Workstream | Receiving Workstream |
|---|---|---|
| Schema name | BHIV_Drug_Safety_Evidence | EvidenceRecord |
| Schema version | 1.0.0 | 1.1 |
| Input representation | JSON wrapper with records array | JSON array |
| Normalization | Source-specific normalization | OpenFDA/FAERS normalization |
| Output | Integrated evidence JSON | Current validator returns validity flags and errors |

A schema mapping or import adapter is required before direct exchange.

The canonical shared schema and its versioning strategy must be mutually agreed upon before implementation is considered complete.

## 5. Proposed Provenance Mapping

The following mapping is proposed based on the receiving workstream's feedback.

| Producing Field | Receiving Field | Decision |
|---|---|---|
| original_source | source/source-system | Map |
| original_evidence_id | source_record_id | Map |
| ingestion_timestamp | retrieved_at | Map |
| source_version | No direct equivalent | Preserve separately |
| transformation_path | Provenance transformation information | Map |

### 5.1 Source Version

The source_version field represents the version of the original source dataset or source system.

It must remain separate from schema_version.

### 5.2 Transformation Path

The producing workstream already represents transformation_path as an ordered JSON array.

Example:

[
  "existing_csv",
  "field_mapping",
  "canonical_normalization"
]

This representation is compatible with the proposed sequence-of-steps approach.

The receiving workstream should confirm whether its provenance representation accepts this structure directly.

### 5.3 Additional Evidence Fields

| Field | Proposed Requirement |
|---|---|
| Study ID | Optional |
| Protocol reference | Optional |
| Study details | Optional |
| Synthetic-data status | Required in shared contract |

Study-related fields should be included when applicable and available.

They should not be mandatory for every evidence type, particularly non-study-based records such as adverse-event reports.

### 5.4 Synthetic-Data Declaration

The proposed shared contract will include:

synthetic_data_status

Allowed values:

- real
- synthetic
- unknown

Unknown must be used when the origin of the data cannot be verified.

This is a proposed addition and is not currently present in the producing workstream's canonical schema.

## 6. Input and Output Format

### 6.1 Producing Workstream Input

The current producing workstream uses:

{
  "schema_name": "BHIV_Drug_Safety_Evidence",
  "schema_version": "1.0.0",
  "record_count": 10,
  "records": []
}

### 6.2 Shared Integration Input

The receiving workstream has requested a JSON array.

The proposed import adapter will extract the records array from the producing workstream's JSON wrapper.

The adapter must preserve the original evidence fields and provenance information.

### 6.3 QC Output

The receiving workstream should implement a structured JSON QC output document.

The proposed output should contain:

- Evidence identifier.
- Validation status.
- Validation errors.
- Duplicate status.
- Conflict status.
- Human-review requirement.
- Processing status.

The exact output schema and field names remain subject to agreement.

## 7. Normalization and Processing Ownership

### 7.1 Producing Workstream Responsibilities

The Drug Safety Evidence Integration workstream will own:

- Source-specific normalization.
- Evidence standardization.
- Drug/product matching.
- Provenance preservation.
- Shared-format JSON export.
- Source-level evidence preparation.

### 7.2 Receiving Workstream Responsibilities

The Biomedical Evidence Engine will own:

- Shared-format import adapter.
- Incoming record compatibility.
- QC validation.
- Structured QC output generation.
- Validation error reporting.
- Duplicate-versus-conflict reporting.
- Invalid-record preservation.
- Human-review routing improvements.

The receiving engine should not independently repeat source-specific normalization already completed by the producing workstream.

## 8. Error Handling and Human Review

Both workstreams agree that uncertain or incomplete evidence must not be silently discarded or automatically resolved.

| Scenario | Required Behaviour |
|---|---|
| Valid record | Report successful validation |
| Invalid record | Preserve and report validation errors |
| Incomplete record | Flag missing information |
| Duplicate record | Identify and report duplicate status |
| Conflicting record | Preserve both records and flag conflict |
| Unresolved entity match | Preserve unresolved status |
| Uncertain evidence | Route for human review |

Records sharing the same source identifier must be evaluated to distinguish genuine duplicates from conflicting records.

Invalid records must remain available for review through an agreed retention or error-reporting mechanism.

Scientific interpretation and decisions must remain under human responsibility.

## 9. Authority Boundaries

The integrated system may:

- Process evidence records.
- Preserve provenance.
- Normalize and standardize evidence.
- Validate data structure.
- Identify duplicates.
- Identify conflicting records.
- Identify unresolved entity matches.
- Generate human-review flags.
- Produce structured QC reports.

The integrated system must not:

- Automatically establish clinical causality.
- Make clinical treatment decisions.
- Independently determine scientific conclusions.
- Make regulatory approval decisions.
- Replace scientific or pharmacovigilance professionals.

## 10. Pending Implementation Requirements

| Requirement | Responsible Workstream | Status |
|---|---|---|
| Agree on canonical shared schema | Both | Pending |
| Finalize schema mapping | Both | Pending |
| Confirm provenance field mapping | Both | Proposed |
| Confirm transformation-path compatibility | Both | Pending |
| Add synthetic-data declaration | Both | Pending |
| Implement shared-format import adapter | Biomedical Evidence Engine | Pending |
| Generate structured QC output | Biomedical Evidence Engine | Pending |
| Preserve invalid records | Biomedical Evidence Engine | Pending |
| Distinguish duplicates and conflicts | Biomedical Evidence Engine | Pending |
| Finalize human-review routing | Biomedical Evidence Engine | Pending |
| Define integration test cases | Both | Pending |

## 11. Integration Acceptance Criteria

The integration boundary will be considered agreed when:

1. Both workstreams approve the shared schema.
2. Schema versioning and compatibility are documented.
3. Mandatory and optional fields are finalized.
4. Provenance mapping is approved.
5. Synthetic-data declaration is agreed upon.
6. JSON input and output formats are documented.
7. Normalization and validation ownership are confirmed.
8. Import adapter requirements are agreed upon.
9. Invalid-record and conflict-handling behaviour is defined.
10. Human-review responsibilities are documented.

Technical integration will be considered complete only after the pending implementation requirements have been implemented and tested.

## 12. Current Integration Status

The two workstreams have agreed in principle on:

- JSON-based exchange.
- File-based initial integration.
- Provenance preservation.
- Producing-workstream normalization ownership.
- Human responsibility for scientific decisions.

The following remain pending:

- Canonical schema agreement.
- Final field mapping.
- Import adapter.
- Structured QC output.
- Invalid-record preservation.
- Duplicate-versus-conflict reporting.
- Final integration testing.

This document records the proposed integration boundary based on the current discussion. It does not claim that live integration has been implemented.

## 13. Conclusion

The proposed integration establishes a clear separation between evidence preparation and downstream QC processing.

The Drug Safety Evidence Integration workstream will prepare normalized, traceable evidence records and export them using the agreed shared format.

The Biomedical Evidence Engine will provide compatible ingestion, QC validation, structured reporting, and appropriate error and human-review handling.

Both workstreams will preserve evidence integrity, provenance, and human decision-making authority throughout the integration process.