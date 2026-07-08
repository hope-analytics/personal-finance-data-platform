# ADR-001: Layer Responsibilities

## Status

Accepted

---

## Date

2026-07-08

---

## Context

The Personal Finance Data Platform consists of multiple processing stages that transform financial data from source documents into analytical datasets.

Without clearly defined ownership, business logic can become duplicated across components, resulting in inconsistent processing, difficult maintenance, and reduced traceability.

To establish a maintainable architecture, each processing layer must have a clearly defined responsibility and ownership.

---

# Decision

The platform adopts a layered architecture in which every layer has exactly one primary responsibility.

Business logic is implemented only within the layer that owns that responsibility.

Data flows in one direction through the platform.

No downstream layer may modify data owned by an upstream layer.

---

# Layer Responsibilities

## Document Parsing

**Responsibility**

Convert supported source documents into structured transaction records.

The parser extracts information from source documents without applying business transformations.

---

## Raw Layer

**Responsibility**

Persist the structured output produced by the parser.

The Raw layer preserves extracted source values and serves as the permanent audit layer for downstream processing.

Business transformations are prohibited.

---

## Staging Layer

**Responsibility**

Transform raw transactional data into standardized business data.

Responsibilities include:

* Data validation
* Standardization
* Normalization
* Business rule application
* Operational enrichment

---

## Warehouse Layer

**Responsibility**

Represent standardized business information using dimensional models optimized for analytical workloads.

The Warehouse layer assumes business standardization has already been completed.

---

## Power BI

**Responsibility**

Provide semantic modeling, visualization, and presentation-layer calculations.

Power BI consumes analytical datasets but does not perform operational data transformations.

---

# Rationale

Separating responsibilities provides several architectural benefits.

* Prevents duplicated business logic.
* Improves maintainability.
* Supports independent evolution of each processing layer.
* Simplifies debugging.
* Preserves complete data lineage.
* Enables deterministic processing.
* Reduces coupling between components.

This decision establishes clear ownership boundaries throughout the platform.

---

# Consequences

## Positive

* Clear ownership of business logic.
* Easier onboarding for future contributors.
* Consistent placement of transformations.
* Improved auditability.
* Stronger separation of concerns.

## Trade-offs

* Additional planning is required before implementation.
* New transformations must be evaluated to determine the correct owning layer.
* Architectural discipline must be maintained throughout development.

---

# Related Documents

* `docs/project-blueprint.md`
* `docs/database.md`

---

# Notes

This decision forms the architectural foundation for all future database, pipeline, and transformation design decisions.

Any future processing component introduced into the platform must have a clearly defined responsibility and ownership before implementation begins.
