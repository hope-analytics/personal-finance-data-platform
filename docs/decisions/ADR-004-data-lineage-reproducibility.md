# ADR-004: Data Lineage and Reproducibility

## Status

Accepted

---

## Date

2026-07-08

---

# Context

The Personal Finance Data Platform transforms financial transactions through multiple processing layers before they become analytical datasets.

As the platform evolves, parser improvements, transformation enhancements, and warehouse changes will occur independently.

Without complete lineage and reproducibility, it becomes difficult to:

* Explain how an analytical record was produced.
* Validate historical processing.
* Investigate data quality issues.
* Reproduce historical results.
* Audit changes throughout the ELT pipeline.

The platform therefore requires a formal strategy for preserving traceability across every processing stage.

---

# Decision

The platform adopts end-to-end data lineage and deterministic processing as fundamental architectural principles.

Every dataset must be traceable to its immediate upstream dataset, ultimately leading back to the original source document.

Processing must be reproducible when executed using the same:

* Source document
* Parser version
* Pipeline configuration

---

# Data Lineage

Every transaction follows a complete lineage throughout the platform.

```text
Source Document
        │
        ▼
Pipeline Execution
        │
        ▼
Raw Dataset
        │
        ▼
Staging Dataset
        │
        ▼
Warehouse Dataset
        │
        ▼
Power BI
```

Each downstream dataset maintains a relationship to its upstream source.

No processing stage may create records whose origin cannot be identified.

---

# Reproducibility

The platform must be capable of producing identical Raw datasets when provided with identical inputs.

Reproducibility requires:

* The same source document.
* The same parser version.
* The same pipeline configuration.

Changes to parsers or transformation logic create new processing executions rather than modifying historical datasets.

---

# Operational Principles

The platform shall:

* Preserve historical processing results.
* Preserve complete lineage across all layers.
* Support replay of historical processing.
* Enable comparison between processing executions.
* Prevent loss of historical audit information.

---

# Rationale

Complete lineage and reproducibility improve the platform by:

* Simplifying debugging.
* Supporting audit requirements.
* Increasing confidence in analytical outputs.
* Allowing safe parser evolution.
* Supporting long-term maintainability.

These capabilities are foundational for any platform that produces analytical datasets from operational source data.

---

# Consequences

## Positive

* End-to-end traceability.
* Reliable auditing.
* Repeatable processing.
* Easier root cause analysis.
* Improved confidence in reporting.

## Trade-offs

* Additional metadata must be maintained.
* Pipeline execution history grows over time.
* Metadata design becomes a critical part of the platform architecture.

---

# Related Documents

* `docs/project-blueprint.md`
* `docs/database.md`
* `ADR-001: Layer Responsibilities`
* `ADR-002: Raw Layer Contract`
* `ADR-003: Reprocessing Strategy`

---

# Notes

Data lineage is established through relationships between processing layers.

The physical implementation of lineage (identifiers, metadata tables, and relationships) will be defined during the Metadata schema design.

This ADR defines the architectural principle rather than the implementation.
