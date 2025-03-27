# Scraper Workers (Celery)

The Workers are the core execution units responsible for performing the actual web scraping tasks.

## Role & Purpose

*   Consume job messages from the Message Queue.
*   Interpret the job message to understand the target URL and required actions/parameters.
*   Execute the data fetching logic using either lightweight HTTP clients (`requests`) or browser automation (`Playwright`) as specified.
*   Implement politeness delays, proxy rotation, and User-Agent rotation during fetching.
*   Handle fetcher-level errors and retries.
*   Process pagination or dynamic content loading logic.
*   Upon successful fetching, store the raw HTML content into the S3 Bucket.
*   Report task status (success, failure, retry needed) back via Celery and potentially to the Monitoring system.
*   Acknowledge messages from the queue upon definitive completion or failure.

## Technology Choice: Celery

*   **Rationale:** Celery is the standard, feature-rich distributed task queue framework for Python.
*   **Benefits:**
    *   Integrates seamlessly with message brokers (RabbitMQ, Redis, SQS).
    *   Handles worker management, task distribution, and concurrency.
    *   Provides built-in support for task retries with backoff.
    *   Mature and well-documented.

## Worker Implementation Details

### Task Definition
*   Define Celery tasks corresponding to the scraping jobs (e.g., a single task type that adapts based on message parameters).
*   The task receives the job message content as input.

### Fetching Logic Integration
*   The task code imports and uses the chosen fetching libraries (`requests` with `Retry` adapter, `Playwright`).
*   Logic branches based on `requires_playwright` flag in the job message.

### Resource Management
*   **Concurrency:** Configure the number of concurrent tasks each Celery worker process can handle (`--concurrency` flag). This needs careful tuning, especially when using Playwright, to avoid overwhelming CPU/RAM. Consider using `gevent` or `eventlet` execution pools for I/O-bound `requests` tasks, but stick to `prefork` (default) or `solo` for CPU/RAM-intensive Playwright tasks.
*   **Playwright Cleanup:** Ensure Playwright browser instances and contexts are closed properly at the end of each task (or reused carefully) to prevent resource leaks.
*   **Containerization:** Run workers inside Docker containers managed by Kubernetes for scaling and resource isolation. Define appropriate CPU/Memory requests and limits in Kubernetes deployments.

### Scaling
*   Scale the number of worker instances (pods in Kubernetes) horizontally based on the depth of the Message Queue or CPU/Memory load using Kubernetes HPA and potentially KEDA for queue-based scaling.

## Error Handling within Workers
*   Implement `try...except` blocks around fetching and processing logic.
*   Utilize Celery's `task.retry()` for recoverable errors.
*   Log all errors comprehensively to the central logging system.
*   Ensure tasks eventually terminate (success or failure) and acknowledge the message queue correctly.