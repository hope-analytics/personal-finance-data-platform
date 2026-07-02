# Personal Finance Data Platform

## Overview

Personal Finance Data Platform is an end-to-end data engineering project that transforms raw credit card statements into a structured analytical data warehouse. The platform automates the complete data lifecycle—from file ingestion and parsing to transformation, validation, warehouse loading, and business intelligence reporting.

The primary objective is to provide complete visibility into personal spending behavior while demonstrating production-oriented data engineering practices, including ELT architecture, dimensional modeling, database design, testing, version control, and documentation.

---

## Objectives

* Automate ingestion of credit card statements from multiple financial institutions.
* Standardize transaction data into a consistent analytical model.
* Build a PostgreSQL-based dimensional warehouse.
* Generate reliable financial metrics for reporting and trend analysis.
* Deliver interactive dashboards through Power BI.
* Design the project using modular, maintainable, and extensible architecture.

---

## Core Features

### Data Ingestion

* File discovery
* Duplicate file detection
* Checksum validation
* File archival
* Error handling

### Statement Parsing

* Multi-bank parser framework
* PDF extraction
* Standardized transaction output

### Data Transformation

* Merchant normalization
* Transaction standardization
* Installment detection
* Category assignment
* Business rule processing

### Data Validation

* Schema validation
* Duplicate transaction detection
* Required field validation
* Data quality checks

### Data Warehouse

* Raw layer
* Staging layer
* Dimensional model
* Fact and dimension loading
* Historical reporting support

### Analytics

* Spending trends
* Category analysis
* Merchant analysis
* Monthly and yearly summaries
* Installment tracking
* Credit card utilization

---

## Technology Stack

| Layer                   | Technology         |
| ----------------------- | ------------------ |
| Programming Language    | Python             |
| Database                | PostgreSQL         |
| Data Processing         | Pandas             |
| Version Control         | Git                |
| Reporting               | Power BI           |
| Development Environment | Visual Studio Code |

---

## High-Level Architecture

```
Credit Card Statements
          │
          ▼
     File Ingestion
          │
          ▼
 Statement Parser Layer
          │
          ▼
 Transformation Layer
          │
          ▼
 Validation Layer
          │
          ▼
 Raw Layer
          │
          ▼
 Staging Layer
          │
          ▼
 Data Warehouse
          │
          ▼
 Power BI Dashboard
```

---

## Repository Structure

```
config/
data/
docs/
logs/
sql/
src/
tests/
```

Each directory has a single responsibility to maintain separation of concerns and support long-term maintainability.

---

## Project Status

Current Phase:

**Project Foundation**

The repository structure, documentation, and development standards are being established before implementation begins.

---

## License

This project is intended for educational, portfolio, and personal analytics purposes.
