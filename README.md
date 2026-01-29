# Smart Hospital ICU Management System - Design Kata

## Overview
This design kata models a **High-Availability, Real-Time ICU Management System** inspired by a hybrid of China's high-volume integrated models and the US's patient-centric interoperability standards. The goal is to solve the critical "Bed Availability" problem through automated allocation, predictive analytics, and real-time visualization.

## Documentation Index

### 1. Requirements & Business Analysis
- **[KEY_REQUIREMENTS.md](./KEY_REQUIREMENTS.md)**: 8 Core requirements (Real-time dashboard, Priority allocation, etc.).
- **[BUSINESS_PERSPECTIVE.md](./BUSINESS_PERSPECTIVE.md)**: Market analysis, ROI models, and value proposition.
- **[ACTORS_AND_USE_CASES.md](./ACTORS_AND_USE_CASES.md)**: Primary actors (Staff, Doctors) and detailed use case flows.
- **[FEATURES.md](./FEATURES.md)**: Detailed feature specifications (Interactive Map, Smart Notification Hub).

### 2. Technical Architecture
- **[ARCHITECTURE.md](./ARCHITECTURE.md)**: Event-Driven Microservices design, tech stack, and guiding patterns (CQRS, Saga).
- **[CLASS_DIAGRAM.md](./CLASS_DIAGRAM.md)**: Domain model showing relationships between Hospital, Ward, Bed, and Patient.
- **[DATABASE_SCHEMA.md](./DATABASE_SCHEMA.md)**: Physical data model including Table definitions and Indexes.
- **[SEQUENCE_DIAGRAMS.md](./SEQUENCE_DIAGRAMS.md)**: Sequence flows for Admission, Emergency Override, and Discharge.
- **[API_SPECIFICATION.md](./API_SPECIFICATION.md)**: RESTful API definitions for Bed Management and Reservations.

## Key Design Highlights
- **Real-Time Visualization:** WebSocket-driven dashboard for "Air Traffic Control" view of beds.
- **Priority Algorithm:** Automated reasoning to prioritize critical patients (Code Blue) over elective ones.
- **IoT Integration:** "Smart Bed" status updates based on telemetry; auto-transfer suggestions based on acuity.
- **Resilience:** Circuit breakers for legacy HIS integration; Offline-first capabilities for critical admission paths.

## Status
- **Version:** 1.0
- **Status:** Design Complete
- **Last Updated:** January 2026
