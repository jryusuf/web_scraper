# Error Handling & Retries

A robust scraping system must anticipate and handle various failures gracefully. This section outlines the multi-layered approach to error handling and retries.

## Fetcher-Level Retries

*   **Mechanism:** Implemented directly within the data fetching logic used by the Celery workers.
*   **Tools:** Utilizing the `requests` library with `urllib3.Retry` adapter or the `backoff` library decorator.
*   **Strategy:**
    *   Automatically retry requests that fail due to transient network errors (timeouts, connection errors).
    *   Automatically retry requests based on specific HTTP status codes indicating temporary server issues (e.g., 500, 502, 503, 504, 429 - configurable).
    *   Implement exponential backoff with jitter to avoid overwhelming the target server during retries.
    *   Configure a maximum number of retries per request to prevent infinite loops.

## Worker-Level Task Retries (Celery)

*   **Mechanism:** Celery provides built-in support for retrying tasks if they fail.
*   **Strategy:**
    *   Wrap the core scraping logic within the Celery task in `try...except` blocks.
    *   Catch exceptions that represent potentially recoverable task failures (e.g., persistent fetch errors after fetcher-level retries, temporary infrastructure issues).
    *   Use `task.retry(exc=exception, countdown=...)` to schedule the task for a retry after a specified delay, often with exponential backoff.
    *   Configure a maximum number of retries for each Celery task.

## Handling Specific Scraping Errors

### Website Blocks / Bans (e.g., 403 Forbidden)

*   **Detection:** Identify status codes (403) or page content changes indicating a block.
*   **Immediate Action (Middleware/Worker Logic):**
    *   Rotate Proxy IP address immediately for the failed request and subsequent requests to that domain.
    *   Rotate User-Agent string.
    *   Potentially introduce a longer, domain-specific cool-down period before retrying.
*   **Long-Term Action (Monitoring/Dispatcher):** If blocks persist for a specific site across multiple jobs, the central monitoring system should detect this pattern. The Dispatcher might then be configured (manually or automatically based on alerts) to temporarily reduce the scraping frequency or pause job dispatch for that site.

### CAPTCHAs

*   **Detection:** Identify CAPTCHA presence in page content or via specific selectors/status codes.
*   **Strategy:**
    *   **Avoidance:** The primary strategy through polite scraping (delays, AutoThrottle principles), good user agents, and high-quality proxies.
    *   **Rotation:** Try rotating proxies/user agents as CAPTCHAs can be session/IP specific.
    *   **Solving Services (If unavoidable):** Integrate with third-party CAPTCHA solving services (e.g., 2Captcha, Anti-CAPTCHA). This adds cost and complexity and is often a last resort. Requires specific logic within the worker or potentially using commercial scraping APIs that handle this.
    *   **Failure & Reporting:** If CAPTCHAs cannot be bypassed/solved, the job should fail gracefully and log the issue clearly for investigation.

## Dead-Letter Queues (DLQ)

*   **Concept:** A separate queue where messages are sent if they fail processing repeatedly (exceeding all retry limits) in the main worker queue.
*   **Purpose:**
    *   Prevents failing messages from blocking the main queue indefinitely.
    *   Allows for investigation of persistently failing jobs without interrupting normal processing.
    *   Enables potential manual intervention or bulk reprocessing later.
*   **Implementation:** Configure the primary message broker (RabbitMQ, SQS) to route expired or max-retried messages to a designated DLQ. Monitor the DLQ depth via alerting.

## Error Reporting & Logging

*   All retry attempts, backoff delays, and terminal failures at both the fetcher and worker levels must be logged comprehensively.
*   Logs should include the target URL, error type/message, stack trace (if applicable), number of attempts, and relevant context (e.g., proxy used).
*   This centralized logging (see Monitoring) is crucial for diagnosing issues, understanding failure patterns, and improving scraper robustness.