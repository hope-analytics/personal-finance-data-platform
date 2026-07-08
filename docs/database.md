# Database Architecture

## Purpose

This document defines the database architecture of the Personal Finance Data Platform. It establishes the responsibilities of each database layer, the movement of data throughout the ELT pipeline, and the architectural principles that govern database design.

This document is implementation-independent. It describes the logical design of the database rather than the physical implementation.

Only approved architectural decisions are documented here.

---

# Database Philosophy

The database is designed around the principle that each layer has a single responsibility. Data progresses through the platform in one direction, becoming increasingly standardized and optimized for analytics while preserving the integrity of the original source data.

The platform follows an Extract–Load–Transform (ELT) architecture.

Source documents are first parsed into structured records before being loaded into the Raw layer. Business transformations occur only after the Raw layer has been populated.

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
| raw       | Immutable storage of structured data extracted from source documents.       |
| staging   | Standardized, validated, and enriched business data.                        |
| warehouse | Dimensional models optimized for reporting and analytics.                   |

Each schema owns a single responsibility and must not perform functions that belong to another layer.

---

# ELT Data Flow

```
Source Documents
        │
        ▼
Document Parsing
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

Document Parsing converts unstructured source documents into structured records.

The Raw layer stores those records exactly as extracted.

Business transformations occur in the Staging layer.

The Warehouse layer models standardized business data for reporting.

---

# Layer Responsibilities

## Document Parsing

Purpose

Convert supported financial documents into structured transaction records.

Responsibilities

* Extract transaction data
* Preserve source values
* Produce standardized parser output

The parser does not perform business transformations.

---

## Raw Layer

Purpose

Persist the first structured representation of the extracted source data.

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

* Data validation
* Merchant standardization
* Data normalization
* Duplicate detection
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
| Document Parsing | Parser                  |
| Raw              | Ingestion Pipeline      |
| Staging          | Transformation Pipeline |
| Warehouse        | Warehouse Load Pipeline |

Ownership is exclusive.

Downstream layers consume upstream data but never modify it.

---

# Raw Layer Contract

The Raw layer serves as the permanent landing layer for structured financial transactions extracted from supported source documents.

## Purpose

Preserve the extracted source data exactly as produced by the parser while maintaining complete auditability and reproducibility.

## Grain

One row represents one financial transaction extracted from one source document.

## Allowed Data

The Raw layer may contain:

* Business attributes extracted directly from the source document.
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

When a source document is reprocessed:

* A new pipeline execution is created.
* A new Raw dataset is generated.
* Previous processing history is preserved.
* Downstream processing consumes the authoritative pipeline execution.

This strategy preserves historical traceability while allowing parser improvements to be applied safely.

---

# Data Lineage

Every analytical record must be traceable through the complete ELT pipeline.

```
Source Document
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

* The same source document
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

---

# Current Scope

This document currently defines:

* Database philosophy
* Database schemas
* ELT architecture
* Layer responsibilities
* Dataset ownership
* Raw layer contract
* Reprocessing strategy
* Data lineage
* Reproducibility

Logical table definitions, physical database design, and SQL implementation will be documented after their respective design reviews have been completed.
