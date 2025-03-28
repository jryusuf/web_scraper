# System Overview

This section provides a high-level overview of the proposed architecture for the job scraping and processing system. The design emphasizes modularity, scalability, and decoupling through an event-driven approach.

| Component                 | Primary Function                                     |
| :------------------------ | :--------------------------------------------------- |
| Config DB (PostgreSQL)    | Stores scraping targets, settings, site metadata   |
| Django Admin              | UI for managing the Config DB                      |
| Dispatcher Service        | Reads config, creates & queues job messages        |
| Message Queue             | Decouples Dispatcher from Workers, holds jobs      |
| Celery Workers            | Execute scraping tasks (fetch, interact)           |
| S3 Bucket                 | Stores raw fetched HTML                            |
| Parsing Service/Worker    | Extracts structured data from raw HTML             |
| Structured DB (PostgreSQL)| Stores cleaned, structured job data                |
| Monitoring & Logging      | Collects logs & metrics for observability        |
| Analysis/ML Environment   | Consumes structured data for analysis              |

## Architecture Diagram

This section provides a high-level overview of the proposed architecture for the job scraping and processing system. To understand the system clearly, we'll look at it from two perspectives, inspired by the C4 model: System Context (Level 1) and focused Container views (Level 2).

## Level 1: System Context Diagram

This diagram shows the Job Scraping System as a single "black box" and illustrates how it interacts with its users and the external systems it depends on.

```mermaid
graph TD
    subgraph External Actors & Systems
        A(Admin User)
        TWS[Target Websites]
        AMS[Analysis/ML Systems]
    end

    subgraph Our System
        S[Job Scraping System]
    end

    A -- Manages Configuration --> S;
    S -- Scrapes Job Data From --> TWS;
    S -- Provides Cleaned Data To --> AMS;

    style S fill:#lightblue,stroke:#333,stroke-width:2px```
```

**Key Interactions (Context):**

1.  **Admin User:** Configures scraping targets, manages settings, monitors system health.
2.  **Job Scraping System:** The complete system.
3.  **Target Websites:** External sites providing job data.
4.  **Analysis/ML Systems:** Downstream consumers of the cleaned data.

---

## Level 2: Container Diagrams (Focused Views)

Instead of a single complex diagram, we break down the system's internal containers and interactions by focusing on one major subsystem at a time.

### 1. Configuration Subsystem Focus

This view shows how configuration is managed and accessed.

```mermaid
graph TD
    %% Actors
    A(Admin User)

    %% Focused Subsystem
    subgraph Configuration
        B(Config Service / Django App);
        C{Config DB / PostgreSQL};
        style C fill:#9cf,stroke:#333,stroke-width:2px;
    end

    %% Interacting Systems/Containers
    D(Dispatcher Service)

    %% Interactions
    A -- Manages via UI --> B;
    B -- Reads/Writes --> C;
    D -- Reads Config --> C;
```

Focus: The Admin User interacts with the Config Service (Django App) to manage settings stored in the Config DB. The Dispatcher Service reads from the Config DB to know what jobs to schedule.

### 2. Job Dispatching Subsystem Focus

This view shows how jobs are created and queued.

```mermaid
graph TD
    %% Focused Subsystem
    subgraph Dispatching & Queueing
        D(Dispatcher Service);
        E{Message Queue};
        style E fill:#f9f,stroke:#333,stroke-width:2px;
    end

    %% Interacting Systems/Containers
    C{Config DB / PostgreSQL};
    F(Scraping Worker Pool);
    L(Monitoring System);

    %% Interactions
    D -- Reads Config --> C;
    D -- Publishes Job --> E;
    F -- Consumes Job --> E;
    D -- Sends Logs/Metrics --> L;
```

Focus: The Dispatcher Service reads from the Config DB, creates job messages, and publishes them to the Message Queue. The Scraping Worker Pool consumes jobs from the queue. The dispatcher also sends operational data to the Monitoring System.

### 3. Scraping Execution Subsystem Focus

This view shows how scraping tasks are executed and raw data is stored.

```mermaid
graph TD
    %% Focused Subsystem
    subgraph Scraping Execution
        F(Scraping Worker Pool / Celery);
        H(Raw HTML Storage / S3);
        style H fill:#ccf,stroke:#333,stroke-width:2px;
    end

    %% Interacting Systems/Containers
    E{Message Queue};
    TWS[Target Websites];
    L(Monitoring System);

    %% Interactions
    F -- Consumes Job --> E;
    F -- Fetches From --> TWS;
    F -- Writes Raw HTML --> H;
    F -- Sends Logs/Metrics --> L;
```

Focus: The Scraping Worker Pool consumes jobs from the Message Queue, interacts with external Target Websites, stores the fetched raw HTML in Raw HTML Storage, and sends operational data to the Monitoring System.

### 4. Data Processing Subsystem Focus

This view shows how raw data is processed and stored structurally.

```mermaid
graph TD
    %% Focused Subsystem
    subgraph Data Processing
        I(Parsing Service / Celery or Lambda);
        J{Structured Data Store / PostgreSQL};
        style J fill:#9cf,stroke:#333,stroke-width:2px;
    end

    %% Interacting Systems/Containers
    H(Raw HTML Storage / S3);
    AMS[Analysis/ML Systems];
    L(Monitoring System);

    %% Interactions
    I -- Reads Raw HTML --> H;
    I -- Writes Structured Data --> J;
    AMS -- Reads Clean Data --> J;
    I -- Sends Logs/Metrics --> L;
```

Focus: The Parsing Service reads raw HTML from Raw HTML Storage, processes it, and writes the cleaned, structured data into the Structured Data Store. Downstream Analysis/ML Systems read from this store. The service also reports to the Monitoring System.

### 5. Monitoring & Analysis Subsystem Focus

This view shows how monitoring data is collected and how final data is consumed.

```mermaid
graph TD
    %% Focused Subsystem
    subgraph Observability & Consumption
        L(Monitoring System);
        J{Structured Data Store / PostgreSQL};
        style J fill:#9cf,stroke:#333,stroke-width:2px;
    end

    %% Interacting Systems/Containers
    A(Admin User)
    F(Scraping Worker Pool);
    I(Parsing Service);
    D(Dispatcher Service);
    AMS[Analysis/ML Systems];

    %% Interactions
    F -- Sends Logs/Metrics --> L;
    I -- Sends Logs/Metrics --> L;
    D -- Sends Logs/Metrics --> L;
    %% B(Config Service) -- Sends Logs/Metrics --> L; 

    L -- Provides Dashboards/Alerts --> A;
    AMS -- Reads Clean Data --> J;
```

Focus: Various system components (Workers, Parsing, Dispatcher, etc.) send logs and metrics to the central Monitoring System. The Admin User views dashboards and receives alerts from Monitoring. Analysis/ML Systems consume the final data from the Structured Data Store.