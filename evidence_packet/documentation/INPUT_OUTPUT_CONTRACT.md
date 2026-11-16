# BHIV Drug Safety Evidence Integration
## Input/Output Contract

### 1. Purpose
This document describes the input structure, processing workflow, and output structure of the BHIV Drug Safety Evidence Integration module.

### 2. Schema Information
- Schema name: BHIV_Drug_Safety_Evidence
- Schema version: 1.0.0
- Integration name: BHIV Drug Safety Evidence Integration
- Integration version: 1.0.0

### 3. Input Format
The integration accepts a JSON file containing:
- schema_name
- schema_version
- record_count
- records

The records field contains the individual normalized drug safety or quality evidence records.

The current input file is normalized_evidence.json.

### 4. Required Evidence Fields
According to canonical_schema.json, each evidence record requires:

1. evidence_id
2. source
3. source_organisation
4. evidence_type
5. drug_product_identifier
6. observation
7. date
8. population_context
9. measurement_result
10. supporting_documentation
11. evidence_provenance
12. evidence_quality
13. validation_status
14. limitations
15. reviewer_action
16. human_review_flag

The date follows YYYY-MM-DD format.

The human_review_flag field is Boolean.

### 5. Provenance Information
The evidence_provenance object contains:
- original_source
- original_evidence_id
- transformation_path
- ingestion_timestamp
- source_version

These fields help preserve the origin and transformation history of evidence.

### 6. Processing Workflow
The integration performs the following operations:

1. Loads the normalized JSON evidence file.
2. Extracts the evidence records.
3. Creates internal processing copies and maps selected canonical fields to legacy processing fields.
4. Validates individual records.
5. Detects potential duplicate relationships.
6. Detects potential evidence conflicts.
7. Matches records against the requested drug or product.
8. Generates human-review items when review conditions are identified.
9. Combines the results into a structured JSON output.

### 7. Internal Field Mapping
The integration uses the following mappings for internal processing:

- drug_product_identifier -> drug_product
- evidence_id -> source_identifier

Evidence types and validation statuses are also mapped to internal processing categories.

These mappings are applied to processing copies. The original canonical records are retained in the output.

### 8. Output Format
The integration generates:

integration/integrated_evidence_output.json

The output contains:

- integration_name
- integration_version
- source_schema
- source_schema_version
- record_count
- query
- matching
- validation_results
- duplicate_results
- conflict_results
- human_review_results
- records

### 9. Output Section Descriptions

**Matching:** Reports the requested query, match status, matched products, and whether matching requires human review.

**Validation results:** Reports each evidence ID, validation status, validation errors, and human-review requirement.

**Duplicate results:** Contains detected duplicate relationships.

**Conflict results:** Contains detected evidence conflicts.

**Human review results:** Identifies evidence requiring human assessment, including the reason and related evidence IDs.

**Records:** Preserves the original canonical evidence records, including their provenance information.

### 10. Observed Verification Results
During fresh GitHub ZIP verification:

- Input records processed: 10
- Drug query: Drug A
- Matching status: EXACT_MATCH
- Validation results: 10 records reported VALID
- Duplicate relationships: 0
- Conflict relationships: 0
- Human-review items: 9
- Integration output generated successfully.

These are observed results for the current test dataset, not guarantees for every future dataset.

### 11. Error and Input Handling
The current implementation:
- Raises FileNotFoundError if the specified input file does not exist.
- Uses Python JSON loading; malformed JSON can raise a JSON decoding error.
- Defaults to an empty records list if the records key is absent.
- Creates the output directory when saving the generated JSON file.

The loader does not itself establish that every input record complies with the complete canonical schema. Record validation is handled separately by the validation component.

### 12. Authority and Safety Boundaries
The integration supports evidence organization and review preparation.

It does not:
- Automatically determine clinical causality.
- Make clinical decisions.
- Make regulatory approval decisions.
- Replace scientific or regulatory human review.

Uncertain evidence and records flagged for review must remain subject to human assessment.

### 13. Limitations
- The current verification uses the repository's supplied test dataset.
- Some records explicitly describe fictional populations or test scenarios.
- A VALID result indicates that the implemented validation checks passed; it does not establish clinical truth or drug safety.
- The observed test results do not establish production deployment or live external-source integration.

---
End of document.
