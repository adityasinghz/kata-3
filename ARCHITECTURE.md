# Architecture - Smart Hospital ICU Management System

## 1. High-Level Architecture
We will use an **Event-Driven Microservices Architecture** to ensure real-time updates and high scalability.

### Key Components:
- **API Gateway (Kong/NGINX):** Single entry point for all clients (Web Dashboard, Mobile App, IoT Gateways).
- **Bed Management Service:** Core logic for bed status, attributes, and inventory.
- **Allocation Service:** Rule engine for assigning beds based on priority and constraints.
- **Notification Service:** Handles multi-channel alerts (SMS, Email, Push, Pager).
- **IoT Ingestion Service:** Receives high-frequency telemetry from medical devices (MQTT/CoAP).
- **Analytics Service:** Aggregates data for reporting and prediction.

## 2. Technology Stack
- **Frontend:** React.js (Web), React Native (Mobile), D3.js (Hospital Map Visualization).
- **Backend:** Go (for high-throughput Microservices), Node.js (for BFF/API Layer).
- **Messaging:** Apache Kafka (Event Backbone) + Redis Pub/Sub (Real-time UI updates).
- **Database:** PostgreSQL (Core Relational Data), MongoDB (Unstructured Audit Logs/Metadata), InfluxDB (IoT Time-series).
- **Infrastructure:** Kubernetes (Container Orchestration), Docker.

## 3. Communication Patterns
- **Synchronous (REST/gRPC):** For direct user actions (e.g., "Reserve Bed Request" from UI).
- **Asynchronous (Events):** For side effects (e.g., `BedReservedEvent` -> triggers `NotificationService` and `UpdateAnalytics`).
- **Real-Time (WebSockets):** For pushing state changes to the Dashboard.

## 4. Guiding Patterns & Principles

### A. CQRS (Command Query Responsibility Segregation)
- **Why:** Separates the write model (Allocating a Bed) from the read model (Viewing Dashboard).
- **Benefit:** Allows us to scale the "Read" side accurately (which has 100x more traffic) independently of the critical "Write" side.

### B. Event Sourcing
- **Why:** We need a perfect audit trail of every status change a bed has gone through.
- **Benefit:** We can replay events to reconstruct the state of the hospital at any past point in time for audit/legal reasons.

### C. Saga Pattern
- **Why:** Distribute transactions across services (e.g., ADT Admission + Bed Reservation + Notification).
- **Benefit:** Ensures data consistency without tight coupling. If Notification fails, we can retry without rolling back the Reservation.

### D. Optimistic Concurrency Control
- **Why:** Two nurses might try to book the same bed simultaneously.
- **Benefit:** Prevents overbooking. The second request fails with a "Bed no longer available" message prompting a refresh.

### E. Circuit Breaker
- **Why:** External dependencies (like legacy EMR) might go down.
- **Benefit:** Prevents cascading failures. If EMR is down, queue updates locally and retry later, keeping the Bed System operational.
