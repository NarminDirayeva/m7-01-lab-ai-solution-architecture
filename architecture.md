graph TD
    subgraph ONLINE_BOUNDARY [Online Serving Boundary - Latency Sensitive]
        A[Client App / Payment Gateway] -->|1. POST /score request| B(API Gateway)
        B --> C[Fraud Scoring Service]
        C -->|2. Fetch Features| X[(Online Feature Store - Redis)]
        X -->|Features| C
        C -->|3. Run Inference| D[Model Serving Engine - Triton]
        D -->|4. Return Score| C
        C -->|5. Action Decision| F[Decision Engine]
        F -->|6. Allow / Block / Step-up| G[Payment Processor]
    end

    subgraph STREAMING_INGESTION [Streaming & Logging]
        C -->|7. Log Prediction & Features| E[Kafka - Event Stream]
        E -->|8. Log Transactions| L[Data Lake - S3/Iceberg]
    end

    subgraph OFFLINE_BOUNDARY [Offline Boundary - Batch Processing]
        L -->|Historical Data| M[Offline Feature Store]
        M -->|Training Dataset| H[Model Training Pipeline]
        H -->|9. Push Artifacts| I[Model Registry - MLflow]
        I -->|10. Deploy Model| D
    end

    subgraph MONITORING_FEEDBACK [Monitoring & Feedback Loop]
        C -->|Stream Logs & Metrics| J[EvidentlyAI / Prometheus]
        J -->|Data Drift Alert| K{Retrain Trigger?}
        K -->|Yes: Airflow DAG| H
    end

    style ONLINE_BOUNDARY fill:#fff3cd,stroke:#ffc107,stroke-width:2px
    style STREAMING_INGESTION fill:#d1ecf1,stroke:#17a2b8,stroke-width:2px
    style OFFLINE_BOUNDARY fill:#e2e3e5,stroke:#6c757d,stroke-width:2px
    style MONITORING_FEEDBACK fill:#d4edda,stroke:#28a745,stroke-width:2px