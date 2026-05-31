# ADR 0001: Selection of Redis as the Online Feature Store for Low-Latency Scoring

## Context
The fraud detection system must make predictions under an 80 ms p95 latency budget at 300 RPS. The ML model requires immediate access to historical feature aggregates (e.g., 24h transaction counts). Querying these from the main transactional database or calculating them on-the-fly during the request cycle introduces latency spikes (>200 ms) that violate the SLA.

## Decision
I decided to implement an in-memory Online Feature Store using **Redis**, paired with an offline Data Lake (S3/Iceberg). Redis acts as a dedicated low-latency cache for pre-computed features. During the scoring flow, the `Fraud Scoring Service` fetches all required inputs using low-latency bulk retrieval operations (e.g., MGET), typically within a few milliseconds, preserving most of the latency budget for model inference and decision logic.

## Alternatives rejected
* **PostgreSQL Read Replicas:** Rejected due to unpredictable query latencies (15–50 ms) under high concurrent load, leaving insufficient buffer for inference.
* **On-the-fly Feature Calculation:** Rejected because real-time aggregation of historical data introduces CPU bottlenecks and network overhead within the critical path.

## Consequences
* **Good:** Ensures predictable, low-latency feature retrieval, helping the system meet the 80 ms requirement.
* **Good:** Decouples ML workloads from the primary transactional database.
* **Bad:** Adds operational complexity in maintaining consistency between the Data Lake and Redis.
* **Bad:** Increases infrastructure cost due to high memory requirements.

## Revisit if
Revisit this decision if memory costs grow beyond budget or if alternative storage solutions (e.g., NVMe-backed systems) can provide similar latency at a lower cost.