# Dataform Multi-Region Smart Meter Analytics

## Overview

This project demonstrates how Google Cloud Dataform can be used to build reusable, parameterized data transformation pipelines across multiple geographic regions using a single codebase.

The solution leverages Dataform variables and dynamic table references to process smart meter data stored in region-specific BigQuery datasets while minimizing code duplication.

## Business Problem

Utility organizations often maintain separate datasets across regions due to:

* Data residency requirements
* Regulatory compliance
* Regional operational boundaries
* Performance optimization

Maintaining separate transformation pipelines for each region increases development effort, testing complexity, and operational overhead.

This project demonstrates a scalable pattern where a single transformation model dynamically targets different datasets based on deployment configuration.

## Architecture

```text
                   ┌─────────────────────┐
                   │ Dataform Variables  │
                   │ region_suffix=us/eu │
                   └──────────┬──────────┘
                              │
                              ▼
                   ┌─────────────────────┐
                   │ hourly_usage.sqlx   │
                   │ Reusable Logic      │
                   └──────────┬──────────┘
                              │
          ┌───────────────────┴───────────────────┐
          │                                       │
          ▼                                       ▼

 ┌─────────────────────┐              ┌─────────────────────┐
 │ smart_meters_us     │              │ smart_meters_eu     │
 │ meter_dataset_us    │              │ meter_dataset_eu    │
 └─────────────────────┘              └─────────────────────┘

          ▼                                       ▼

   BigQuery Output Tables Generated Per Region
```

## Repository Structure

```text
definitions/
├── declarations.sqlx
├── declaration_eu.sqlx
└── hourly_usage.sqlx

workflow_settings.yaml
```

## Components

### Source Declarations

#### US Dataset

```sql
config {
  type: "declaration",
  database: "dataform-demo-483620",
  schema: "meter_dataset_us",
  name: "smart_meters_us"
}
```

Declares an existing BigQuery source table for US smart meter data.

#### EU Dataset

```sql
config {
  type: "declaration",
  database: "dataform-demo-483620",
  schema: "meter_dataset_eu",
  name: "smart_meters_eu"
}
```

Declares an existing BigQuery source table for EU smart meter data.

---

### Transformation Model

```sql
config {
  type: "table",
  schema: "meter_dataset_" + dataform.projectConfig.vars.region_suffix
}
```

The transformation dynamically selects the target schema and source table using the configured region variable.

```sql
SELECT *
FROM ${ref("smart_meters_" + dataform.projectConfig.vars.region_suffix)}
```

This enables:

* Single transformation logic
* Multiple deployment targets
* Reduced maintenance effort
* Consistent analytics processing

---

### Workflow Configuration

```yaml
vars:
  region_suffix: "us"
```

The deployment variable controls which source declaration and target dataset are used during execution.

Examples:

```yaml
region_suffix: "us"
```

Processes:

```text
smart_meters_us
meter_dataset_us
```

```yaml
region_suffix: "eu"
```

Processes:

```text
smart_meters_eu
meter_dataset_eu
```

## Key Data Engineering Patterns Demonstrated

### Parameterized ELT

A single transformation pipeline supports multiple environments through configuration rather than duplicated code.

### Source Abstraction

Dataform declarations separate source definitions from transformation logic.

### Regional Data Isolation

Supports data residency and regulatory requirements while maintaining centralized governance.

### Reusable Analytics Framework

Additional regions can be onboarded by:

1. Creating a new declaration
2. Adding a region variable
3. Deploying the same transformation logic

No SQL rewrites are required.

## Technologies

* Google Cloud Platform (GCP)
* BigQuery
* Dataform
* SQLX
* YAML

## Potential Enhancements

Future improvements could include:

* Data quality assertions
* Incremental processing
* Partitioned tables
* CI/CD deployment pipelines
* Automated testing
* Data lineage visualization
* Multi-environment promotion (Dev/Test/Prod)
* Governance and metadata management

## Learning Objectives

This project demonstrates:

* Dataform project structure
* BigQuery source declarations
* Dynamic table references
* Configuration-driven development
* Multi-region analytics architecture
* Enterprise ELT design patterns


