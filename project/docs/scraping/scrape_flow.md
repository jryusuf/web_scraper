# Scrape Flow: Search vs. Detail Pages

Scraping job listings typically involves a two-stage process: first identifying potential jobs on search result pages, and second, extracting detailed information from individual job posting pages.

## Stage 1: Search Result Page Scraping

*   **Goal:** Identify URLs of potentially relevant individual job postings based on configured search criteria (keywords, location, filters).
*   **Input:** A configured search task (e.g., target URL for a specific search on Site X).
*   **Process:**
    1.  Worker fetches the search results page (using `requests` or Playwright depending on site complexity).
    2.  Handles pagination/infinite scroll to navigate through multiple pages of results, if necessary.
    3.  Parses the search result page(s) to extract:
        *   **Job Detail Page URLs:** The primary output.
        *   (Optional) Basic metadata visible on the search page (e.g., title snippet, company, location, date posted, sometimes salary snippet).
        *   (Optional) Unique Job IDs if available.
    4.  Stores the raw HTML of the search result page(s) in S3 for auditing/debugging.
*   **Output:** A list of Job Detail Page URLs (and potentially basic associated metadata).

## Stage 2: Job Detail Page Scraping

*   **Goal:** Extract comprehensive, structured information from an individual job posting page.
*   **Input:** A specific Job Detail Page URL obtained from Stage 1.
*   **Process:**
    1.  Worker fetches the job detail page (often simpler, potentially just `requests`, but may still need Playwright if content loads dynamically).
    2.  Parses the page content to extract detailed fields (Job Title, Company Name, Full Description, Location, Salary Details, Employment Type, etc.) based on predefined selectors for that site.
    3.  Stores the raw HTML of the job detail page in S3.
*   **Output:** Raw HTML stored in S3, ready for the dedicated Parsing Service to extract structured data.

## Triggering Stage 2 from Stage 1

There are two primary models for managing this flow within the event-driven architecture:

### Model A: Internal Handling (e.g., within Scrapy or similar logic)

*   **Description:** A single job message triggers the worker to start the Stage 1 scrape. As the worker parses search results and finds detail page URLs, it *internally* generates and schedules requests for those Stage 2 pages within the same process/framework execution context (similar to how `yield scrapy.Request` works).
*   **Pros:** Simpler from an external orchestration perspective; keeps related scrape activities together; efficient if using a framework like Scrapy that manages internal requests well.
*   **Cons:** A single long-running task; failure during Stage 2 fetching might complicate overall job status reporting for the initial Stage 1 message.

### Model B: External Dispatching

*   **Description:** The worker processing the Stage 1 job message fetches search results. For each Job Detail URL found, it creates and publishes a *new, distinct job message* onto the central message queue, specifically for that Stage 2 URL. Separate worker instances pick up these Stage 2 messages.
*   **Pros:** Granular jobs; Stage 2 failures are isolated; potentially better load distribution if Stage 2 tasks are numerous or long-running.
*   **Cons:** Increased message volume on the central queue; requires careful deduplication logic (don't dispatch Stage 2 job if already processed recently); more complex orchestration.

**Chosen Approach:**

*   While Model B offers granularity, **Model A (Internal Handling)** is often simpler to manage initially, especially if the number of detail pages per search result page is reasonable. The worker task handles the entire flow from search to extracting all detail page HTMLs initiated by that search. The complexity shifts to ensuring the worker task manages this internal flow robustly. If a framework like Scrapy were used inside the worker, this would be its natural mode of operation. If using plain `requests`/`Playwright`, the worker code would need to loop or manage these sub-requests. *(Decision should be finalized based on implementation complexity vs. desired granularity)*.