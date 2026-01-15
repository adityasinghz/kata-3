# GramSeva Health - Architecture Documentation

> **⚠️ Core Requirements**: This architecture is designed around 5 critical requirements. See [KEY_REQUIREMENTS.md](./KEY_REQUIREMENTS.md) for the complete reference.

## Table of Contents
1. [High-Level Design (HLD)](#high-level-design-hld)
2. [Low-Level Design (LLD)](#low-level-design-lld)
3. [Design Patterns](#design-patterns)
4. [System Components](#system-components)
5. [Data Flow](#data-flow)

---

## High-Level Design (HLD)

### System Overview

GramSeva Health is a **rural telemedicine platform** designed to bridge the healthcare gap in Tier-2/3 cities and villages through an assisted kiosk model with offline-first capabilities.

### Architecture Principles

1. **Offline-First Architecture**: Store-and-forward pattern for unreliable connectivity
2. **Low-Bandwidth Optimization**: Video compression, adaptive streaming, data prioritization
3. **Assisted Model**: Sahayak (health worker) as intermediary between patient and doctor
4. **Privacy & Compliance**: DPDP Act compliance with role-based access control
5. **Scalability**: Microservices architecture for horizontal scaling
6. **Resilience**: Circuit breakers, retry mechanisms, graceful degradation

### System Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                        Client Layer                              │
├─────────────────────────────────────────────────────────────────┤
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐         │
│  │ Kiosk App    │  │ Sahayak App  │  │ Doctor App   │         │
│  │ (Patient UI) │  │ (Assisted)   │  │ (Desktop)    │         │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘         │
│         │                  │                  │                  │
└─────────┼──────────────────┼──────────────────┼─────────────────┘
          │                  │                  │
          │                  │                  │
┌─────────┼──────────────────┼──────────────────┼─────────────────┐
│         │                  │                  │                  │
│  ┌──────▼──────────────────▼──────────────────▼──────┐         │
│  │           API Gateway / Load Balancer              │         │
│  │         (Authentication, Rate Limiting)            │         │
│  └──────┬──────────────────┬──────────────────┬──────┘         │
│         │                  │                  │                  │
│  ┌──────▼──────────────────▼──────────────────▼──────┐         │
│  │              Microservices Layer                   │         │
│  ├───────────────────────────────────────────────────┤         │
│  │ ┌────────────┐ ┌────────────┐ ┌────────────┐    │         │
│  │ │ Tele-      │ │ AI Triage  │ │ Prescription│   │         │
│  │ │ Consultation│ │ & Translation│ │ & Fulfillment│ │         │
│  │ │ Service    │ │ Service    │ │ Service    │    │         │
│  │ └────────────┘ └────────────┘ └────────────┘    │         │
│  │ ┌────────────┐ ┌────────────┐ ┌────────────┐    │         │
│  │ │ Patient    │ │ Consent    │ │ Offline    │    │         │
│  │ │ Management │ │ & Privacy  │ │ Sync       │    │         │
│  │ │ Service    │ │ Service    │ │ Service    │    │         │
│  │ └────────────┘ └────────────┘ └────────────┘    │         │
│  │ ┌────────────┐ ┌────────────┐ ┌────────────┐    │         │
│  │ │ Visual     │ │ Govt       │ │ WhatsApp   │    │         │
│  │ │ Diagnosis  │ │ Scheme     │ │ Bot        │    │         │
│  │ │ Service    │ │ Integration│ │ Service    │    │         │
│  │ └────────────┘ └────────────┘ └────────────┘    │         │
│  └──────┬──────────────────┬──────────────────┬──────┘         │
│         │                  │                  │                  │
│  ┌──────▼──────────────────▼──────────────────▼──────┐         │
│  │         Message Queue (Event Bus)                  │         │
│  │    (Kafka/RabbitMQ for async communication)       │         │
│  └──────┬──────────────────┬──────────────────┬──────┘         │
│         │                  │                  │                  │
│  ┌──────▼──────────────────▼──────────────────▼──────┐         │
│  │              Data Layer                            │         │
│  ├───────────────────────────────────────────────────┤         │
│  │ ┌────────────┐ ┌────────────┐ ┌────────────┐    │         │
│  │ │ Primary    │ │ Offline    │ │ Cache      │    │         │
│  │ │ Database   │ │ Local DB   │ │ (Redis)    │    │         │
│  │ │ (PostgreSQL│ │ (SQLite)   │ │            │    │         │
│  │ │ /MongoDB)  │ │            │ │            │    │         │
│  │ └────────────┘ └────────────┘ └────────────┘    │         │
│  └───────────────────────────────────────────────────┘         │
│                                                                  │
│  ┌───────────────────────────────────────────────────┐         │
│  │         External Services Integration              │         │
│  ├───────────────────────────────────────────────────┤         │
│  │ ┌────────────┐ ┌────────────┐ ┌────────────┐    │         │
│  │ │ Ayushman   │ │ Pharmacy   │ │ IoT Device │    │         │
│  │ │ Bharat API │ │ Supply API │ │ Gateway    │    │         │
│  │ └────────────┘ └────────────┘ └────────────┘    │         │
│  └───────────────────────────────────────────────────┘         │
└─────────────────────────────────────────────────────────────────┘
```

### Key Components

#### 1. **Kiosk Application (Patient Interface)**
- **Purpose**: Patient-facing interface at kiosk locations
- **Technology**: React Native / Flutter (offline-capable)
- **Features**: 
  - Patient registration/login
  - Symptom input (voice/text in local dialect)
  - Video consultation interface
  - Prescription viewing
  - Appointment scheduling

#### 2. **Sahayak Application (Health Worker Interface)**
- **Purpose**: Assisted interface for health workers
- **Technology**: React Native / Flutter
- **Features**:
  - Patient onboarding
  - IoT device integration (BP monitor, Oximeter)
  - Vitals upload
  - Consultation assistance
  - Offline data capture

#### 3. **Doctor Application (Desktop/Web)**
- **Purpose**: Doctor-facing consultation interface
- **Technology**: React / Angular (WebRTC for video)
- **Features**:
  - Patient queue management
  - Video consultation
  - AI-translated symptom summaries
  - Prescription generation
  - Patient history viewing

#### 4. **Tele-Consultation Service**
- **Purpose**: Video consultation orchestration
- **Technology**: WebRTC, Janus/Kurento media server
- **Features**:
  - Low-bandwidth video optimization
  - Multi-party video (Patient, Sahayak, Doctor)
  - Screen sharing
  - Recording (with consent)

#### 5. **AI Triage & Translation Service**
- **Purpose**: Symptom analysis and translation
- **Technology**: LLM (GPT-4/Claude), Speech-to-Text, Translation APIs
- **Features**:
  - Dialect-to-English translation
  - Medical symptom summarization
  - Severity scoring
  - Triage recommendations

#### 6. **Prescription & Fulfillment Service**
- **Purpose**: Prescription management and pharmacy integration
- **Technology**: REST APIs, Pharmacy partner APIs
- **Features**:
  - Digital prescription generation
  - Medicine availability check
  - Order placement
  - Delivery tracking

#### 7. **Offline Sync Service**
- **Purpose**: Store-and-forward data synchronization
- **Technology**: Background sync, Conflict resolution
- **Features**:
  - Local data caching
  - Conflict detection and resolution
  - Incremental sync
  - Sync status tracking

#### 8. **Consent & Privacy Service**
- **Purpose**: DPDP Act compliance and access control
- **Technology**: OAuth 2.0, RBAC, Encryption
- **Features**:
  - Role-based access control
  - Consent management (OTP/Biometric)
  - Data encryption at rest and in transit
  - Audit logging

#### 9. **Visual Diagnosis Service**
- **Purpose**: Computer vision for skin conditions/wounds
- **Technology**: TensorFlow/PyTorch, Image processing
- **Features**:
  - Image upload and analysis
  - Condition classification
  - Severity assessment
  - Report generation

#### 10. **Government Scheme Integration Service**
- **Purpose**: Ayushman Bharat integration
- **Technology**: REST APIs, ABHA ID verification
- **Features**:
  - ABHA ID validation
  - Insurance eligibility check
  - Health record fetching
  - Scheme benefit verification

#### 11. **WhatsApp Bot Service**
- **Purpose**: Follow-up reminders and communication
- **Technology**: WhatsApp Business API, NLP
- **Features**:
  - Medication reminders (vernacular)
  - Appointment scheduling
  - Follow-up queries
  - Health tips

---

## Low-Level Design (LLD)

### 1. Tele-Consultation Service - Detailed Design

#### Component Structure
```
TeleConsultationService
├── ConsultationController
│   ├── createConsultation()
│   ├── joinConsultation()
│   ├── endConsultation()
│   └── getConsultationStatus()
├── MediaManager
│   ├── initializeWebRTC()
│   ├── optimizeBandwidth()
│   ├── handleLowBandwidth()
│   └── recordSession()
├── QueueManager
│   ├── addToQueue()
│   ├── assignDoctor()
│   ├── prioritizeQueue()
│   └── notifyDoctor()
└── NotificationService
    ├── sendNotification()
    └── updateStatus()
```

#### Data Models

**Consultation**
```typescript
interface Consultation {
  id: string;
  patientId: string;
  sahayakId: string;
  doctorId: string | null;
  status: 'QUEUED' | 'IN_PROGRESS' | 'COMPLETED' | 'CANCELLED';
  priority: 'LOW' | 'MEDIUM' | 'HIGH' | 'CRITICAL';
  scheduledAt: Date;
  startedAt: Date | null;
  endedAt: Date | null;
  symptoms: string;
  translatedSymptoms: string;
  severityScore: number;
  videoRoomId: string;
  recordingUrl: string | null;
  consentGiven: boolean;
  createdAt: Date;
  updatedAt: Date;
}
```

**Vitals**
```typescript
interface Vitals {
  id: string;
  consultationId: string;
  patientId: string;
  bloodPressure: { systolic: number; diastolic: number };
  oxygenSaturation: number;
  heartRate: number;
  temperature: number;
  recordedBy: string; // Sahayak ID
  recordedAt: Date;
  deviceType: 'BP_MONITOR' | 'OXIMETER' | 'THERMOMETER';
}
```

#### API Endpoints

```
POST   /api/v1/consultations
GET    /api/v1/consultations/:id
POST   /api/v1/consultations/:id/join
POST   /api/v1/consultations/:id/end
POST   /api/v1/consultations/:id/vitals
GET    /api/v1/consultations/queue
POST   /api/v1/consultations/:id/consent
```

#### Sequence Diagram: Consultation Flow

```
Patient → Sahayak App → TeleConsultation Service → AI Service → Doctor App
   │            │                │                    │              │
   │──Register──>│                │                    │              │
   │            │──Create Consult─>                   │              │
   │            │                │──Translate Symptoms─>              │
   │            │                │<──Translated Summary              │
   │            │                │──Add to Queue───────>             │
   │            │                │                    │              │
   │            │                │                    │──Notify Doctor
   │            │                │                    │              │
   │            │                │<──Doctor Accepts───              │
   │            │                │                    │              │
   │            │──Upload Vitals─>│                    │              │
   │            │                │                    │              │
   │            │<──Video Room───│                    │              │
   │            │                │                    │              │
   │<──Join Room─│                │                    │              │
   │            │                │                    │<──Join Room──
   │            │                │                    │              │
   │──Consultation in Progress───│                    │              │
   │            │                │                    │              │
   │            │                │<──End Consultation─              │
   │            │                │                    │              │
   │<──Prescription──────────────│                    │              │
```

### 2. AI Triage & Translation Service - Detailed Design

#### Component Structure
```
AITriageService
├── TranslationEngine
│   ├── translateDialect()
│   ├── detectLanguage()
│   └── normalizeText()
├── SymptomAnalyzer
│   ├── extractSymptoms()
│   ├── classifySeverity()
│   └── generateSummary()
├── TriageEngine
│   ├── calculateSeverityScore()
│   ├── recommendPriority()
│   └── suggestTriage()
└── LLMService
    ├── callLLM()
    ├── formatPrompt()
    └── parseResponse()
```

#### Data Models

**SymptomInput**
```typescript
interface SymptomInput {
  id: string;
  consultationId: string;
  patientId: string;
  rawText: string;
  audioUrl?: string;
  detectedLanguage: string;
  dialect: string;
  timestamp: Date;
}

interface SymptomOutput {
  id: string;
  symptomInputId: string;
  translatedText: string;
  extractedSymptoms: string[];
  severityScore: number; // 1-10
  priority: 'LOW' | 'MEDIUM' | 'HIGH' | 'CRITICAL';
  medicalSummary: string;
  suggestedTriage: string;
  confidence: number; // 0-1
  processedAt: Date;
}
```

#### Processing Flow

```
1. Receive symptom input (text/audio)
2. Detect language and dialect
3. Translate to English (if needed)
4. Extract medical symptoms using LLM
5. Classify severity using ML model
6. Generate medical summary
7. Calculate triage priority
8. Return structured output
```

### 3. Offline Sync Service - Detailed Design

#### Component Structure
```
OfflineSyncService
├── LocalStorageManager
│   ├── storeLocally()
│   ├── retrieveLocal()
│   └── clearLocal()
├── SyncManager
│   ├── detectConnectivity()
│   ├── queueForSync()
│   ├── syncData()
│   └── handleConflict()
├── ConflictResolver
│   ├── detectConflict()
│   ├── resolveConflict()
│   └── mergeData()
└── SyncStatusTracker
    ├── trackSyncStatus()
    ├── getPendingItems()
    └── retryFailedSync()
```

#### Data Models

**SyncQueue**
```typescript
interface SyncQueueItem {
  id: string;
  entityType: 'CONSULTATION' | 'VITALS' | 'PRESCRIPTION' | 'PATIENT';
  entityId: string;
  operation: 'CREATE' | 'UPDATE' | 'DELETE';
  payload: any;
  deviceId: string;
  timestamp: Date;
  status: 'PENDING' | 'SYNCING' | 'SYNCED' | 'FAILED' | 'CONFLICT';
  retryCount: number;
  lastError?: string;
  syncedAt?: Date;
}
```

#### Sync Strategy

**Store-and-Forward Pattern:**
1. **Write Path**: All writes go to local SQLite first, then queued for sync
2. **Read Path**: Read from local cache, fallback to server if online
3. **Sync Trigger**: 
   - On connectivity restore
   - Periodic background sync (every 5 minutes when online)
   - Manual sync trigger
4. **Conflict Resolution**: Last-write-wins with timestamp comparison, manual resolution for critical conflicts

### 4. Consent & Privacy Service - Detailed Design

#### Component Structure
```
ConsentPrivacyService
├── ConsentManager
│   ├── requestConsent()
│   ├── verifyConsent()
│   ├── revokeConsent()
│   └── getConsentStatus()
├── AccessControl
│   ├── checkPermission()
│   ├── enforceRBAC()
│   └── auditAccess()
├── EncryptionService
│   ├── encryptData()
│   ├── decryptData()
│   └── rotateKeys()
└── AuditLogger
    ├── logAccess()
    ├── logConsent()
    └── generateAuditReport()
```

#### Data Models

**Consent**
```typescript
interface Consent {
  id: string;
  patientId: string;
  consentType: 'DATA_SHARING' | 'RECORDING' | 'PHARMACY_SHARING' | 'GOVT_SCHEME';
  grantedTo: string; // Role or specific user ID
  grantedBy: string; // Patient ID
  verificationMethod: 'OTP' | 'BIOMETRIC' | 'SIGNATURE';
  verificationToken: string;
  status: 'PENDING' | 'GRANTED' | 'REVOKED' | 'EXPIRED';
  grantedAt: Date;
  expiresAt: Date;
  revokedAt: Date | null;
}

interface AccessLog {
  id: string;
  userId: string;
  role: 'SAHAYAK' | 'DOCTOR' | 'ADMIN';
  resourceType: string;
  resourceId: string;
  action: 'READ' | 'WRITE' | 'DELETE';
  ipAddress: string;
  timestamp: Date;
  consentId: string | null;
  success: boolean;
}
```

#### RBAC Model

```
Roles:
- PATIENT: Can view own data, grant consent
- SAHAYAK: Can view active patient data (only during consultation)
- DOCTOR: Can view assigned patient data, generate prescriptions
- ADMIN: Full access (with audit trail)
- PHARMACY: Can view prescription details (with consent)
- GOVT_SCHEME: Can verify eligibility (with consent)
```

### 5. Database Schema Design

#### Core Tables

**patients**
```sql
CREATE TABLE patients (
  id UUID PRIMARY KEY,
  abha_id VARCHAR(50) UNIQUE,
  name VARCHAR(255) NOT NULL,
  phone VARCHAR(20) NOT NULL,
  date_of_birth DATE,
  gender VARCHAR(10),
  address TEXT,
  village VARCHAR(100),
  district VARCHAR(100),
  state VARCHAR(100),
  emergency_contact VARCHAR(20),
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  encrypted_data JSONB -- For sensitive fields
);
```

**consultations**
```sql
CREATE TABLE consultations (
  id UUID PRIMARY KEY,
  patient_id UUID REFERENCES patients(id),
  sahayak_id UUID REFERENCES users(id),
  doctor_id UUID REFERENCES users(id),
  status VARCHAR(20) NOT NULL,
  priority VARCHAR(20),
  severity_score INTEGER,
  symptoms TEXT,
  translated_symptoms TEXT,
  video_room_id VARCHAR(100),
  recording_url TEXT,
  consent_given BOOLEAN DEFAULT FALSE,
  scheduled_at TIMESTAMP,
  started_at TIMESTAMP,
  ended_at TIMESTAMP,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  INDEX idx_patient_id (patient_id),
  INDEX idx_doctor_id (doctor_id),
  INDEX idx_status (status),
  INDEX idx_created_at (created_at)
);
```

**vitals**
```sql
CREATE TABLE vitals (
  id UUID PRIMARY KEY,
  consultation_id UUID REFERENCES consultations(id),
  patient_id UUID REFERENCES patients(id),
  blood_pressure_systolic INTEGER,
  blood_pressure_diastolic INTEGER,
  oxygen_saturation DECIMAL(5,2),
  heart_rate INTEGER,
  temperature DECIMAL(5,2),
  recorded_by UUID REFERENCES users(id),
  device_type VARCHAR(50),
  recorded_at TIMESTAMP DEFAULT NOW(),
  INDEX idx_consultation_id (consultation_id),
  INDEX idx_patient_id (patient_id)
);
```

**prescriptions**
```sql
CREATE TABLE prescriptions (
  id UUID PRIMARY KEY,
  consultation_id UUID REFERENCES consultations(id),
  patient_id UUID REFERENCES patients(id),
  doctor_id UUID REFERENCES users(id),
  medicines JSONB NOT NULL, -- Array of medicine objects
  instructions TEXT,
  follow_up_date DATE,
  status VARCHAR(20) DEFAULT 'PENDING',
  pharmacy_order_id VARCHAR(100),
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  INDEX idx_consultation_id (consultation_id),
  INDEX idx_patient_id (patient_id)
);
```

**consents**
```sql
CREATE TABLE consents (
  id UUID PRIMARY KEY,
  patient_id UUID REFERENCES patients(id),
  consent_type VARCHAR(50) NOT NULL,
  granted_to VARCHAR(100) NOT NULL,
  verification_method VARCHAR(20),
  verification_token VARCHAR(255),
  status VARCHAR(20) NOT NULL,
  granted_at TIMESTAMP,
  expires_at TIMESTAMP,
  revoked_at TIMESTAMP,
  created_at TIMESTAMP DEFAULT NOW(),
  INDEX idx_patient_id (patient_id),
  INDEX idx_status (status)
);
```

**sync_queue**
```sql
CREATE TABLE sync_queue (
  id UUID PRIMARY KEY,
  entity_type VARCHAR(50) NOT NULL,
  entity_id UUID NOT NULL,
  operation VARCHAR(20) NOT NULL,
  payload JSONB NOT NULL,
  device_id VARCHAR(100),
  status VARCHAR(20) DEFAULT 'PENDING',
  retry_count INTEGER DEFAULT 0,
  last_error TEXT,
  created_at TIMESTAMP DEFAULT NOW(),
  synced_at TIMESTAMP,
  INDEX idx_status (status),
  INDEX idx_created_at (created_at)
);
```

---

## Design Patterns

### 1. **Store-and-Forward Pattern**
**Problem**: Unreliable network connectivity in rural areas
**Solution**: 
- All data writes go to local database first
- Background service queues data for sync when connectivity is available
- Conflict resolution strategy for concurrent updates

**Implementation**:
```typescript
// Pseudo-code structure
class OfflineSyncManager {
  async writeData(entity: Entity) {
    // Write to local DB
    await localDB.save(entity);
    // Queue for sync
    await syncQueue.add({
      entityType: entity.type,
      entityId: entity.id,
      operation: 'CREATE',
      payload: entity
    });
    // Trigger sync if online
    if (this.isOnline()) {
      this.sync();
    }
  }
}
```

### 2. **Circuit Breaker Pattern**
**Problem**: External service failures (Govt APIs, Pharmacy APIs) can cascade
**Solution**: 
- Monitor failure rates
- Open circuit after threshold failures
- Fail fast during open state
- Attempt recovery periodically

**Implementation**:
```typescript
class CircuitBreaker {
  private failures = 0;
  private state: 'CLOSED' | 'OPEN' | 'HALF_OPEN' = 'CLOSED';
  
  async execute(fn: Function) {
    if (this.state === 'OPEN') {
      throw new Error('Circuit breaker is OPEN');
    }
    try {
      const result = await fn();
      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure();
      throw error;
    }
  }
}
```

### 3. **Adapter Pattern**
**Problem**: Multiple IoT device protocols and formats
**Solution**: 
- Create device adapters for each IoT device type
- Normalize data format internally
- Easy to add new devices

**Implementation**:
```typescript
interface DeviceAdapter {
  connect(): Promise<void>;
  readVitals(): Promise<Vitals>;
  disconnect(): Promise<void>;
}

class BPMonitorAdapter implements DeviceAdapter {
  // BP Monitor specific implementation
}

class OximeterAdapter implements DeviceAdapter {
  // Oximeter specific implementation
}
```

### 4. **Strategy Pattern**
**Problem**: Different bandwidth optimization strategies based on network conditions
**Solution**: 
- Define bandwidth optimization strategies
- Switch strategies dynamically based on network quality

**Implementation**:
```typescript
interface BandwidthStrategy {
  optimizeVideo(stream: VideoStream): VideoStream;
}

class LowBandwidthStrategy implements BandwidthStrategy {
  optimizeVideo(stream) {
    // Reduce resolution, frame rate
  }
}

class HighBandwidthStrategy implements BandwidthStrategy {
  optimizeVideo(stream) {
    // Maintain quality
  }
}
```

### 5. **Observer Pattern**
**Problem**: Multiple components need to react to consultation status changes
**Solution**: 
- Event-driven architecture
- Publish consultation events
- Subscribers react accordingly

**Implementation**:
```typescript
class ConsultationEventBus {
  private subscribers: Map<string, Function[]> = new Map();
  
  subscribe(event: string, handler: Function) {
    // Add subscriber
  }
  
  publish(event: string, data: any) {
    // Notify all subscribers
  }
}

// Usage
eventBus.subscribe('CONSULTATION_STARTED', (data) => {
  notificationService.notifyDoctor(data);
  queueService.removeFromQueue(data.id);
});
```

### 6. **Repository Pattern**
**Problem**: Decouple data access logic from business logic
**Solution**: 
- Abstract data access behind repository interfaces
- Easy to swap implementations (local DB, remote DB, cache)

**Implementation**:
```typescript
interface ConsultationRepository {
  findById(id: string): Promise<Consultation>;
  save(consultation: Consultation): Promise<void>;
  findByPatientId(patientId: string): Promise<Consultation[]>;
}

class LocalConsultationRepository implements ConsultationRepository {
  // SQLite implementation
}

class RemoteConsultationRepository implements ConsultationRepository {
  // API implementation
}
```

### 7. **Factory Pattern**
**Problem**: Creating different types of notifications (SMS, WhatsApp, Push)
**Solution**: 
- Notification factory creates appropriate notification handler
- Easy to add new notification channels

**Implementation**:
```typescript
interface NotificationHandler {
  send(message: string, recipient: string): Promise<void>;
}

class NotificationFactory {
  create(type: 'SMS' | 'WHATSAPP' | 'PUSH'): NotificationHandler {
    switch(type) {
      case 'WHATSAPP': return new WhatsAppHandler();
      case 'SMS': return new SMSHandler();
      case 'PUSH': return new PushHandler();
    }
  }
}
```

### 8. **Facade Pattern**
**Problem**: Complex interactions between multiple services for consultation flow
**Solution**: 
- ConsultationFacade provides simple interface
- Hides complexity of orchestrating multiple services

**Implementation**:
```typescript
class ConsultationFacade {
  async startConsultation(patientId: string, symptoms: string) {
    // 1. Create consultation
    const consultation = await consultationService.create(patientId);
    // 2. Translate symptoms
    const translated = await aiService.translate(symptoms);
    // 3. Calculate severity
    const severity = await aiService.analyze(translated);
    // 4. Add to queue
    await queueService.add(consultation.id, severity);
    // 5. Notify doctor
    await notificationService.notify(consultation.id);
    return consultation;
  }
}
```

### 9. **Proxy Pattern**
**Problem**: Need to add caching, logging, access control to service calls
**Solution**: 
- Service proxy intercepts calls
- Adds cross-cutting concerns

**Implementation**:
```typescript
class ConsultationServiceProxy {
  constructor(private service: ConsultationService) {}
  
  async getConsultation(id: string) {
    // Check cache
    const cached = await cache.get(id);
    if (cached) return cached;
    
    // Check access
    if (!this.hasAccess(id)) throw new Error('Unauthorized');
    
    // Log access
    await auditLogger.log('READ', id);
    
    // Call actual service
    const result = await this.service.getConsultation(id);
    
    // Cache result
    await cache.set(id, result);
    
    return result;
  }
}
```

### 10. **Command Pattern**
**Problem**: Need to queue, undo, and log operations (especially for offline sync)
**Solution**: 
- Encapsulate operations as command objects
- Queue commands for execution
- Support undo/redo

**Implementation**:
```typescript
interface Command {
  execute(): Promise<void>;
  undo(): Promise<void>;
}

class CreateConsultationCommand implements Command {
  constructor(private consultation: Consultation) {}
  
  async execute() {
    await consultationService.create(this.consultation);
  }
  
  async undo() {
    await consultationService.delete(this.consultation.id);
  }
}

class CommandQueue {
  private queue: Command[] = [];
  
  async execute(command: Command) {
    this.queue.push(command);
    await command.execute();
  }
}
```

### 11. **Template Method Pattern**
**Problem**: Similar flows for different consultation types with slight variations
**Solution**: 
- Define template method with common steps
- Subclasses override specific steps

**Implementation**:
```typescript
abstract class ConsultationFlow {
  async process() {
    await this.validatePatient();
    await this.collectSymptoms();
    await this.analyzeSymptoms();
    await this.assignDoctor();
    await this.startConsultation();
    await this.completeConsultation();
  }
  
  protected abstract collectSymptoms(): Promise<void>;
  protected abstract assignDoctor(): Promise<void>;
}

class EmergencyConsultationFlow extends ConsultationFlow {
  protected async assignDoctor() {
    // Assign available doctor immediately
  }
}

class ScheduledConsultationFlow extends ConsultationFlow {
  protected async assignDoctor() {
    // Assign based on schedule
  }
}
```

### 12. **Decorator Pattern**
**Problem**: Need to add features like encryption, compression, logging to data without modifying core classes
**Solution**: 
- Wrap data handlers with decorators
- Compose features dynamically

**Implementation**:
```typescript
interface DataHandler {
  handle(data: any): Promise<any>;
}

class EncryptionDecorator implements DataHandler {
  constructor(private handler: DataHandler) {}
  
  async handle(data: any) {
    const encrypted = await this.encrypt(data);
    return this.handler.handle(encrypted);
  }
}

class CompressionDecorator implements DataHandler {
  constructor(private handler: DataHandler) {}
  
  async handle(data: any) {
    const compressed = await this.compress(data);
    return this.handler.handle(compressed);
  }
}
```

---

## System Components

### Technology Stack Recommendations

#### Frontend
- **Kiosk/Sahayak Apps**: React Native (offline-first, cross-platform)
- **Doctor App**: React (WebRTC support, desktop-friendly)
- **State Management**: Redux Toolkit / Zustand
- **Offline Storage**: SQLite (via react-native-sqlite-storage)

#### Backend
- **API Framework**: Node.js (Express) / Python (FastAPI) / Java (Spring Boot)
- **Microservices Communication**: gRPC (internal), REST (external)
- **Message Queue**: Apache Kafka / RabbitMQ
- **Service Discovery**: Consul / Eureka

#### Database
- **Primary DB**: PostgreSQL (structured data) / MongoDB (flexible schemas)
- **Offline DB**: SQLite (mobile apps)
- **Cache**: Redis (session, queue, rate limiting)
- **Search**: Elasticsearch (patient search, symptom search)

#### Infrastructure
- **Containerization**: Docker
- **Orchestration**: Kubernetes
- **API Gateway**: Kong / AWS API Gateway
- **CDN**: CloudFront / Cloudflare (for static assets, video)
- **Monitoring**: Prometheus + Grafana
- **Logging**: ELK Stack (Elasticsearch, Logstash, Kibana)

#### AI/ML Services
- **LLM**: OpenAI GPT-4 / Anthropic Claude / Local LLM (for privacy)
- **Speech-to-Text**: Google Speech-to-Text / AWS Transcribe
- **Translation**: Google Translate API / Custom model
- **Computer Vision**: TensorFlow / PyTorch models
- **ML Framework**: TensorFlow Serving / MLflow

#### External Integrations
- **Video**: WebRTC (Janus/Kurento media server)
- **WhatsApp**: WhatsApp Business API
- **Govt APIs**: Ayushman Bharat APIs (REST)
- **Pharmacy**: Partner pharmacy APIs

---

## Data Flow

### Consultation Flow (End-to-End)

```
1. Patient arrives at kiosk
   ↓
2. Sahayak registers/authenticates patient
   ↓
3. Patient describes symptoms (voice/text in dialect)
   ↓
4. Sahayak uploads vitals from IoT devices
   ↓
5. System translates symptoms to English
   ↓
6. AI analyzes and calculates severity score
   ↓
7. Consultation created and added to queue
   ↓
8. Doctor notified (if available)
   ↓
9. Doctor accepts consultation
   ↓
10. Video room created (low-bandwidth optimized)
    ↓
11. Patient, Sahayak, Doctor join video call
    ↓
12. Consultation proceeds (with AI assistance)
    ↓
13. Doctor generates prescription
    ↓
14. Prescription saved and shared with patient
    ↓
15. Pharmacy integration (if consent given)
    ↓
16. Medicine order placed
    ↓
17. Follow-up reminders scheduled (WhatsApp)
    ↓
18. Consultation marked complete
```

### Offline Sync Flow

```
1. App detects offline mode
   ↓
2. All writes go to local SQLite
   ↓
3. Operations queued in sync_queue table
   ↓
4. App detects connectivity restored
   ↓
5. Background sync service starts
   ↓
6. For each queued item:
   a. Send to server
   b. If conflict detected → resolve
   c. Mark as synced
   ↓
7. Update local cache with server data
   ↓
8. Clear synced items from queue
```

### Consent & Access Flow

```
1. Patient needs to share data
   ↓
2. System requests consent (OTP/Biometric)
   ↓
3. Patient verifies identity
   ↓
4. Consent granted and stored
   ↓
5. Access request comes in
   ↓
6. System checks:
   - Role has permission?
   - Consent exists and valid?
   - Access within scope?
   ↓
7. If all checks pass → grant access + log
   ↓
8. If fails → deny + log + notify
```

---

## Security Considerations

1. **Data Encryption**: 
   - At rest: AES-256
   - In transit: TLS 1.3
   - End-to-end encryption for video calls

2. **Authentication**:
   - OAuth 2.0 / JWT tokens
   - Multi-factor authentication for doctors/admins
   - Biometric authentication for patients

3. **Authorization**:
   - Role-Based Access Control (RBAC)
   - Attribute-Based Access Control (ABAC) for fine-grained control
   - Consent-based data sharing

4. **Audit Logging**:
   - All data access logged
   - Immutable audit trail
   - Regular compliance audits

5. **Privacy**:
   - Data minimization
   - Right to be forgotten
   - Consent management
   - DPDP Act compliance

---

## Scalability & Performance

1. **Horizontal Scaling**: Microservices can scale independently
2. **Caching Strategy**: Redis for frequently accessed data
3. **CDN**: Static assets and video content delivery
4. **Database Sharding**: By region/state for patient data
5. **Load Balancing**: Round-robin / Least connections
6. **Rate Limiting**: Per user, per API endpoint
7. **Connection Pooling**: Database connection management
8. **Async Processing**: Background jobs for heavy operations

---

## Monitoring & Observability

1. **Metrics**: 
   - Response times
   - Error rates
   - Queue lengths
   - Active consultations
   - Sync success rates

2. **Logging**:
   - Structured logging (JSON)
   - Log levels (DEBUG, INFO, WARN, ERROR)
   - Centralized log aggregation

3. **Tracing**:
   - Distributed tracing (Jaeger/Zipkin)
   - Request ID propagation
   - Service dependency mapping

4. **Alerting**:
   - High error rates
   - Service downtime
   - Queue backup
   - Sync failures

---

This architecture document provides a comprehensive foundation for building the GramSeva Health platform. Each component can be further detailed based on specific implementation requirements.
