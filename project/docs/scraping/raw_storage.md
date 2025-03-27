# Raw Data Storage

Storing the raw, unprocessed HTML content fetched from target websites is a crucial step for resilience and decoupling.

## Storage Choice: Cloud Object Storage

*   **Technology:** AWS S3, Google Cloud Storage (GCS), Azure Blob Storage.
*   **Rationale:**
    *   **Scalability:** Virtually unlimited storage capacity.
    *   **Durability & Availability:** Designed for high durability and availability, reducing risk of data loss.
    *   **Cost-Effectiveness:** Generally cheaper than database storage for large volumes of raw data.
    *   **Decoupling:** Separates the fetching process from the parsing process.
    *   **Integration:** Easily integrates with other cloud services (e.g., triggering parsing functions via S3 events).

## Storage Format & Structure

### Content

*   Store the **full HTML content** exactly as received from the target website after fetching (and rendering, if Playwright was used).

### File Naming Convention

*   Use a consistent and informative naming convention that allows easy retrieval and avoids collisions. Example structure:
    `{source_website}/{job_id_or_hash}/{scrape_timestamp}.html.gz`
    *   `source_website`: e.g., `linkedin`, `indeed_com`
    *   `job_id_or_hash`: A unique identifier for the job (ideally from the source site, or a hash of the URL/content if no ID exists).
    *   `scrape_timestamp`: ISO 8601 format timestamp (e.g., `2025-03-27T183000Z`).
*   Consider partitioning within the bucket (using prefixes that look like folders) by date (e.g., `year=2025/month=03/day=27/`) for efficient querying by date ranges if using tools like AWS Athena later.

### Compression

*   **Method:** Compress the HTML content using `gzip` (`.gz`) or `brotli` (`.br`) before uploading.
*   **Benefits:** Significantly reduces storage space requirements (HTML compresses well) and associated costs. Also reduces data transfer time/cost.
*   **Implementation:** Use standard Python libraries (`gzip`, `brotli`) in the worker process after fetching the content and before uploading to the object storage bucket. The parsing service will need to decompress the files upon retrieval.

## Process Integration

*   The scraping worker (Celery task) is responsible for fetching the content, optionally compressing it, and uploading it to the designated S3 bucket path.
*   The S3 object key (path) should ideally be included in any subsequent message triggering the parsing stage, or be derivable from the job metadata.