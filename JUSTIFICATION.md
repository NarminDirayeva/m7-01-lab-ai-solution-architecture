## Serving Pattern Selection  
I chose an **online serving pattern with synchronous request-response inference**. The scenario clearly requires that each transaction must be scored **before the user sees confirmation**, with a strict **p95 latency constraint of 80 ms**. Because of this, batch processing is not applicable, and a purely streaming-based approach would not meet the need for immediate decisions.

In my design, the API Gateway forwards incoming requests to a Fraud Scoring Service. This service retrieves features from an **Online Feature Store (Redis)** to ensure low-latency access, then performs inference using a **Triton model server**, and finally passes the score to a Decision Engine that determines whether to allow, block, or step up authentication.

## Inference Location  
I decided to run inference fully in the **cloud (centralized microservice architecture)** instead of on the edge. Fraud detection models must operate in a **secure and controlled environment**, since exposing them on edge devices increases the risk of reverse engineering.

Also, the model depends on **global features** such as account history and device-level signals, which cannot be reliably maintained on edge devices. Running inference in the cloud ensures consistency, security, and easier model updates.

## Latency, Throughput, and Cost Targets  

- **Optimize: Latency**  
  The system must maintain **p95 latency under 80 ms**, because the user is actively waiting for transaction confirmation. Any delay directly impacts user experience and can lead to failed or abandoned payments.

- **Optimize: Throughput**  
  The system is designed to handle approximately **300 transactions per second at peak**, using horizontally scalable stateless services.

- **Budget: Cost**  
  Cost is treated as a constraint. To meet strict latency and throughput requirements, I accept the overhead of **always-on infrastructure**, including in-memory feature stores and dedicated inference services.

## Fallback Strategy  
I implemented a **rule-based fallback mechanism** inside the Decision Engine. If the Fraud Scoring Service does not return a score within a **45 ms timeout**, the system bypasses the ML model.

In that case, the system evaluates a set of predefined rules such as transaction velocity or high-risk patterns. Instead of blocking transactions aggressively, it prefers **step-up authentication (e.g., MFA)** for uncertain cases, balancing security with user experience.

All fallback decisions are logged and fed back into the monitoring pipeline, allowing the system to continuously improve through retraining.