# LearnCraft Online Learning Platform - Documentation

## Overview

LearnCraft is a **modern online learning platform** designed to deliver high-quality video tutorials combined with live, hands-on browser-based labs, powered by AI-driven insights and personalized learning assistance.

## Documentation Structure

This repository contains comprehensive documentation for the LearnCraft Online Learning Platform.

### Core Documents

1. **[KEY_REQUIREMENTS.md](./KEY_REQUIREMENTS.md)**
   - 8 core requirements that form the foundation
   - Mapping to architecture, features, and business impact
   - Success criteria and assumptions

2. **[ARCHITECTURE.md](./ARCHITECTURE.md)**
   - High-Level Design (HLD)
   - Low-Level Design (LLD)
   - Design Patterns (12 patterns)
   - System Components
   - Data Flow
   - Security, Scalability, Monitoring

3. **[FEATURES.md](./FEATURES.md)**
   - Detailed feature specifications (F1-F8)
   - User stories and acceptance criteria
   - Technical specifications
   - Feature dependencies and prioritization
   - Implementation roadmap

4. **[BUSINESS_PERSPECTIVE.md](./BUSINESS_PERSPECTIVE.md)**
   - Business objectives and market analysis
   - Value proposition
   - Revenue model (4 revenue streams)
   - Go-to-market strategy
   - Financial projections

### Design Documents

5. **[ACTORS_AND_USE_CASES.md](./ACTORS_AND_USE_CASES.md)**
   - 5 actors (Learner, Instructor, Content Admin, etc.)
   - 30+ use cases with detailed flows
   - Use case relationships (include, extend, generalization)

6. **[SEQUENCE_DIAGRAMS.md](./SEQUENCE_DIAGRAMS.md)**
   - Course enrollment flow
   - Video streaming flow
   - Lab session lifecycle
   - AI assistance request flow
   - Progress tracking flow

7. **[CLASS_DIAGRAM.md](./CLASS_DIAGRAM.md)**
   - Core domain classes (Course, Lab, User, etc.)
   - Service layer classes
   - Complete class diagram
   - Class relationships

8. **[DATABASE_SCHEMA.md](./DATABASE_SCHEMA.md)**
   - Entity Relationship Diagram (ERD)
   - 20+ database tables with SQL schemas
   - Indexes and relationships
   - Data types and constraints

9. **[API_SPECIFICATION.md](./API_SPECIFICATION.md)**
   - Complete REST API documentation
   - 8 API categories (Course, Lab, User, AI, etc.)
   - Request/response formats
   - Authentication and rate limiting
   - Webhooks

## Quick Start Guide

### For Architects
1. Start with **[KEY_REQUIREMENTS.md](./KEY_REQUIREMENTS.md)** to understand core requirements
2. Review **[ARCHITECTURE.md](./ARCHITECTURE.md)** for system design
3. Check design patterns and technology stack

### For Developers
1. Read **[FEATURES.md](./FEATURES.md)** for feature specifications
2. Review **[API_SPECIFICATION.md](./API_SPECIFICATION.md)** for API details
3. Check **[DATABASE_SCHEMA.md](./DATABASE_SCHEMA.md)** for database design
4. Refer to **[CLASS_DIAGRAM.md](./CLASS_DIAGRAM.md)** for class structure

### For Business Analysts
1. Start with **[BUSINESS_PERSPECTIVE.md](./BUSINESS_PERSPECTIVE.md)** for business model
2. Review **[KEY_REQUIREMENTS.md](./KEY_REQUIREMENTS.md)** for requirements
3. Check **[FEATURES.md](./FEATURES.md)** for feature details

### For Testers
1. Review **[ACTORS_AND_USE_CASES.md](./ACTORS_AND_USE_CASES.md)** for test scenarios
2. Check **[SEQUENCE_DIAGRAMS.md](./SEQUENCE_DIAGRAMS.md)** for flow validation
3. Refer to **[API_SPECIFICATION.md](./API_SPECIFICATION.md)** for API testing

## Key Highlights

### Core Requirements (8)
1. Learning Content Delivery (Video + AI Summaries)
2. Hands-On Lab Environment (Browser-based, Isolated)
3. AI-Driven Learning Assistance
4. Scalability & Reliability
5. User Experience & Accessibility
6. Security & Data Protection
7. Progress Tracking & Analytics
8. Content Management

### Architecture Highlights
- **Microservices Architecture**: 15+ services
- **Cloud-Native**: Kubernetes, auto-scaling
- **Event-Driven**: Kafka for async processing
- **Real-Time**: WebSocket for labs, Redis for caching
- **AI-Powered**: LLM integration for assistance
- **Design Patterns**: 12 patterns (Strategy, Factory, Circuit Breaker, etc.)

### Technology Stack
- **Frontend**: React (Web), WebSocket (real-time)
- **Backend**: Node.js/Go (microservices)
- **Lab Infrastructure**: Kubernetes, Docker, code-server
- **AI/ML**: OpenAI API, LangChain, Vector DB
- **Database**: PostgreSQL (primary), Redis (cache), ClickHouse (analytics)
- **Streaming**: CDN (CloudFront), HLS for video
- **Message Queue**: Kafka

### Business Model
- **Revenue Streams**: Subscriptions (60%), Enterprise (25%), Certifications (10%), Marketplace (5%)
- **Year 1 Target**: 50K MAU, 500+ courses, $2M ARR
- **Year 3 Target**: 500K MAU, 2000+ courses, profitability

## Document Status

| Document | Status | Version | Last Updated |
|----------|--------|---------|--------------|
| KEY_REQUIREMENTS.md | ✅ Complete | 1.0 | Jan 2026 |
| ARCHITECTURE.md | ✅ Complete | 1.0 | Jan 2026 |
| FEATURES.md | ✅ Complete | 1.0 | Jan 2026 |
| BUSINESS_PERSPECTIVE.md | ✅ Complete | 1.0 | Jan 2026 |
| ACTORS_AND_USE_CASES.md | ✅ Complete | 1.0 | Jan 2026 |
| SEQUENCE_DIAGRAMS.md | ✅ Complete | 1.0 | Jan 2026 |
| CLASS_DIAGRAM.md | ✅ Complete | 1.0 | Jan 2026 |
| DATABASE_SCHEMA.md | ✅ Complete | 1.0 | Jan 2026 |
| API_SPECIFICATION.md | ✅ Complete | 1.0 | Jan 2026 |

## Next Steps

1. **Review Documentation**: Go through all documents to understand the complete system
2. **Clarify Questions**: Identify any ambiguities or missing requirements
3. **Create Implementation Plan**: Break down into sprints/phases
4. **Set Up Infrastructure**: Prepare cloud environment, Kubernetes cluster
5. **Start Development**: Begin with MVP features (Phase 1)

---

**Last Updated**: January 2026  
**Version**: 1.0  
**Status**: Design Complete, Implementation Pending
