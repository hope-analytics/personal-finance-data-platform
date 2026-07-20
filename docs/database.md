# Database Architecture

## Purpose

This document defines the database architecture of the Personal Finance Data Platform. It establishes the responsibilities of each database layer, the movement of data throughout the ELT pipeline, and the architectural principles that govern database design.

This document is implementation-independent. It describes the logical design of the database rather than the physical implementation.

Only approved architectural decisions are documented here.

---

# Database Philosophy

The database is designed around the principle that each layer has a single responsibility. Data progresses through the platform in one direction, becoming increasingly standardized and optimized for analytics while preserving the integrity of the original source data.

The platform follows an Extract–Load–Transform (ELT) architecture.

Source artifact are first parsed into structured records before being loaded into the Raw layer. Business transformations occur only after the Raw layer has been populated.

The database is designed to provide:

* Complete auditability
* Full data lineage
* Reproducible processing
* Separation of concerns
* Analytics-ready dimensional models

---

# Database Schemas

The platform is organized into five logical schemas.

| Schema    | Responsibility                                                              |
| --------- | --------------------------------------------------------------------------- |
| metadata  | Operational metadata, execution history, pipeline monitoring, and auditing. |
| config    | Pipeline configuration and future business rule configuration.              |
| raw       | Immutable storage of structured data extracted from source artifact.       |
| staging   | Standardized, validated, and enriched business data.                        |
| warehouse | Dimensional models optimized for reporting and analytics.                   |

Each schema owns a single responsibility and must not perform functions that belong to another layer.

---

# ELT Data Flow

```
Source Artifact
        │
        ▼
Artifact Parsing
        │
        ▼
Raw Layer
        │
        ▼
Staging Layer
        │
        ▼
Warehouse Layer
        │
        ▼
Power BI
```

Artifact Parsing converts unstructured source artifact into structured records.

The Raw layer stores those records exactly as extracted.

Business transformations occur in the Staging layer.

The Warehouse layer models standardized business data for reporting.

---

# Layer Responsibilities

## Artifact Parsing

Purpose

Convert supported source artifacts into structured transaction records.

Responsibilities

* Extract transaction data
* Preserve source values
* Produce standardized parser output

The parser does not perform business transformations.

---

## Raw Layer

Purpose

Persist the first accepted structured representation of source artifacts produced by the ingestion pipeline.

Responsibilities

* Preserve extracted values
* Preserve auditability
* Preserve lineage
* Support reproducibility

Business transformations are prohibited.

---

## Staging Layer

Purpose

Convert raw transactional data into a standardized business dataset.

Responsibilities

* Merchant standardization
* Data normalization
* Business rule application
* Operational enrichment

The Staging layer prepares data for analytical modeling.

---

## Warehouse Layer

Purpose

Represent standardized business information using dimensional models optimized for analytical workloads.

Responsibilities

* Fact tables
* Dimension tables
* Surrogate key management
* Historical analytics

The Warehouse layer assumes business standardization has already been completed.

---

# Data Ownership

Every dataset has exactly one producer.

| Layer            | Producer                |
| ---------------- | ----------------------- |
| Artifact Parsing | Parser                  |
| Raw              | Ingestion Pipeline      |
| Staging          | Transformation Pipeline |
| Warehouse        | Warehouse Load Pipeline |

Ownership is exclusive.

Downstream layers consume upstream data but never modify it.

---

# Raw Layer Contract

The Raw layer serves as the permanent landing layer for structured financial transactions extracted from supported source artifact.

## Purpose

Preserve the extracted source data exactly as produced by the parser while maintaining complete auditability and reproducibility.

## Grain

One row represents one financial transaction extracted from one source artifact.

## Allowed Data

The Raw layer may contain:

* Business attributes extracted directly from the source artifact.
* Technical metadata required for ingestion, auditing, and lineage.

## Forbidden Data

The Raw layer must not contain:

* Business standardization
* Merchant normalization
* Category assignment
* Derived analytical values
* Reporting calculations
* Business enrichment

## Ownership

Owned exclusively by the Ingestion Pipeline.

## Immutability

Once successfully loaded, extracted source values are never modified.

Corrections are introduced through reprocessing rather than updates.

## Retention

Raw data is retained indefinitely to preserve auditability, reproducibility, and historical processing records.

---

# Reprocessing Strategy

Reprocessing never modifies existing Raw records.

When a source artifact is reprocessed:

* A new pipeline execution is created.
* A new Raw dataset is generated.
* Previous processing history is preserved.
* Downstream processing consumes the authoritative pipeline execution.

This strategy preserves historical traceability while allowing parser improvements to be applied safely.

## Source Artifact Lifecycle

A source artifact is registered immediately when it is discovered by the ingestion pipeline, before parsing or validation begins.

The purpose of registration is to establish a complete operational audit trail of every artifact encountered by the platform, regardless of its eventual processing outcome.

A registered source artifact remains part of the platform's history whether processing:

* completes successfully,
* fails during parsing,
* is rejected during validation, or
* is intentionally reprocessed.

Reprocessing does not create a new source artifact. Instead, it creates a new pipeline execution associated with the existing source artifact.

The platform prevents duplicate source artifact registration according to its artifact uniqueness policy. The logical architecture requires duplicate registrations to be rejected; the physical implementation of uniqueness (such as filename, file hash, or a combination of both) will be determined during the physical database design.

---

# Data Lineage

Every analytical record must be traceable through the complete ELT pipeline.

```
Source Artifact
        │
        ▼
Pipeline Execution
        │
        ▼
Raw Record
        │
        ▼
Staging Record
        │
        ▼
Warehouse Record
```

Lineage must remain intact throughout the lifecycle of every transaction.

---

# Reproducibility

The platform must be capable of reproducing identical Raw datasets when provided with:

* The same source artifact.
* The same parser version
* The same pipeline configuration

Parser improvements generate new datasets rather than modifying historical ones.

---

# Design Principles

Every database object must satisfy the following principles.

* Single Responsibility
* One-Way Data Flow
* Immutable Raw Layer
* Complete Data Lineage
* Reproducibility
* Exclusive Dataset Ownership
* Documentation Before Implementation
* Data Contracts Before Physical Design

## Business Uniqueness

The Raw layer does not infer the business uniqueness of financial transactions unless the originating source artifact provides a stable business identifier. When the source cannot distinguish between repeated transactions and duplicate records, the platform preserves all transactions exactly as received.

---

# Current Scope

This document currently defines the approved logical architecture of the Personal Finance Data Platform database.

The approved scope includes:

* Database philosophy
* Database schemas
* ELT architecture
* Layer responsibilities
* Dataset ownership
* Raw layer contract
* Reprocessing strategy
* Data lineage
* Reproducibility
* Metadata Domain logical architecture
  * source_artifact
  * pipeline_run
  * failed_record

The Metadata Domain Logical Model is complete and serves as the authoritative source for the Physical Database Design.

The following topics remain outside the current scope and will be documented after their respective architecture reviews:

* Physical database design
* SQL implementation
* Index strategy
* Constraints
* Performance optimization
* Database deployment

---

# Metadata Domain

## Overview

The Metadata Domain is responsible for capturing the operational information required to monitor, audit, reproduce, and troubleshoot the ingestion platform.

The Metadata Domain owns operational facts about the ingestion platform. It does not own financial transactions. Instead, it owns operational evidence describing source artifacts, pipeline executions, and validation outcomes. Business transaction data belongs exclusively to the business schemas.

This section documents the approved Logical Metadata Model. It defines logical responsibilities and information ownership only. Physical tables, columns, constraints, and implementation details are introduced during the Physical Database Design phase.

The Metadata Domain consists of three logical entities:

* **source_artifact** – Represents the identity and lifecycle of every discovered source artifact.
* **pipeline_run** – Represents the operational history of each pipeline execution performed against a source artifact.
* **failed_record** – Represents immutable record-level validation evidence for structured records rejected during Dataset Validation.

The Metadata Domain follows these principles:

* Every operational concept has exactly one owner.
* Historical evidence is immutable.
* Relationships establish aggregate boundaries and lineage, while information ownership remains with the owning entity.
* Primary facts are persisted; calculated facts are derived.
* Every logical attribute is derived from an approved information group.
* The Logical Metadata Model is the authoritative source for the physical database design.

## Information Groups

Information Groups organize the logical facts owned by an entity.

They define logical ownership before physical implementation.

Every logical attribute belongs to exactly one Information Group.

Physical database columns are derived exclusively from approved logical attributes within these Information Groups.

Information Groups provide the bridge between an entity's logical responsibility and its physical database implementation.

## Physical Design Principles

The implementation follows the following principles:

- The pipeline owns all operational values.
- PostgreSQL owns structural integrity.
- The database does not generate operational state through default values.
- Every physical column is derived from an approved logical attribute.
- Every constraint exists to protect structural integrity rather than enforce pipeline business rules.
- Indexes exist only for approved access patterns.

### Aggregate Identity

When an entity's approved grain depends upon membership within an aggregate, the aggregate identifier shall be materialized in the physical model.

Materializing aggregate identity allows every persisted row to explicitly identify the business object to which it belongs, improving traceability, operational debugging, and lineage.

Aggregate identity is distinct from execution lineage. Multiple foreign keys may therefore coexist when they represent different architectural responsibilities rather than duplicated information.

---

# source_artifact

## Logical Design

### Purpose

Represents the authoritative identity and operational lifecycle of one discovered source artifact within the platform.

### Grain

One discovered source artifact.

### Producer

Artifact Discovery

### Persister

Pipeline

### Consumers

* Pipeline Execution
* Monitoring
* Reprocessing
* Operational Reporting

### Information Groups

#### Artifact Identity

Owns:

* Artifact Identifier
* Original File Name
* Source System
* File Hash *(future capability)*

#### Lifecycle

Owns:

* Current Lifecycle State

#### Operational Audit

Owns:

* Discovery Timestamp

### Relationships

* One `source_artifact` may participate in many `pipeline_run` executions.
* One `source_artifact` aggregates: Financial Transactions and Failed Records
* Lineage to failed validation evidence is provided through `pipeline_run`.

## Physical Implementation

The physical implementation of `metadata.source_artifact` is derived directly from the approved Logical Metadata Model.

The physical schema preserves the approved grain, ownership, and responsibilities established during the Conceptual and Logical Design phases. No physical attribute exists without an approved logical attribute.

### Physical Columns

| Column | Data Type | Constraints | Notes |
|----------|-----------|-------------|------|
| artifact_id | BIGINT GENERATED ALWAYS AS IDENTITY | PRIMARY KEY | Platform-generated surrogate identifier. |
| original_file_name | TEXT | NOT NULL | Original filename supplied by the pipeline. |
| source_system | BIGINT | NOT NULL, FOREIGN KEY | References the approved source system in the Configuration domain. |
| lifecycle_status | ENUM | NOT NULL | Current lifecycle state supplied by the pipeline. |
| discovered_at | TIMESTAMP WITH TIME ZONE | NOT NULL | Timestamp when the pipeline first discovered the artifact. |

### Deferred Attributes

The following logical attributes are approved but intentionally deferred until the corresponding platform capability is implemented.

| Attribute | Reason |
|-----------|--------|
| file_hash | File hash validation has not yet been introduced into the ingestion pipeline. The physical column will be added when content-based uniqueness becomes part of the approved architecture. |

### Index Strategy

| Column | Index | Justification |
|----------|-------|---------------|
| artifact_id | Primary Key | Entity identity and joins. |
| original_file_name | Yes | High-frequency pipeline lookup for duplicate filename validation. |
| lifecycle_status | Yes | Operational monitoring and troubleshooting of exceptional lifecycle states. |

Indexes are created only where an approved operational access pattern exists.

No indexes are created for speculative future workloads.

---

# pipeline_run

## Logical Design

### Purpose

Represents the authoritative operational history of one pipeline execution performed against one discovered source artifact.

### Grain

One execution against one source artifact.

### Producer

Pipeline Orchestrator

### Persister

Pipeline

### Consumers

* Operational Monitoring
* Troubleshooting
* Reprocessing
* Audit
* Metadata Lineage

### Information Groups

#### Execution Identity

Owns:

* Pipeline Run Identifier
* Source Artifact Identifier
* Execution Sequence

#### Execution Outcome

Owns:

* Execution Status
* Failure Stage
* Failure Code
* Primary Error Message

#### Execution Metrics

Owns:

* Start Timestamp
* End Timestamp

Execution Duration is derived and is not persisted.

#### Execution Context

Owns:

* Parser
* Parser Version

#### Execution Summary

Owns:

* Total Records
* Successful Records
* Failed Records

Rows Committed is a derived value and is not persisted.

### Relationships

* Each `pipeline_run` belongs to one `source_artifact`.
* One `pipeline_run` may produce many `failed_record` entries.

## Physical Implementation

The physical implementation of `metadata.pipeline_run` is derived directly from the approved Logical Metadata Model.

The table represents the authoritative operational history of every pipeline execution performed against a discovered source artifact.

Each execution is immutable and captures the execution context, outcome, metrics, and summary produced by the pipeline.

### Physical Columns

| Column | Data Type | Constraints | Notes |
|----------|-----------|-------------|------|
| pipeline_run_id | BIGINT GENERATED ALWAYS AS IDENTITY | PRIMARY KEY | Platform-generated surrogate identifier. |
| artifact_id | BIGINT | NOT NULL, FOREIGN KEY | References the source artifact processed during the execution. |
| execution_sequence | INTEGER | NOT NULL | Pipeline-generated execution number for the associated source artifact. |
| execution_status | ENUM | NOT NULL | Final execution outcome supplied by the pipeline. |
| failure_stage | TEXT | NULL | Pipeline stage where execution terminated. NULL when execution succeeds. |
| failure_code | TEXT | NULL | Stable machine-readable failure classification. NULL when execution succeeds. |
| primary_error_message | TEXT | NULL | Human-readable summary of the primary execution failure. NULL when execution succeeds. |
| start_timestamp | TIMESTAMP WITH TIME ZONE | NOT NULL | Timestamp when execution started. |
| end_timestamp | TIMESTAMP WITH TIME ZONE | NULL | Timestamp when execution completed. NULL while execution is in progress. |
| parser_identifier | TEXT | NOT NULL | Unique parser identifier selected by the pipeline (for example, `bpi_statement_parser-1.0.0`). |
| total_records | INTEGER | NOT NULL | Total structured records produced by the parser. |
| failed_records | INTEGER | NOT NULL | Number of structured records rejected during Dataset Validation. |
| successful_records | INTEGER | NOT NULL | Number of structured records accepted into the Raw layer. |

### Constraint Strategy

The implementation follows the approved Metadata Physical Design principles.

PostgreSQL enforces structural integrity only.

The pipeline owns all execution state and operational decisions.

The following constraints are implemented:

| Constraint | Purpose |
|------------|---------|
| Primary Key (`pipeline_run_id`) | Platform identity |
| Foreign Key (`artifact_id`) | Preserves lineage to the processed source artifact |
| UNIQUE (`artifact_id`, `execution_sequence`) | Guarantees execution sequence uniqueness for each source artifact |

Foreign key behavior:

| Action | Strategy |
|--------|----------|
| ON DELETE | RESTRICT |
| ON UPDATE | RESTRICT |

Historical execution records must never become orphaned through parent deletion or identity changes.

### Index Strategy

| Column | Index | Justification |
|----------|-------|---------------|
| pipeline_run_id | Primary Key | Entity identity and joins. |
| artifact_id | Yes | High-frequency lineage lookup and reprocessing history. |
| execution_status | Yes | Operational monitoring and troubleshooting. |
| failure_code | Yes | Efficient investigation of execution failures. |
| start_timestamp | Yes | Operational history ordered by execution time. |
| (`artifact_id`, `execution_sequence`) | UNIQUE Composite Index | Enforces execution numbering and supports execution history retrieval. |

Indexes are introduced only for approved operational access patterns.

No speculative indexes are created.

### Design Rationale

`pipeline_run` represents immutable operational execution history.

Consequently:

* Each execution is preserved permanently.
* Historical executions are never updated after completion.
* Execution context is owned exclusively by the pipeline.
* Relationships provide lineage rather than ownership.
* Downstream processing determines which execution is authoritative without modifying historical execution records.

---

# failed_record

## Logical Design

### Purpose

Represents immutable record-level validation evidence for one structured record rejected during Dataset Validation during a single pipeline execution.

### Grain

One structured record.

### Producer

Dataset Validation

### Persister

Pipeline

### Consumers

* Debugging
* Parser Improvement
* Operational Learning
* Audit
* Reprocessing

### Information Groups

#### Structured Record Snapshot

Owns the parser contract output evaluated by Dataset Validation.

The snapshot mirrors the parser contract that existed at the time of execution.

Historical snapshots are never rewritten when the parser contract evolves.

#### Validation Evidence

Owns:

* Validation Rule
* Failure Code
* Primary Validation Message

Only the primary validation failure is recorded for each failed record.

#### Record Context

Owns:

* Structured Record Reference

The reference identifies the structured record within the parser output.

Debugging is performed against the structured dataset rather than the original source artifact.

### Relationships

* Each `failed_record` participates in exactly one `source_artifact` aggregate.
* Each `failed_record` belongs to exactly one `pipeline_run`.
* Source Artifact defines the immutable origin of the structured record.
* Pipeline Run defines the execution during which the record was evaluated and rejected.
* Information ownership remains with `failed_record`; relationships establish aggregate boundaries and execution lineage only.

## Physical Implementation

The physical implementation of `metadata.failed_record` is derived directly from the approved Logical Metadata Model.

The table preserves immutable record-level validation evidence produced during Dataset Validation.

Every physical column is derived from an approved logical attribute.

Historical evidence is preserved exactly as observed during execution and is never rewritten by future parser improvements or historical corrections.

### Physical Columns

| Column | Data Type | Constraints | Notes |
|----------|-----------|-------------|------|
| failed_record_id | BIGINT GENERATED ALWAYS AS IDENTITY | PRIMARY KEY | Platform-generated surrogate identifier. |
| artifact_id | BIGINT | NOT NULL, FOREIGN KEY | References the immutable Source Artifact to which the structured record belongs. Materializes aggregate identity defined by the approved grain. |
| pipeline_run_id | BIGINT | NOT NULL, FOREIGN KEY | References the pipeline execution that evaluated the structured record and produced the validation evidence. |
| transaction_date | DATE | NULL | Parser-produced transaction date preserved exactly as observed. NULL is permitted because Dataset Validation—not PostgreSQL—determines business validity. |
| transaction_description | TEXT | NULL | Parser-produced transaction description preserved exactly as observed. |
| transaction_amount | NUMERIC(18,2) | NULL | Parser-produced monetary value preserved exactly as observed, including its original sign. |
| validation_rule | TEXT | NOT NULL | Stable machine-readable validation rule identifier. |
| failure_code | TEXT | NOT NULL | Stable machine-readable failure classification. |
| primary_validation_message | TEXT | NOT NULL | Human-readable explanation of the primary validation failure. |
| structured_record_reference | BIGINT | NOT NULL | Pipeline-generated identifier representing the structured record within a single pipeline execution. |

### Constraint Strategy

The implementation follows the approved Metadata Physical Design principles.

PostgreSQL enforces structural integrity only.

The pipeline owns all operational state and business validation.

Business validation failures are intentionally preserved rather than rejected by database constraints.

The following constraints are implemented:

| Constraint | Purpose |
|------------|---------|
| Primary Key (`failed_record_id`) | Platform identity |
| Foreign Key (`artifact_id`) | Preserve aggregate identity to the immutable Source Artifact |
| Foreign Key (`pipeline_run_id`) | Preserves execution lineage |
| UNIQUE (`pipeline_run_id`, `structured_record_reference`) | Guarantees execution-local uniqueness of structured records |

Foreign key behavior:

| Action | Strategy |
|--------|----------|
| ON DELETE | RESTRICT |
| ON UPDATE | RESTRICT |

Historical validation evidence must never become orphaned through parent deletion or identity changes.

### Index Strategy

| Column | Index | Justification |
|----------|-------|---------------|
| `failed_record_id` | Primary Key | Entity identity and joins. |
| `artifact_id` | Yes | Aggregate-level debugging, lineage, and artifact-based investigation |
| (`pipeline_run_id`, `structured_record_reference`) | UNIQUE Composite Index | Supports execution-local record lookup while enforcing uniqueness. |

No additional indexes are currently introduced.

Indexes are created only for approved operational access patterns.

Additional indexes may be introduced after production workload analysis demonstrates a measurable operational benefit.

### Design Rationale

`failed_record` represents immutable validation evidence for a structured record.

Consequently:

* Aggregate identity belongs to `source_artifact`.
* Execution context belongs to `pipeline_run`.
* Business validation belongs to Dataset Validation.
* PostgreSQL preserves historical evidence without enforcing business validation.
* Structured transaction values are preserved exactly as produced by the parser.
* Future parser improvements create new validation evidence rather than modifying historical records.

The physical model intentionally materializes both aggregate identity (`artifact_id`) and execution lineage (`pipeline_run_id`) because they answer different architectural questions.

Aggregate identity identifies the business object to which the record belongs.

Execution lineage identifies the pipeline execution that produced the validation evidence.

# Business Domain

## Overview

The Business Domain is responsible for preserving and transforming financial information throughout the ELT platform.

Unlike the Metadata Domain, which owns operational evidence about pipeline execution, the Business Domain owns structured representations of financial information produced and consumed by the platform.

Business entities progress through successive processing layers while preserving auditability, lineage, and reproducibility.

The Business Domain currently consists of:

* **raw.financial_transaction** – Represents the first accepted structured representation of financial transactions extracted from source artifacts.

Additional entities will be documented as the platform evolves.

---

# raw.financial_transaction

## Logical Design

### Purpose

Represents the first accepted structured representation of one financial transaction extracted from a source artifact.

The entity preserves the parser-produced representation exactly as accepted by the ingestion pipeline.

It does not represent the financial event itself or the source artifact from which it originated.

### Grain

One row represents one accepted structured representation of one financial transaction extracted from one source artifact.

### Producer

Ingestion Pipeline

### Persister

Pipeline

### Consumers

* Staging Transformation
* Reprocessing
* Audit

### Information Groups

Owns the accepted structured representation of the financial transaction.

Owns:

* Transaction Date
* Transaction Description
* Transaction Amount

### Relationships

* Each `raw.financial_transaction` participates in exactly one `source_artifact` aggregate.
* Each `raw.financial_transaction` belongs to exactly one `pipeline_run`.
* Source Artifact defines the immutable business origin of the accepted structured representation.
* Pipeline Run defines the execution that accepted and persisted the structured representation.
* Information ownership remains with `raw.financial_transaction`; relationships establish aggregate boundaries and execution lineage only.

## Physical Implementation

The physical implementation of `raw.financial_transaction` is derived directly from the approved Business Domain Logical Model.

The table preserves the first accepted structured representation of financial transactions within the platform.

Every physical column is derived from an approved logical attribute.

The implementation maintains the approved grain, ownership, and architectural responsibilities established during the Conceptual and Logical Design phases.

### Physical Columns

| Column                  | Data Type                           | Constraints           | Notes                                                               |
| ----------------------- | ----------------------------------- | --------------------- | ------------------------------------------------------------------- |
| raw_transaction_id      | BIGINT GENERATED ALWAYS AS IDENTITY | PRIMARY KEY           | Platform surrogate identifier.                                      |
| artifact_id             | BIGINT                              | NOT NULL, FOREIGN KEY | Aggregate identity.                                                 |
| pipeline_run_id         | BIGINT                              | NOT NULL, FOREIGN KEY | Execution lineage.                                                  |
| transaction_date        | DATE                                | NOT NULL              | Accepted transaction date exactly as produced by the parser.        |
| transaction_description | TEXT                                | NOT NULL              | Accepted transaction description exactly as produced by the parser. |
| transaction_amount      | NUMERIC(18,2)                       | NOT NULL              | Accepted monetary value including original sign.                    |

### Constraint Strategy

The implementation follows the approved Business Domain and Metadata Physical Design principles.

PostgreSQL enforces structural integrity only.

The pipeline owns all parser execution, Dataset Validation, and business validation.

The following constraints are implemented:

| Constraint | Purpose |
|------------|---------|
| Primary Key (`raw_transaction_id`) | Platform identity |
| Foreign Key (`artifact_id`) | Preserves aggregate identity to the immutable Source Artifact |
| Foreign Key (`pipeline_run_id`) | Preserves execution lineage |

Foreign key behavior:

| Action | Strategy |
|--------|----------|
| ON DELETE | RESTRICT |
| ON UPDATE | RESTRICT |

Historical financial transactions must never become orphaned through parent deletion or identity changes.

No UNIQUE constraints are introduced because the Raw layer does not infer business uniqueness beyond the originating source artifact.

No CHECK constraints are introduced because business validation belongs to Dataset Validation rather than PostgreSQL.

### Index Strategy

| Column | Index | Justification |
|----------|-------|---------------|
| `raw_transaction_id` | Primary Key | Entity identity and joins. |
| `artifact_id` | Yes | Aggregate-level lineage, audit, and reprocessing. |
| `pipeline_run_id` | Yes | Execution lineage and downstream transformation. |

No additional indexes are currently introduced.

Indexes are created only for approved operational access patterns.

Additional indexes may be introduced after production workload analysis demonstrates a measurable operational benefit.

### Design Rationale

`raw.financial_transaction` represents the first accepted structured representation of a financial transaction within the Business Domain.

The entity preserves the parser-produced representation exactly as accepted by the ingestion pipeline and serves as the immutable foundation for all downstream business transformations.

Consequently:

* Aggregate identity belongs to `source_artifact`.
* Execution lineage belongs to `pipeline_run`.
* Business validation belongs to Dataset Validation.
* PostgreSQL enforces structural integrity without performing business validation.
* Structured transaction values are preserved exactly as accepted from the parser.
* Business uniqueness is intentionally not inferred beyond the originating source artifact.
* Historical transactions are never modified. Reprocessing produces new Raw datasets through new pipeline executions rather than updating existing records.

The physical model intentionally materializes both aggregate identity (`artifact_id`) and execution lineage (`pipeline_run_id`) because they represent different architectural responsibilities.

Aggregate identity identifies the immutable source artifact from which the accepted transaction originated.

Execution lineage identifies the pipeline execution that accepted and persisted the transaction into the Raw layer.

# Glossary

* **Source Artifact** - Any externally produced input accepted by the ingestion pipeline, such as PDF, XLSX, or future supported formats.


