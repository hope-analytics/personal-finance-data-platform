# ARR-003: Business Domain Physical Design Review

## Review Status

Approved

---

## Review Date

2026-07-20

---

# Review Objective

This architecture review validates the completed physical database design of the Business Domain.

The objective is to confirm that the physical implementation faithfully derives from the approved Conceptual Design, Platform Contracts, and Logical Model while preserving the architectural principles established for the Personal Finance Data Platform.

---

# Problem Statement

Following completion of the Metadata Domain, the Business Domain required a complete physical design before implementation could begin.

The review verifies that the Business Domain:

* Preserves approved business responsibilities.
* Maintains architectural consistency with the Metadata Domain.
* Separates business information from operational metadata.
* Produces a physical model that satisfies the approved Financial Transaction Contract.

---

# Scope Reviewed

## Business Domain

* Business Domain Overview
* raw.financial_transaction Logical Design
* Physical Columns
* Constraint Strategy
* Index Strategy
* Design Rationale

## Architectural Alignment

* Financial Transaction Contract
* Raw Layer Contract
* Metadata Physical Design Principles

---

# Alternatives Considered

## Introduce business uniqueness constraints

**Decision:** Rejected

### Reason

The Raw layer preserves transactions exactly as accepted from the parser.

When the originating source artifact does not provide sufficient information to distinguish duplicate records from legitimate repeated transactions, the platform intentionally preserves every accepted transaction.

Business uniqueness therefore remains undefined within the Raw layer.

---

## Enforce business validation using database constraints

**Decision:** Rejected

### Reason

Business validation belongs exclusively to Dataset Validation.

PostgreSQL protects structural integrity rather than business correctness.

---

## Persist transformed business values

**Decision:** Rejected

### Reason

Business transformations belong to the Staging Layer.

The Raw layer preserves the first accepted structured representation exactly as accepted by the ingestion pipeline.

---

# Approved Architectural Decisions

The following architectural decisions were validated during this review.

* Physical columns derive exclusively from approved logical attributes.
* Aggregate identity and execution lineage are materialized independently.
* PostgreSQL enforces structural integrity only.
* Business validation remains the responsibility of Dataset Validation.
* Business uniqueness is intentionally not inferred.
* The Raw layer remains immutable.
* Physical indexes exist only for approved operational access patterns.

---

# Risks Identified

No architectural risks were identified.

The remaining work consists of implementation activities rather than architectural design.

Future enhancements such as additional transaction attributes or performance tuning shall be evaluated through future architecture reviews as the platform evolves.

---

# Outcomes

The Business Domain Physical Design is approved.

The Metadata Domain and Business Domain now form the complete database foundation for the platform.

The approved database architecture now provides:

* Operational metadata design.
* Business data design.
* Platform contracts.
* Physical database specification.

Implementation may proceed only in accordance with the approved architecture.

---

# Next Architectural Objective

Begin Milestone 3 — Ingestion Layer.

The next architecture review will define the ingestion workflow responsible for:

* Source Artifact Discovery
* Artifact Registration
* Parser Execution
* Dataset Validation
* Pipeline Execution Lifecycle

before implementation begins.

---

# Review Summary

This review successfully completes the database design phase of the Personal Finance Data Platform.

The platform now possesses a complete architecture for both operational metadata and business data, establishing a stable foundation for ingestion pipeline design and subsequent implementation.