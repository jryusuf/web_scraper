# Monitoring & Alerting

Continuous monitoring is essential for understanding system health, identifying performance bottlenecks, diagnosing errors, and ensuring data quality at scale.

## Log Aggregation

*   **Purpose:** Centralize logs from all distributed components (Dispatcher, Workers, Parsers, Django App, Databases) into a single, searchable system.
*   **Benefits:** Enables comprehensive debugging, tracing requests across services, analyzing error patterns, and understanding application behavior.
*   **Tools:** ELK Stack (Elasticsearch, Logstash, Kibana), Grafana Loki, Splunk, Datadog Logs, AWS CloudWatch Logs, Google Cloud Logging.
*   **Implementation:** Configure all services and applications (including Celery and potentially detailed Scrapy stats/logs if applicable) to ship logs (preferably in a structured format like JSON) to the chosen aggregation platform.

## Metrics Collection

*   **Purpose:** Collect time-series numerical data about system performance and behavior.
*   **Benefits:** Provides quantitative insights into system health, resource utilization, throughput, and error rates over time. Essential for dashboarding, alerting, and capacity planning.
*   **Tools:** Prometheus + Grafana, InfluxDB, Datadog, StatsD, AWS CloudWatch Metrics, Google Cloud Monitoring.
*   **Key Metrics to Track:**
    *   **Message Queue:**
        *   `queue_depth` (messages ready/unacknowledged)
        *   `message_age` (oldest message)
        *   `publish_rate` / `consume_rate`
    *   **Celery Workers:**
        *   `worker_count` (running instances)
        *   `cpu_utilization` / `memory_utilization` (per worker/node)
        *   `task_queued_duration` / `task_execution_duration`
        *   `task_success_rate` / `task_failure_rate` / `task_retry_rate` (per task type)
    *   **Scraping Specific (Custom Metrics / Logs):**
        *   `items_scraped_count` (per site/total)
        *   `pages_fetched_count` (per site/status code)
        *   `error_rate_per_domain`
        *   `proxy_success_rate` / `proxy_failure_rate`
        *   `captcha_detected_count`
    *   **Databases (Config & Structured):**
        *   `connection_count`
        *   `query_latency`
        *   `cpu/memory/disk_utilization`
        *   `replication_lag` (if applicable)
    *   **Business Metrics:**
        *   `new_jobs_added_per_hour/day`
        *   `data_freshness` (e.g., max time since last update for active sources)

## Visualization & Dashboarding

*   **Purpose:** Create visual representations of collected metrics and log data to provide an at-a-glance overview of system health and performance trends.
*   **Tools:** Grafana, Kibana, Datadog Dashboards, Google Cloud Monitoring Dashboards, AWS CloudWatch Dashboards.
*   **Implementation:** Build dashboards displaying key metrics (queue depths, worker status, error rates, scrape rates per site, DB performance). Allow filtering by time range, service, site, etc.

## Alerting

*   **Purpose:** Proactively notify operators about critical issues or potential problems based on predefined thresholds or patterns in metrics and logs.
*   **Tools:** Alertmanager (with Prometheus), Grafana Alerting, Datadog Monitors, Cloud provider native alerting (CloudWatch Alarms, Google Cloud Alerting). Integrations with PagerDuty, Slack, etc.
*   **Example Alert Conditions:**
    *   High message queue depth (above threshold for X minutes).
    *   High Celery task failure rate (above Y% over Z minutes).
    *   Persistently high error rate for a specific target website.
    *   Dead-letter queue depth increasing.
    *   Low scrape success rate for critical sources.
    *   High CPU/Memory utilization on workers or database nodes.
    *   Critical errors detected in aggregated logs.