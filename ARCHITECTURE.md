# LearnCraft - Architecture Documentation

> **⚠️ Core Requirements**: This architecture is designed around 8 critical requirements. See [KEY_REQUIREMENTS.md](./KEY_REQUIREMENTS.md) for the complete reference.

## Table of Contents
1. [High-Level Design (HLD)](#high-level-design-hld)
2. [Requirements-Driven Architecture](#requirements-driven-architecture)
3. [Low-Level Design (LLD)](#low-level-design-lld)
4. [Design Patterns](#design-patterns)
5. [System Components](#system-components)
6. [Data Flow](#data-flow)
7. [Security Considerations](#security-considerations)
8. [Scalability & Performance](#scalability--performance)
9. [Monitoring & Observability](#monitoring--observability)

---

## High-Level Design (HLD)

### System Overview

LearnCraft is a **cloud-native online learning platform** designed to deliver video tutorials combined with browser-based hands-on labs, powered by AI-driven learning assistance and personalized recommendations.

### Architecture Principles

1. **Cloud-Native**: Microservices architecture on Kubernetes for scalability
2. **Event-Driven**: Asynchronous processing for lab provisioning and analytics
3. **Real-Time**: WebSocket for live lab interaction and progress updates
4. **AI-Powered**: LLM integration for learning assistance and content generation
5. **Resilient**: Circuit breakers, retry mechanisms, graceful degradation
6. **Secure**: Isolated lab environments, encrypted data, RBAC

### System Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                        Client Layer                              │
├─────────────────────────────────────────────────────────────────┤
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐         │
│  │ Learner App  │  │ Instructor   │  │ Admin        │         │
│  │ (Web/Mobile) │  │ Portal       │  │ Dashboard    │         │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘         │
│         │                  │                  │                  │
└─────────┼──────────────────┼──────────────────┼─────────────────┘
          │                  │                  │
          │                  │                  │
┌─────────┼──────────────────┼──────────────────┼─────────────────┐
│         │                  │                  │                  │
│  ┌──────▼──────────────────▼──────────────────▼──────┐         │
│  │           API Gateway / Load Balancer              │         │
│  │    (Authentication, Rate Limiting, Routing)       │         │
│  └──────┬──────────────────┬──────────────────┬──────┘         │
│         │                  │                  │                  │
│  ┌──────▼──────────────────▼──────────────────▼──────┐         │
│  │              Microservices Layer                   │         │
│  ├───────────────────────────────────────────────────┤         │
│  │ ┌────────────┐ ┌────────────┐ ┌────────────┐    │         │
│  │ │ User       │ │ Course     │ │ Content    │    │         │
│  │ │ Service    │ │ Service    │ │ Service    │    │         │
│  │ └────────────┘ └────────────┘ └────────────┘    │         │
│  │ ┌────────────┐ ┌────────────┐ ┌────────────┐    │         │
│  │ │ Lab        │ │ AI         │ │ Progress   │    │         │
│  │ │ Service    │ │ Service    │ │ Service    │    │         │
│  │ └────────────┘ └────────────┘ └────────────┘    │         │
│  │ ┌────────────┐ ┌────────────┐ ┌────────────┐    │         │
│  │ │ Assessment │ │ Analytics  │ │ Notification│    │         │
│  │ │ Service    │ │ Service    │ │ Service    │    │         │
│  │ └────────────┘ └────────────┘ └────────────┘    │         │
│  │ ┌────────────┐ ┌────────────┐ ┌────────────┐    │         │
│  │ │ Payment    │ │ Certificate│ │ Search     │    │         │
│  │ │ Service    │ │ Service    │ │ Service    │    │         │
│  │ └────────────┘ └────────────┘ └────────────┘    │         │
│  └──────┬──────────────────┬──────────────────┬──────┘         │
│         │                  │                  │                  │
│  ┌──────▼──────────────────▼──────────────────▼──────┐         │
│  │         Message Queue (Event Bus)                  │         │
│  │       (Kafka for async communication)             │         │
│  └──────┬──────────────────┬──────────────────┬──────┘         │
│         │                  │                  │                  │
│  ┌──────▼──────────────────▼──────────────────▼──────┐         │
│  │              Data Layer                            │         │
│  ├───────────────────────────────────────────────────┤         │
│  │ ┌────────────┐ ┌────────────┐ ┌────────────┐    │         │
│  │ │ Primary    │ │ Cache      │ │ Search     │    │         │
│  │ │ Database   │ │ (Redis)    │ │ (Elastic)  │    │         │
│  │ │ (PostgreSQL│ │            │ │            │    │         │
│  │ └────────────┘ └────────────┘ └────────────┘    │         │
│  │ ┌────────────┐ ┌────────────┐                    │         │
│  │ │ Analytics  │ │ Vector DB  │                    │         │
│  │ │ (ClickHouse│ │ (Pinecone) │                    │         │
│  │ └────────────┘ └────────────┘                    │         │
│  └───────────────────────────────────────────────────┘         │
│                                                                  │
│  ┌───────────────────────────────────────────────────┐         │
│  │         Lab Infrastructure Layer                   │         │
│  ├───────────────────────────────────────────────────┤         │
│  │ ┌────────────┐ ┌────────────┐ ┌────────────┐    │         │
│  │ │ Kubernetes │ │ Container  │ │ Resource   │    │         │
│  │ │ Cluster    │ │ Registry   │ │ Manager    │    │         │
│  │ └────────────┘ └────────────┘ └────────────┘    │         │
│  └───────────────────────────────────────────────────┘         │
│                                                                  │
│  ┌───────────────────────────────────────────────────┐         │
│  │         External Services Integration              │         │
│  ├───────────────────────────────────────────────────┤         │
│  │ ┌────────────┐ ┌────────────┐ ┌────────────┐    │         │
│  │ │ CDN        │ │ AI/LLM     │ │ Payment    │    │         │
│  │ │ (CloudFront│ │ (OpenAI)   │ │ Gateway    │    │         │
│  │ └────────────┘ └────────────┘ └────────────┘    │         │
│  │ ┌────────────┐ ┌────────────┐                    │         │
│  │ │ Email/SMS  │ │ Identity   │                    │         │
│  │ │ Provider   │ │ Provider   │                    │         │
│  │ └────────────┘ └────────────┘                    │         │
└─────────────────────────────────────────────────────────────────┘
```

### Key Components

#### 1. **Learner Application (Web/Mobile)**
- **Purpose**: Primary interface for learners
- **Technology**: React (Web), React Native (Mobile)
- **Features**: 
  - Course browsing and enrollment
  - Video player with progress tracking
  - Browser-based lab terminal
  - AI chat assistant
  - Progress dashboard

#### 2. **Instructor Portal**
- **Purpose**: Content creation and management
- **Technology**: React (Web)
- **Features**:
  - Course builder and editor
  - Video upload and management
  - Lab template creation
  - Learner analytics
  - Revenue tracking

#### 3. **Admin Dashboard**
- **Purpose**: Platform administration
- **Technology**: React (Web)
- **Features**:
  - User management
  - Content moderation
  - System monitoring
  - Analytics and reporting
  - Configuration management

#### 4. **User Service**
- **Purpose**: User lifecycle management
- **Technology**: Node.js / Go
- **Features**:
  - Authentication (OAuth 2.0, SSO)
  - User profiles
  - Role management
  - Subscription management
  - Preferences

#### 5. **Course Service**
- **Purpose**: Course structure and catalog management
- **Technology**: Node.js / Go
- **Features**:
  - Course CRUD operations
  - Module and lesson management
  - Catalog search and filtering
  - Course recommendations
  - Enrollment management

#### 6. **Content Service**
- **Purpose**: Media content management
- **Technology**: Node.js / Go
- **Features**:
  - Video upload and transcoding
  - CDN integration
  - Streaming URL generation
  - DRM protection
  - Subtitle management

#### 7. **Lab Service**
- **Purpose**: Lab environment orchestration
- **Technology**: Go
- **Features**:
  - Environment provisioning
  - Session management
  - Resource allocation
  - Environment reset
  - Multi-stack support

#### 8. **AI Service**
- **Purpose**: AI-powered learning assistance
- **Technology**: Python / Node.js
- **Features**:
  - Video summarization
  - Contextual hints
  - Q&A assistance
  - Recommendations
  - Content generation

#### 9. **Progress Service**
- **Purpose**: Learning progress tracking
- **Technology**: Node.js / Go
- **Features**:
  - Video progress tracking
  - Lab completion tracking
  - Course progress aggregation
  - Streak tracking
  - Achievement management

#### 10. **Assessment Service**
- **Purpose**: Quizzes and skill assessments
- **Technology**: Node.js / Go
- **Features**:
  - Quiz creation and delivery
  - Automated grading
  - Skill scoring
  - Certification exams
  - Analytics

#### 11. **Analytics Service**
- **Purpose**: Business intelligence
- **Technology**: Python
- **Features**:
  - Learner behavior analytics
  - Content performance
  - Revenue analytics
  - Instructor analytics
  - System metrics

#### 12. **Notification Service**
- **Purpose**: Multi-channel notifications
- **Technology**: Node.js
- **Features**:
  - Email notifications
  - Push notifications
  - In-app notifications
  - SMS notifications
  - Notification preferences

---

## Requirements-Driven Architecture

### Mapping Requirements to Architecture

#### Requirement 1: Learning Content Delivery
- **Services**: ContentService, CourseService, ProgressService
- **Components**: VideoPlayer, Transcoder, CDN, ProgressTracker
- **Patterns**: Strategy Pattern (streaming quality), Observer Pattern (progress)
- **Technology**: CloudFront CDN, HLS streaming, FFmpeg

#### Requirement 2: Hands-On Lab Environment
- **Services**: LabService, EnvironmentOrchestrator, ResourceManager
- **Components**: LabProvisioner, SessionManager, TerminalService
- **Patterns**: Factory Pattern (environment types), Pool Pattern (containers)
- **Technology**: Kubernetes, Docker, code-server, WebSocket

#### Requirement 3: AI-Driven Learning Assistance
- **Services**: AIService, RecommendationEngine, ContextAnalyzer
- **Components**: SummaryGenerator, HintEngine, ChatBot, RAG Pipeline
- **Patterns**: Strategy Pattern (AI models), Chain of Responsibility
- **Technology**: OpenAI API, LangChain, Pinecone, Redis

#### Requirement 4: Scalability & Reliability
- **Services**: All services designed for horizontal scaling
- **Components**: LoadBalancer, AutoScaler, CircuitBreaker, HealthChecker
- **Patterns**: Circuit Breaker, Bulkhead, Retry, CQRS
- **Technology**: Kubernetes HPA, Redis, CDN, Database replication

#### Requirement 5: User Experience & Accessibility
- **Services**: UIService, LocalizationService
- **Components**: ResponsiveRenderer, LanguageManager, ThemeManager
- **Patterns**: Adapter Pattern (devices), Strategy Pattern (themes)
- **Technology**: React, PWA, i18next, ARIA

#### Requirement 6: Security & Data Protection
- **Services**: AuthService, SecurityService, AuditService
- **Components**: IdentityManager, AccessController, EncryptionService
- **Patterns**: Decorator Pattern (security), Proxy Pattern (access)
- **Technology**: OAuth 2.0, JWT, TLS, HashiCorp Vault

#### Requirement 7: Progress Tracking & Analytics
- **Services**: ProgressService, AnalyticsService, ReportingService
- **Components**: ProgressTracker, EventCollector, ReportGenerator
- **Patterns**: Observer Pattern (tracking), CQRS (analytics)
- **Technology**: PostgreSQL, ClickHouse, Redis

#### Requirement 8: Content Management
- **Services**: ContentManagementService, MediaService, WorkflowService
- **Components**: CourseBuilder, MediaProcessor, VersionManager
- **Patterns**: Builder Pattern (course creation), State Pattern (publishing)
- **Technology**: S3, FFmpeg, Git-based versioning

---

## Low-Level Design (LLD)

### Lab Service - Detailed Design

**Class Diagram:**
```
LabService
├── LabProvisioner
│   ├── +provisionLab(labId: string, userId: string): LabSession
│   ├── +deprovisionLab(sessionId: string): void
│   └── +resetLab(sessionId: string): LabSession
├── SessionManager
│   ├── +createSession(userId: string, labId: string): Session
│   ├── +getSession(sessionId: string): Session
│   ├── +extendSession(sessionId: string, minutes: number): void
│   └── +terminateSession(sessionId: string): void
└── ResourceManager
    ├── +allocateResources(sessionId: string, spec: ResourceSpec): void
    ├── +releaseResources(sessionId: string): void
    └── +monitorResources(sessionId: string): ResourceMetrics
```

**Key Methods:**
- `provisionLab(labId, userId)`: Creates isolated container environment for learner
- `resetLab(sessionId)`: Destroys current container and creates fresh instance
- `allocateResources(sessionId, spec)`: Sets CPU, memory, storage limits

**Database Schema:**
```sql
CREATE TABLE labs (
    id UUID PRIMARY KEY,
    course_id UUID NOT NULL,
    title VARCHAR(255) NOT NULL,
    description TEXT,
    template_id UUID NOT NULL,
    duration_minutes INT DEFAULT 60,
    tech_stack VARCHAR(50)[],
    difficulty_level VARCHAR(20),
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE lab_sessions (
    id UUID PRIMARY KEY,
    lab_id UUID NOT NULL,
    user_id UUID NOT NULL,
    status VARCHAR(20) DEFAULT 'PENDING',
    container_id VARCHAR(100),
    started_at TIMESTAMP,
    expires_at TIMESTAMP,
    terminal_url TEXT,
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE lab_templates (
    id UUID PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    base_image VARCHAR(255) NOT NULL,
    init_script TEXT,
    resource_limits JSONB,
    ports INT[],
    created_at TIMESTAMP DEFAULT NOW()
);
```

### AI Service - Detailed Design

**Class Diagram:**
```
AIService
├── SummaryGenerator
│   ├── +generateVideoSummary(videoId: string): Summary
│   ├── +generateLabSummary(sessionId: string): Summary
│   └── +extractKeyPoints(content: string): KeyPoint[]
├── HintEngine
│   ├── +getContextualHint(sessionId: string, context: Context): Hint
│   ├── +getTroubleshootingGuide(error: Error): Guide
│   └── +getNextStepSuggestion(state: LabState): Suggestion
├── ChatBot
│   ├── +processQuery(query: string, context: Context): Response
│   ├── +getConversationHistory(sessionId: string): Message[]
│   └── +clearContext(sessionId: string): void
└── RecommendationEngine
    ├── +getPersonalizedCourses(userId: string): Course[]
    ├── +getNextLessons(userId: string): Lesson[]
    └── +getSkillGapAnalysis(userId: string): SkillGap[]
```

**Key Methods:**
- `generateVideoSummary(videoId)`: Uses LLM to create video summary from transcript
- `getContextualHint(sessionId, context)`: Analyzes lab state and gives relevant hints
- `processQuery(query, context)`: RAG-based Q&A using course content

**Database Schema:**
```sql
CREATE TABLE ai_interactions (
    id UUID PRIMARY KEY,
    user_id UUID NOT NULL,
    session_id UUID,
    interaction_type VARCHAR(50), -- 'query', 'hint', 'summary'
    input_text TEXT,
    output_text TEXT,
    context JSONB,
    feedback_rating INT,
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE content_embeddings (
    id UUID PRIMARY KEY,
    content_id UUID NOT NULL,
    content_type VARCHAR(50), -- 'video', 'lesson', 'lab'
    chunk_index INT,
    embedding VECTOR(1536),
    chunk_text TEXT,
    created_at TIMESTAMP DEFAULT NOW()
);
```

### Content Service - Detailed Design

**Class Diagram:**
```
ContentService
├── VideoManager
│   ├── +uploadVideo(file: File, metadata: VideoMeta): Video
│   ├── +getStreamingUrl(videoId: string, quality: string): string
│   ├── +processVideo(videoId: string): ProcessingJob
│   └── +generateSubtitles(videoId: string): Subtitle[]
├── TranscodingService
│   ├── +transcode(videoId: string, formats: Format[]): Job
│   ├── +getJobStatus(jobId: string): JobStatus
│   └── +generateThumbnails(videoId: string): Thumbnail[]
└── CDNManager
    ├── +invalidateCache(paths: string[]): void
    ├── +getSignedUrl(path: string, expiry: number): string
    └── +uploadToCDN(file: File, path: string): string
```

**Database Schema:**
```sql
CREATE TABLE videos (
    id UUID PRIMARY KEY,
    lesson_id UUID NOT NULL,
    title VARCHAR(255) NOT NULL,
    duration_seconds INT,
    original_url TEXT,
    hls_url TEXT,
    thumbnail_url TEXT,
    status VARCHAR(20) DEFAULT 'PROCESSING',
    transcript TEXT,
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE video_qualities (
    id UUID PRIMARY KEY,
    video_id UUID NOT NULL,
    quality VARCHAR(20), -- '360p', '720p', '1080p'
    bitrate INT,
    url TEXT,
    created_at TIMESTAMP DEFAULT NOW()
);
```

---

## Design Patterns

### 1. Factory Pattern
**Use Case**: Lab environment creation
**Implementation**: Different factories for different tech stacks (Python, Node.js, Go, etc.)
**Example**: `LabEnvironmentFactory` creates appropriate container based on lab template

### 2. Strategy Pattern
**Use Case**: Video streaming quality, AI model selection
**Implementation**: Interchangeable algorithms for different contexts
**Example**: `StreamingStrategy` switches between qualities based on network

### 3. Observer Pattern
**Use Case**: Progress tracking, real-time notifications
**Implementation**: Services subscribe to events and react accordingly
**Example**: `ProgressObserver` listens to video/lab completion events

### 4. Circuit Breaker Pattern
**Use Case**: External service calls (AI API, payment gateway)
**Implementation**: Prevents cascading failures when external services are down
**Example**: `AIServiceCircuitBreaker` opens circuit after N failures

### 5. Pool Pattern
**Use Case**: Pre-warmed lab containers
**Implementation**: Maintain pool of ready-to-use containers for fast provisioning
**Example**: `ContainerPool` maintains warm containers per tech stack

### 6. CQRS Pattern
**Use Case**: Analytics queries, progress reporting
**Implementation**: Separate read models for complex queries
**Example**: Write to PostgreSQL, read from ClickHouse for analytics

### 7. Event Sourcing Pattern
**Use Case**: Learning activity history, audit trail
**Implementation**: Store all events for replay and analysis
**Example**: `LearningEventStore` stores all learner actions

### 8. Builder Pattern
**Use Case**: Course creation, lab template building
**Implementation**: Step-by-step construction of complex objects
**Example**: `CourseBuilder` constructs course with modules, lessons, labs

### 9. Proxy Pattern
**Use Case**: Access control, caching
**Implementation**: Control access to services/resources
**Example**: `LabAccessProxy` validates subscription before lab access

### 10. Decorator Pattern
**Use Case**: Logging, caching, security wrapping
**Implementation**: Add functionality without modifying core logic
**Example**: `CachingDecorator` wraps AI service for response caching

### 11. Adapter Pattern
**Use Case**: Third-party integrations
**Implementation**: Unified interface for different providers
**Example**: `PaymentAdapter` adapts Stripe/PayPal to common interface

### 12. State Pattern
**Use Case**: Lab session states, course publishing workflow
**Implementation**: State-specific behavior management
**Example**: `LabSessionStateMachine` manages session lifecycle

---

## System Components

### API Gateway
- **Purpose**: Single entry point for all requests
- **Features**: Authentication, rate limiting, routing, request transformation
- **Technology**: Kong / AWS API Gateway / Nginx

### Service Mesh
- **Purpose**: Service-to-service communication
- **Features**: mTLS, circuit breakers, retries, observability
- **Technology**: Istio / Linkerd

### Message Queue
- **Purpose**: Asynchronous communication
- **Features**: Event streaming, pub/sub, guaranteed delivery
- **Technology**: Kafka

### Cache Layer
- **Purpose**: Fast data access
- **Features**: Session cache, content cache, rate limiting
- **Technology**: Redis Cluster

### Search Engine
- **Purpose**: Course and content search
- **Features**: Full-text search, faceted search, autocomplete
- **Technology**: Elasticsearch

### Vector Database
- **Purpose**: AI similarity search
- **Features**: Embedding storage, semantic search
- **Technology**: Pinecone / Weaviate

---

## Data Flow

### Video Streaming Flow
```
1. Learner App
   ↓ (GET /api/videos/{id}/stream)
2. API Gateway
   ↓ (authenticate, validate subscription)
3. Content Service
   ↓ (get streaming URL)
4. CDN (CloudFront)
   ↓ (serve HLS segments)
5. Learner App
   ↓ (report progress)
6. Progress Service
   ↓ (update progress)
7. Event Bus (Kafka)
   ↓ (progress.updated event)
8. Analytics Service
```

### Lab Session Flow
```
1. Learner App
   ↓ (POST /api/labs/{id}/start)
2. API Gateway
   ↓ (authenticate)
3. Lab Service
   ↓ (check session limits)
4. Container Pool
   ↓ (get warm container)
5. Resource Manager
   ↓ (allocate resources)
6. Kubernetes
   ↓ (assign to container)
7. Lab Service
   ↓ (generate terminal URL)
8. Learner App
   ↓ (WebSocket connection)
9. Terminal Service
   ↓ (interactive session)
```

### AI Assistance Flow
```
1. Learner App
   ↓ (POST /api/ai/query)
2. API Gateway
   ↓ (authenticate, rate limit)
3. AI Service
   ↓ (get context from session)
4. Vector Database
   ↓ (semantic search for relevant content)
5. AI Service
   ↓ (build RAG prompt)
6. OpenAI API
   ↓ (generate response)
7. AI Service
   ↓ (post-process, log interaction)
8. Learner App
   ↓ (display response)
```

---

## Security Considerations

### Lab Isolation
- **Network Isolation**: Each lab runs in isolated network namespace
- **Process Isolation**: Container-level process isolation
- **Resource Limits**: CPU, memory, storage quotas per session
- **Egress Control**: Whitelist-only outbound network access
- **Session Timeout**: Automatic termination after inactivity

### Authentication & Authorization
- **OAuth 2.0**: For third-party authentication
- **JWT**: For API authentication
- **RBAC**: Role-based access control
- **SSO**: Enterprise single sign-on support
- **MFA**: Optional multi-factor authentication

### Data Protection
- **Encryption at Rest**: AES-256 for stored data
- **Encryption in Transit**: TLS 1.3 for all traffic
- **DRM**: Video content protection
- **PII Handling**: GDPR-compliant data handling
- **Audit Logging**: Complete audit trail

---

## Scalability & Performance

### Horizontal Scaling
- **Kubernetes HPA**: Auto-scale based on CPU/memory
- **Lab Pool Scaling**: Dynamic container pool sizing
- **Database Sharding**: User-based sharding for scale

### Caching Strategy
- **CDN**: Video content edge caching
- **Redis**: Session, progress, frequent queries
- **Application Cache**: In-memory for hot data

### Performance Targets
- **Video Startup**: < 2 seconds
- **Lab Startup**: < 5 seconds
- **API Response**: < 200ms (p95)
- **AI Response**: < 3 seconds
- **Concurrent Labs**: 100K+ sessions

---

## Monitoring & Observability

### Metrics
- **Application**: Request rate, error rate, latency
- **Business**: Active users, lab sessions, course completions
- **Infrastructure**: CPU, memory, network, storage
- **Technology**: Prometheus, Grafana

### Logging
- **Application Logs**: Structured JSON logging
- **Audit Logs**: Security and access events
- **Technology**: ELK Stack (Elasticsearch, Logstash, Kibana)

### Tracing
- **Distributed Tracing**: Request flow across services
- **Technology**: Jaeger / Zipkin

### Alerting
- **Critical Alerts**: < 1 minute response
- **Warning Alerts**: < 5 minute response
- **Technology**: PagerDuty / Alertmanager

---

**Last Updated**: January 2026
**Version**: 1.0
**Status**: Design Complete, Implementation Pending
