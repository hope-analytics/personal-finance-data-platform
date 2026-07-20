# Project Blueprint

## Purpose

This document defines the current approved architecture of the Personal Finance Data Platform. It serves as the single source of truth for the project's architecture, engineering principles, and system responsibilities.

Only approved architectural decisions are documented here. Proposed enhancements, future improvements, and implementation plans are documented separately and are not included until they have been formally approved.

---

# Project Overview

The Personal Finance Data Platform is an end-to-end ELT data platform designed to ingest financial statements from multiple financial institutions, preserve the original transactional data, standardize business information, model analytical datasets, and deliver reporting through Power BI.

The platform is designed around the following engineering goals:

* Preserve data integrity.
* Maintain complete data lineage.
* Support reproducible processing.
* Separate responsibilities across architectural layers.
* Produce analytics from a dimensional warehouse rather than directly from operational datasets.

---

# High-Level Architecture

```
Source Artifacts
(PDF, XLSX, CSV)

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

The architecture follows an Extract–Load–Transform (ELT) approach. Source artifacts are parsed into structured records before being loaded into the Raw layer. Business transformations occur after the Raw layer has been populated.

---

# Current Database Schemas

The platform currently consists of five logical schemas.

| Schema    | Responsibility                                                                            |
| --------- | ----------------------------------------------------------------------------------------- |
| metadata  | Stores operational metadata, execution history, pipeline tracking, and audit information. |
| config    | Stores pipeline configuration and future business rule configuration.                     |
| raw       | Stores the first structured representation of extracted source data.                      |
| staging   | Stores standardized and validated business data prepared for analytical modeling.         |
| warehouse | Stores dimensional models optimized for reporting and analytics.                          |

---

# Layer Responsibilities

## Artifact Parsing

Responsible for extracting structured records from supported source artifacts.

The parser does not perform business transformations and does not communicate directly with downstream analytical layers.

---

## Raw Layer

Responsible for preserving extracted business data exactly as produced by the parser.

The Raw layer represents the first structured representation of the source artifact and acts as the permanent audit layer for all downstream processing.

Business transformations are not permitted within this layer.

---

## Staging Layer

Responsible for transforming accepted Raw datasets into standardized business datasets through standardization, normalization, business rule application, and operational enrichment.

Business rules belong exclusively to this layer.

---

## Warehouse Layer

Responsible for organizing business data into dimensional models optimized for reporting and analytical workloads.

The Warehouse layer assumes that all business standardization has already been completed.

---

## Power BI

Responsible for semantic modeling, visualization, and presentation-layer calculations.

Business cleansing and operational transformations must not occur within Power BI.

---

# Architectural Principles

The following principles govern all architectural decisions within the platform.

## Single Responsibility

Every architectural layer owns one responsibility and performs one primary function.

---

## One-Way Data Flow

Data moves in one direction only.

```
Source

↓

Raw

↓

Staging

↓

Warehouse

↓

Power BI
```

No downstream layer writes back into an upstream layer.

---

## Immutable Raw Layer

Once records have been successfully loaded into the Raw layer, the extracted source values are never modified.

Corrections and improvements are achieved through reprocessing rather than updates.

---

## Data Lineage

Every analytical record must be traceable back to:

* Pipeline execution
* Source artifact
* Raw record

Complete lineage is maintained throughout the platform.

---

## Reproducibility

Given the same source artifact, parser version, and pipeline configuration, the platform must be capable of reproducing the same Raw dataset.

Parser improvements generate new pipeline executions rather than modifying existing Raw datasets.

---

## Dataset Ownership

Each architectural layer has exactly one producer.

| Layer            | Owner                   |
| ---------------- | ----------------------- |
| Artifact Parsing | Parser                  |
| Raw              | Ingestion Pipeline      |
| Staging          | Transformation Pipeline |
| Warehouse        | Warehouse Load Pipeline |
| Power BI         | Semantic Model          |

Ownership is exclusive. Downstream layers consume data but do not modify datasets owned by upstream layers.

---

# Repository Standards

The repository follows a documentation-first approach.

Architectural decisions are documented before implementation.

Implementation must conform to approved documentation rather than define it.

---

# Documentation Standards

Project documentation is organized according to responsibility.

| Document             | Responsibility                                   |
| -------------------- | ------------------------------------------------ |
| project-blueprint.md | Current approved architecture and project state. |
| contracts.md | Defines the Platform Contracts that govern the Personal Finance Data Platform. Platform Contracts establish authoritative architectural responsibilities and minimum platform representations that all architectural components and implementations must satisfy. |
| architecture.md      | System architecture and component interactions.  |
| database.md          | Database architecture and data contracts.        |
| development.md       | Engineering standards and development workflow.  |
| roadmap.md           | Approved implementation roadmap.                 |
| ADRs                 | Permanent architectural decisions.               |
| ARRs                 | Architecture review history.                     |

---

# Current Project Status

Current Milestone:

**Milestone 2 — Database Design**

Completed:

* Repository foundation
* Git workflow
* Project structure
* ELT architecture
* Layer responsibilities
* Database schema definition
* Core architectural principles

In Progress:

* Logical data model
* Raw layer data contracts

Not Started:

* Physical database implementation
* ELT implementation
* Warehouse implementation
* Reporting implementation
