## Overview

The ETL (Extract, Transform, Load) Pipeline is designed to automate the conversion of structured CSV or Java object data into RDF triples, perform SHACL validation, and load the data into the Stardog database. The custom ETL flow helps configure these steps in a modular way allowing flexibility based on the requirement of each dataset or domain.

This document outlines both the standard process and the custom changes implemented in our internal ETL pipeline variant.

---

## Modules and Core Features (Customized Flow)

### 1. *DB Config*
- Custom property file used to define Stardog DB name and connection parameters.
- Credentials and endpoint configuration handled securely using internal secrets manager (unlike open properties in sample module).
- Used to set up RDF loading parameters and custom query triggers.

### 2. *RDF Triple Generator*
- Accepts input data as CSV or directly from internal Java service responses.
- JSON mapping configurations are provided per dataset.
- In our implementation, an extended validator is integrated to validate CSV data schema before RDF conversion.

### 3. *RDF Writer*
- Generates .ttl turtle files for:
  - output.ttl
  - output_previous.ttl
  - add.ttl
  - delete.ttl
- Path configuration is driven by job-specific configuration instead of static paths.

### 4. *RDF Loader*
- Used to upload the triples into Stardog DB.
- Our implementation supports both:
  - Incremental Load (using add/delete file)
  - Full Load (deletes existing graph and uploads fresh triples)
- Load method is decided dynamically using job parameter load.type=incremental|full.

### 5. *Custom Query Executor*
- Post-load SPARQL queries can be configured for cleanup or data enrichment.
- We extended this to support external query templates stored in a central repository.

### 6. *SHACL Validator*
- Validates RDF data against SHACL shape definitions.
- Shape file location and rules are dynamically selected based on dataset type.
- Reports are generated and saved with timestamped names for auditing.

### 7. *SHACL Loader*
- Loads the validation report into the configured Stardog DB graph for tracking.
- We added notification hooks (email/Slack) on failure scenarios for validation.

---

## Additional Enhancements (vs. Reference Flow)

- *Centralized Config Management*: YAML-based global config to manage properties for each dataset/job.
- *Error Handling and Logging*: Structured logging with job-level logs stored in a central log aggregator.
- *Monitoring & Notifications*: Integrated basic monitoring for job failures with alerts.
- *Parallel Processing*: Where applicable, large CSV inputs are split and processed in parallel before merging.

---
