# Task Queue

The Message Queue is the central communication channel decoupling the Dispatcher from the pool of Scraper Workers.

## Role & Purpose

*   Acts as a buffer holding job messages sent by the Dispatcher.
*   Distributes job messages to available Scraper Workers.
*   Enhances system resilience: If workers are down, messages wait in the queue. If the dispatcher is down, workers continue processing existing messages.
*   Enables independent scaling of Dispatcher and Workers.

```mermaid
sequenceDiagram
    participant D as Dispatcher
    participant Q as Message Queue Broker
    participant W as Celery Worker

    Note over D, W: Worker is idle, waiting for jobs.

    D->>+Q: Publish(Job Message)
    Note over Q: Message stored persistently.
    Q-->>-D: Publish Acknowledged (Optional)

    Note over Q, W: Broker makes message available.
    W->>+Q: Consume()/Get Message
    Note over Q: Message marked as "unacknowledged".
    Q-->>-W: Job Message

    Note over W: Worker begins processing job...
    W->>W: Execute Scraping Task (Fetch, Parse etc.)

    alt Task Successful
        Note over W: Job completed successfully.
        W->>+Q: Acknowledge(Message)
        Note over Q: Message permanently removed.
        Q-->>-W: Ack Confirmed (Optional)
    else Task Failed (After Retries)
        Note over W: Job failed after all retries.
        W->>+Q: Negative Acknowledge / Route to DLQ
        Note over Q: Message moved to Dead-Letter Queue or discarded.
        Q-->>-W: Nack/Route Confirmed (Optional)
    else Worker Crashes Mid-Task
        Note over W: Worker crashes before acknowledging.
        Note over Q: Message remains "unacknowledged".<br/>After visibility timeout,<br/>Broker makes message available again<br/>for another worker.
    end
```

## Technology Choices

*   **RabbitMQ:** Feature-rich, mature, protocol-based (AMQP) message broker. Offers routing flexibility, acknowledgements, persistence, and DLQ support. Requires separate deployment and management.
*   **Redis:** Often used as a simpler message broker for Celery. Fast in-memory store, but persistence and reliability features might be less robust than RabbitMQ depending on configuration. Can serve multiple purposes (caching, broker).
*   **AWS SQS (Simple Queue Service):** Fully managed cloud service. Highly scalable and durable. Offers standard and FIFO queues, DLQ support. Reduces operational overhead. Pay-per-use pricing.
*   **Google Cloud Pub/Sub:** Fully managed cloud service. Global scale, push/pull delivery, filtering. Reduces operational overhead.

**Selection Criteria:** Choice depends on factors like existing infrastructure, operational preferences (managed vs. self-hosted), required features (e.g., message ordering guarantees - usually not needed for scraping jobs), and scalability needs. Managed services (SQS, Pub/Sub) are often preferred for cloud-native deployments to reduce operational burden.

## Key Features Used

*   **Message Persistence:** Ensure messages are not lost if the broker restarts.
*   **Worker Acknowledgements:** Workers should acknowledge messages only *after* successfully completing the task (or deciding it's a terminal failure) to prevent message loss if a worker crashes mid-task.
*   **Dead-Letter Queue (DLQ) Support:** Automatically route messages that fail repeatedly to a separate queue for investigation.
*   **Scalability:** The queue system must handle the peak rate of job dispatch and consumption.

## Interaction Flow

1.  Dispatcher publishes a job message to a specific queue/topic.
2.  The Message Broker makes the message available.
3.  An available Celery Worker retrieves (consumes) the message.
4.  The Worker processes the job.
5.  Upon completion/failure, the Worker sends an acknowledgement (`ack`) to the Broker, removing the message from the queue (or routing to DLQ upon repeated failure).