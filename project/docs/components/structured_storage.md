# Structured Data Storage (PostgreSQL)

This component is the destination for the cleaned, processed, and structured job posting data extracted from the raw HTML.

## Role & Purpose

*   Store the final, usable job data in a structured format suitable for querying and analysis.
*   Enforce data types and potentially basic constraints (e.g., uniqueness on source URL + timestamp).
*   Serve as the primary data source for downstream applications, analytics platforms, or ML model training pipelines.

## Technology Choice: PostgreSQL

*   **Rationale:**
    *   **Relational Integrity:** Strong support for data types, constraints, and relationships.
    *   **Powerful SQL:** Standard, expressive query language for complex data retrieval and aggregation.
    *   **JSONB Support:** Excellent built-in support for storing and efficiently querying semi-structured JSON data within relational tables. This is ideal for handling fields with variable structures (e.g., salary details, extracted skills lists, location breakdowns).
    *   **Maturity & Reliability:** Proven, ACID-compliant database with a large ecosystem and community support.
    *   **Scalability:** Scales well vertically and offers various replication and high-availability options. Extensions like `pgvector` can add specialized capabilities if needed later.

## Data Schema Design (Conceptual)

A primary table (e.g., `JobPostings`) would store the core information.

### `JobPostings` Table Example
*   `id`: Primary Key (Serial).
*   `source_website`: Text (e.g., 'linkedin', 'indeed_com').
*   `source_url`: Text (Unique identifier for the specific posting).
*   `job_id_on_source`: Text (Nullable, ID used by the source site).
*   `scraped_timestamp`: TimestampTZ (When this record was scraped/parsed).
*   `first_seen_timestamp`: TimestampTZ (When this job URL was first encountered).
*   `last_processed_timestamp`: TimestampTZ (When this record was last updated by the parser).
*   `raw_html_s3_path`: Text (Link back to the raw source in S3).
*   `job_title_raw`: Text.
*   `company_name`: Text (Nullable).
*   `location_raw`: Text (Nullable).
*   `location_structured`: **JSONB** (Nullable, e.g., `{"city": "London", "country": "UK", "remote": false}`).
*   `description_text`: Text (Cleaned job description).
*   `description_html`: Text (Nullable, Raw HTML description if needed).
*   `publish_date`: Date/TimestampTZ (Nullable, Standardized).
*   `employment_type`: Text (Nullable, e.g., 'Full-time', 'Contract').
*   `salary_raw`: Text (Nullable, The raw salary string).
*   `salary_structured`: **JSONB** (Nullable, e.g., `{"min": 50000, "max": 70000, "currency": "USD", "period": "annual"}`).
*   `extracted_skills`: **JSONB** (Nullable, Array of strings, e.g., `["Python", "SQL", "AWS"]`).
*   `categorized_role`: Text (Nullable, Standardized role category).
*   `seniority_level`: Text (Nullable, Inferred seniority).
*   ... other relevant fields.

### Indexing
*   Create indexes on frequently queried columns like `source_url`, `scraped_timestamp`, `source_website`, `job_title_raw`, `company_name`, and potentially use GIN indexes for efficient querying within `JSONB` fields.

## Data Interaction

*   The Parsing Service/Worker is responsible for `INSERT`ing new records or potentially `UPDATE`ing existing ones (based on `source_url`) if re-processing occurs.
*   Downstream analysis tools, ML pipelines, or applications query this database using SQL.

This setup provides a robust and flexible foundation for storing and accessing the valuable structured job data.