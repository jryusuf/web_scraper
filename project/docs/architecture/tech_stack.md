
# Technology Stack Summary

This section outlines the proposed core technologies for each major component of the system, along with justifications based on the design goals.

| Component                     | Technology Choices                                    | Justification                                                                                                                               |
| :---------------------------- | :---------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------ |
| **Language**                  | Python 3.x                                            | Mature ecosystem, excellent libraries for web scraping, data processing, web frameworks, ML, and cloud integration.                           |
| **Configuration Mgmt**        | Django + PostgreSQL                                   | **Django:** Built-in Admin UI for easy config management, robust ORM, migrations. **PostgreSQL:** Reliable relational DB for storing config. |
| **Job Scheduling/Dispatch**   | Python Service (potentially Django Mgmt Command/Celery Beat) | Leverages Python ecosystem. Can be scheduled via cron, systemd timer, or Celery Beat integrated with the Django app.                        |
| **Message Queue**             | RabbitMQ / Redis / AWS SQS / Google Pub/Sub           | Decouples dispatcher & workers. Choice depends on scale/ops preference: **RabbitMQ:** Feature-rich. **Redis:** Simpler. **SQS/PubSub:** Managed cloud services. |
| **Scraping Workers**          | Celery + Python                                       | **Celery:** Mature, distributed task queue system for Python, integrates well with brokers.                                                 |
| **Core Fetching Logic**       | `requests` + `urllib3.Retry` / `backoff`              | Lightweight, standard Python HTTP library with robust, configurable retry/backoff mechanisms for simple fetches.                            |
| **Dynamic Content Fetching**  | Playwright (or potential alternative: Splash/Commercial API) | **Playwright:** Modern browser automation for JS-heavy sites. **Alternatives:** Considered for resource optimization/offloading browser mgmt. |
| **Proxy/UA Rotation**         | Custom Scrapy Middleware / Python Logic in Worker     | Integrates rotation logic within the fetching process. Can use lists, files, or proxy service APIs.                                         |
| **Raw Data Storage**          | AWS S3 / Google Cloud Storage / Azure Blob Storage    | Scalable, durable, cost-effective object storage for raw HTML. Cloud provider choice often aligns with other infrastructure.                |
| **Parsing Logic**             | Python (`BeautifulSoup`, `lxml`, potentially `parsel`) | Standard Python libraries for efficient HTML parsing.                                                                                       |
| **Parsing Service/Worker**    | Celery Workers / AWS Lambda / Google Cloud Functions  | **Celery:** Reuse worker infrastructure. **Serverless:** Option if parsing is stateless and triggered by S3 events.                           |
| **Structured Data Storage**   | PostgreSQL (with JSONB)                               | **PostgreSQL:** Strong relational features, SQL querying, data integrity. **JSONB:** Flexibility for semi-structured fields (skills, salary).     |
| **Monitoring - Logs**         | ELK Stack / Grafana Loki / CloudWatch Logs            | Centralized log aggregation for debugging and tracing.                                                                                      |
| **Monitoring - Metrics**      | Prometheus + Grafana / CloudWatch Metrics / Datadog   | Time-series metrics for performance monitoring, dashboarding, and alerting.                                                               |
| **Orchestration (Scale)**     | Docker + Kubernetes (EKS/GKE/AKS)                     | **Docker:** Containerization. **Kubernetes:** Scalable deployment, management, auto-scaling of workers and services.                          |
| **Infrastructure Provisioning** | Terraform / Pulumi / CloudFormation                 | Infrastructure as Code for reproducible and automated environment setup.                                                                      |
| **CI/CD**                     | GitHub Actions / GitLab CI / Jenkins                  | Automated building, testing, and deployment pipelines.                                                                                      |

This stack provides a balance of established, well-supported technologies with scalable cloud-native patterns, aligning with the system's design goals. Specific choices (e.g., RabbitMQ vs. SQS) may be influenced by existing infrastructure or operational preferences.