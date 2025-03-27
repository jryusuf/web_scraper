# System Overview

This section provides a high-level overview of the proposed architecture for the job scraping and processing system. The design emphasizes modularity, scalability, and decoupling through an event-driven approach.

## Architecture Diagram

The following diagram illustrates the main components and data flow:

```mermaid
graph TD
    subgraph Configuration & Control
        A(User/Admin) -->|Manages Targets| B(Django Admin UI);
        B --> C{PostgreSQL Config DB};
    end

    subgraph Job Scheduling & Dispatch
        D(Scheduler/Dispatcher Service) -- Reads Config --> C;
        D -- Creates Job Message --> E{Message Queue};
        style E fill:#f9f,stroke:#333,stroke-width:2px;
    end

    subgraph Data Acquisition  Scraping Workers
        F(Celery Workers) -- Consumes Job --> E;
        F -- Fetch Request --> G[Target Websites];
        F -- Proxy/UA Rotation --> F;
        F -- Stores Raw HTML --> H(S3 Bucket);
        style H fill:#ccf,stroke:#333,stroke-width:2px;
        F -- Reports Status/Errors --> L(Monitoring & Logging);
    end

    subgraph Data Processing & Storage
        I(Parsing Service/Worker) -- Triggered by S3 Event / Queue --> H;
        I -- Parses HTML --> I;
        I -- Stores Structured Data --> J{PostgreSQL Structured DB};
        style J fill:#9cf,stroke:#333,stroke-width:2px;
        I -- Reports Status/Errors --> L;
    end

    subgraph Analysis & Monitoring
        K(Analysis/ML Environment) -- Reads Data --> J;
        L -- Receives Logs/Metrics --> L;
        M(Monitoring Dashboard/Alerting) -- Reads --> L;
        M -- Alerts --> A;
    end


```

```mermaid
graph LR
    hello --> world
    world --> again
    again --> hello
```