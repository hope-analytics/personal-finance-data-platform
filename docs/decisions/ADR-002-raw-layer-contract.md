# ADR-002: Raw Layer Contract

## Status

Accepted

---

## Date

2026-07-08

---

## Context

The Raw layer is the foundation of the Personal Finance Data Platform.

It serves as the permanent landing layer for structured financial transactions extracted from supported source documents.

Without a clearly defined contract, business transformations can gradually migrate into the Raw layer, reducing auditability, complicating debugging, and making historical reprocessing unreliable.

To prevent architectural drift, the responsibilities and constraints of the Raw layer are formally defined in this decision.

---

# Decision

The Raw layer shall preserve the first structured representation of financial transactions exactly as produced by the document parser.

The Raw layer is immutable and serves as the authoritative audit record for all downstream processing.

Business transformations are prohibited within the Raw layer.

---

# Purpose

The purpose of the Raw layer is to preserve extracted source data while maintaining complete auditability, reproducibility, and data lineage.

The Raw layer represents the earliest structured dataset within the platform.

---

# Grain

One row represents one financial transaction extracted from one source document.

Each transaction is stored independently.

---

# Ownership

The Raw layer is owned exclusively by the Ingestion Pipeline.

The parser produces structured transaction records but does not write directly to the database.

Downstream layers consume Raw data but never modify it.

---

# Allowed Data

The Raw layer may contain:

* Business attributes extracted directly from the source document.
* Technical metadata required for ingestion.
* Technical metadata required for auditing.
* Technical metadata required for data lineage.

Technical metadata supports operational management without changing the business meaning of extracted data.

---

# Forbidden Data

The Raw layer must not contain:

* Business standardization
* Merchant normalization
* Category assignment
* Derived analytical attributes
* Reporting calculations
* Business enrichment
* Data produced by downstream transformations

These responsibilities belong to later processing layers.

---

# Immutability

Once successfully loaded, extracted source values must never be modified.

Parser improvements, business rule changes, and transformation corrections must never update historical Raw records.

Historical datasets are preserved permanently.

---

# Retention

Raw data is retained indefinitely.

The Raw layer serves as the permanent audit history of all processed financial transactions.

Retention enables:

* Historical verification
* Debugging
* Auditability
* Reprocessing
* Reproducibility

---

# Reprocessing

Reprocessing creates a new pipeline execution.

Historical Raw datasets remain unchanged.

Each pipeline execution produces an independent Raw dataset.

Downstream processing consumes the authoritative pipeline execution designated by the metadata layer.

---

# Data Lineage

Every Raw record must be traceable to:

* Source document
* Parser
* Pipeline execution

The Raw layer forms the beginning of the platform's end-to-end lineage.

---

# Reproducibility

Given:

* The same source document
* The same parser version
* The same pipeline configuration

The platform must be capable of reproducing an identical Raw dataset.

Parser improvements generate new Raw datasets rather than modifying historical ones.

---

# Rationale

Separating extraction from business transformation provides several architectural advantages.

* Preserves the original business representation.
* Enables complete auditability.
* Supports deterministic processing.
* Simplifies debugging.
* Allows safe parser evolution.
* Preserves historical processing records.

The Raw layer therefore becomes the immutable foundation upon which every downstream transformation is built.

---

# Consequences

## Positive

* Complete audit trail
* Strong data lineage
* Reliable reprocessing
* Deterministic processing
* Simplified debugging
* Independent evolution of transformation logic

## Trade-offs

* Increased storage requirements
* Multiple historical pipeline executions may exist simultaneously
* Additional metadata management is required to identify the authoritative pipeline execution

---

# Business Identity

The Raw layer preserves transactions exactly as represented by the source artifact. It does not infer or construct business uniqueness when the source does not provide sufficient information to do so. Technical row identity and business identity are intentionally treated as separate concepts.

---

# Related Documents

* `docs/project-blueprint.md`
* `docs/database.md`
* `ADR-001: Layer Responsibilities`

---

# Notes

Any proposal to introduce business transformations into the Raw layer must be considered an architectural change and requires a new Architecture Decision Record.
