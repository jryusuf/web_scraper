# Handling Rate Limiting & Politeness

Respecting website resources and avoiding rate limits is crucial for ethical and sustainable scraping.

## Politeness Delays

*   **Mechanism:** Introduce artificial delays between consecutive requests to the same domain.
*   **Implementation:**
    *   Use a base `DOWNLOAD_DELAY` configured per site or globally (e.g., 1-3 seconds).
    *   Respect `Crawl-delay` directive from `robots.txt` if present and stricter than the base delay.
    *   Consider dynamic delays (similar to Scrapy's AutoThrottle) that adjust based on server response times, although this adds complexity outside the Scrapy framework. Logic within the worker could increase delays if rate-limiting responses (429) are encountered.

## Concurrency Limits

*   **Mechanism:** Limit the number of simultaneous requests being made to the same website domain or IP address.
*   **Implementation:**
    *   If using Celery workers without Scrapy's internal scheduler managing concurrency per domain, this needs careful management.
    *   Potential strategies include: using distributed locks (based on domain name) via Redis/ZooKeeper, limiting worker concurrency per domain via routing keys/queues, or implementing domain-aware throttling within the worker's fetching logic.
    *   Kubernetes resource limits also implicitly limit overall concurrency.

## Handling Rate Limit Responses (HTTP 429)

*   **Detection:** Identify the `HTTP 429 Too Many Requests` status code (and potentially others used for rate limiting).
*   **Strategy:**
    *   Treat as a trigger for the fetcher-level retry mechanism (`requests`+`Retry`).
    *   Ensure the retry logic includes a significant backoff delay (exponential backoff) specifically for 429 errors.
    *   Log these events clearly.
    *   If 429 errors persist for a site, it's a strong signal that base delays need increasing or concurrency needs reducing for that domain (potentially requiring configuration updates via monitoring feedback).