# GramSeva Health - Detailed Feature Specifications

## Table of Contents
1. [Feature Interaction Matrix](#feature-interaction-matrix)
2. [Detailed Technical Specifications](#detailed-technical-specifications)
3. [API Specifications](#api-specifications)
4. [Data Flow Diagrams](#data-flow-diagrams)
5. [Error Handling](#error-handling)

---

## Feature Interaction Matrix

| Feature | F1 | F2 | F3 | F4 | F5 | F6 | F7 | F8 |
|---------|----|----|----|----|----|----|----|----|
| **F1: Tele-Consultation** | - | Uses | Enables | Uses | Requires | Can use | - | - |
| **F2: AI Triage** | Enables | - | - | Uses | - | - | - | - |
| **F3: Prescription** | Uses | - | - | Uses | Requires | - | - | Enables |
| **F4: Offline Sync** | Required | Required | Required | - | - | Required | Required | - |
| **F5: Consent** | Required | - | Required | - | - | Required | Required | - |
| **F6: Visual Diagnosis** | Can use | - | - | Uses | Requires | - | - | - |
| **F7: Govt Integration** | - | - | - | Uses | Requires | - | - | - |
| **F8: WhatsApp Bot** | - | - | Uses | - | - | - | - | - |

**Legend**:
- **Uses**: Feature depends on another feature
- **Enables**: Feature makes another feature possible
- **Required**: Feature is mandatory for another feature
- **Can use**: Feature can optionally use another feature

---

## Detailed Technical Specifications

### F1: Assisted Tele-Consultation Platform

#### Component Architecture

```
TeleConsultationService
├── ConsultationController
│   ├── POST /api/v1/consultations
│   ├── GET /api/v1/consultations/:id
│   ├── POST /api/v1/consultations/:id/join
│   ├── POST /api/v1/consultations/:id/end
│   └── GET /api/v1/consultations/queue
│
├── MediaManager
│   ├── createVideoRoom(consultationId)
│   ├── joinRoom(userId, roomId)
│   ├── optimizeBandwidth(connectionQuality)
│   ├── handleLowBandwidth()
│   └── recordSession(roomId, consent)
│
├── VitalsManager
│   ├── uploadVitals(consultationId, vitals)
│   ├── getVitals(consultationId)
│   └── streamVitals(consultationId) // Real-time
│
└── QueueManager
    ├── addToQueue(consultationId, priority)
    ├── assignDoctor(consultationId)
    ├── prioritizeQueue()
    └── notifyDoctor(consultationId)
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
  severityScore: number; // From F2
  symptoms: string;
  translatedSymptoms: string; // From F2
  videoRoomId: string;
  recordingUrl: string | null;
  consentGiven: boolean; // From F5
  scheduledAt: Date;
  startedAt: Date | null;
  endedAt: Date | null;
  createdAt: Date;
  updatedAt: Date;
}
```

**VideoRoom**
```typescript
interface VideoRoom {
  id: string;
  consultationId: string;
  participants: Participant[];
  status: 'CREATED' | 'ACTIVE' | 'ENDED';
  bandwidthMode: 'LOW' | 'MEDIUM' | 'HIGH';
  recordingEnabled: boolean;
  createdAt: Date;
}

interface Participant {
  userId: string;
  role: 'PATIENT' | 'SAHAYAK' | 'DOCTOR';
  connectionStatus: 'CONNECTING' | 'CONNECTED' | 'DISCONNECTED';
  videoEnabled: boolean;
  audioEnabled: boolean;
  joinedAt: Date;
}
```

#### Bandwidth Optimization Strategy

**Low Bandwidth (< 512 Kbps)**:
- Video: 240p, 15fps
- Audio: Opus codec, 32kbps
- Fallback to audio-only if < 256 Kbps

**Medium Bandwidth (512 Kbps - 1 Mbps)**:
- Video: 360p, 20fps
- Audio: Opus codec, 64kbps

**High Bandwidth (> 1 Mbps)**:
- Video: 480p, 30fps
- Audio: Opus codec, 128kbps

**Adaptive Algorithm**:
1. Monitor connection quality every 5 seconds
2. Adjust quality based on packet loss and latency
3. Notify participants of quality changes
4. Auto-downgrade if quality drops below threshold

#### IoT Device Integration

**Supported Devices**:
- BP Monitor (Bluetooth/BLE)
- Pulse Oximeter (Bluetooth/BLE)
- Digital Thermometer (Bluetooth/BLE)

**Integration Flow**:
1. Sahayak pairs device via app
2. Device sends data via BLE
3. App validates and formats data
4. Upload to server via WebSocket
5. Server broadcasts to doctor's screen

**Vitals Data Format**:
```typescript
interface VitalsData {
  deviceType: 'BP_MONITOR' | 'OXIMETER' | 'THERMOMETER';
  deviceId: string;
  readings: {
    timestamp: Date;
    value: number;
    unit: string;
  }[];
  quality: 'GOOD' | 'FAIR' | 'POOR';
}
```

---

### F2: AI-Powered Triage & Translation

#### Component Architecture

```
AITriageService
├── TranslationEngine
│   ├── detectLanguage(text/audio)
│   ├── translateToEnglish(text, sourceLanguage)
│   └── normalizeText(text)
│
├── SymptomAnalyzer
│   ├── extractSymptoms(translatedText)
│   ├── classifySeverity(symptoms)
│   └── generateSummary(symptoms, severity)
│
├── TriageEngine
│   ├── calculateSeverityScore(symptoms, vitals)
│   ├── recommendPriority(severityScore)
│   └── suggestTriage(priority, symptoms)
│
└── LLMService
    ├── callLLM(prompt, context)
    ├── formatMedicalPrompt(symptoms)
    └── parseMedicalResponse(response)
```

#### Processing Pipeline

```
Input (Voice/Text in Dialect)
    ↓
[Language Detection]
    ↓
[Speech-to-Text] (if voice)
    ↓
[Translation to English]
    ↓
[Medical Symptom Extraction] (LLM)
    ↓
[Severity Classification] (ML Model)
    ↓
[Medical Summary Generation] (LLM)
    ↓
[Priority Assignment]
    ↓
Output (Structured Medical Data)
```

#### LLM Prompts

**Symptom Extraction Prompt**:
```
You are a medical assistant. Extract and structure the following patient symptoms:

Patient Description: {translatedText}

Provide:
1. Primary symptoms (list)
2. Secondary symptoms (list)
3. Duration of symptoms
4. Severity (1-10 scale)
5. Possible conditions (list)
6. Urgency level (LOW/MEDIUM/HIGH/CRITICAL)
```

**Medical Summary Prompt**:
```
Create a concise medical summary for a doctor:

Symptoms: {extractedSymptoms}
Vitals: {vitalsData}
Duration: {duration}
Patient Age: {age}
Gender: {gender}

Format as:
- Chief Complaint
- History of Present Illness
- Review of Systems
- Assessment
- Plan
```

#### Severity Scoring Model

**Factors**:
- Symptom intensity (1-10)
- Symptom duration
- Vital signs deviation
- Pain level (if applicable)
- Functional impact

**Scoring Algorithm**:
```
severityScore = (
  symptomIntensity * 0.3 +
  durationFactor * 0.2 +
  vitalsDeviation * 0.3 +
  painLevel * 0.1 +
  functionalImpact * 0.1
) * 10
```

**Priority Mapping**:
- 0-3: LOW
- 4-6: MEDIUM
- 7-8: HIGH
- 9-10: CRITICAL

#### Supported Languages

**Phase 1** (MVP):
- Hindi
- English
- Telugu
- Tamil

**Phase 2**:
- Bengali
- Marathi
- Gujarati
- Kannada
- Malayalam
- Punjabi

---

### F3: Prescription & Fulfillment Engine

#### Component Architecture

```
PrescriptionService
├── PrescriptionController
│   ├── POST /api/v1/prescriptions
│   ├── GET /api/v1/prescriptions/:id
│   ├── GET /api/v1/prescriptions/patient/:patientId
│   └── POST /api/v1/prescriptions/:id/refill
│
├── PharmacyIntegration
│   ├── checkAvailability(medicineId, quantity)
│   ├── getPharmacyList(location, medicineIds)
│   ├── placeOrder(orderDetails)
│   └── trackOrder(orderId)
│
├── OrderManager
│   ├── createOrder(prescriptionId, pharmacyId)
│   ├── updateOrderStatus(orderId, status)
│   └── getOrderHistory(patientId)
│
└── NotificationService
    ├── sendPrescription(prescriptionId, channel)
    ├── sendOrderUpdate(orderId, status)
    └── sendReminder(prescriptionId, type)
```

#### Prescription Data Model

```typescript
interface Prescription {
  id: string;
  consultationId: string;
  patientId: string;
  doctorId: string;
  medicines: Medicine[];
  instructions: string;
  followUpDate: Date | null;
  status: 'ACTIVE' | 'COMPLETED' | 'CANCELLED';
  createdAt: Date;
  digitalSignature: string; // For tamper-proofing
}

interface Medicine {
  name: string;
  genericName: string;
  dosage: string; // e.g., "500mg"
  frequency: string; // e.g., "Twice daily"
  duration: string; // e.g., "7 days"
  quantity: number;
  instructions: string; // e.g., "After meals"
  available: boolean;
  pharmacyPrice: number;
}
```

#### Pharmacy Integration

**Pharmacy Partner API Contract**:
```typescript
interface PharmacyAPI {
  // Check medicine availability
  checkAvailability(request: {
    medicineIds: string[];
    location: { lat: number; lng: number; };
    radius: number; // km
  }): Promise<PharmacyAvailability[]>;
  
  // Place order
  placeOrder(order: {
    prescriptionId: string;
    medicines: MedicineOrder[];
    deliveryAddress: Address;
    patientId: string;
  }): Promise<OrderResponse>;
  
  // Track order
  trackOrder(orderId: string): Promise<OrderStatus>;
}

interface PharmacyAvailability {
  pharmacyId: string;
  pharmacyName: string;
  distance: number; // km
  medicines: {
    medicineId: string;
    available: boolean;
    price: number;
    estimatedDelivery: Date;
  }[];
}
```

#### Order Status Flow

```
PENDING
  ↓
CONFIRMED (Pharmacy confirms)
  ↓
PREPARING (Pharmacy preparing order)
  ↓
DISPATCHED (Out for delivery)
  ↓
DELIVERED
  ↓
COMPLETED
```

**Alternative Flow**:
```
PENDING
  ↓
CANCELLED (by patient/pharmacy)
```

---

### F4: Offline Sync & Storage

#### Component Architecture

```
OfflineSyncService
├── LocalStorageManager
│   ├── save(entityType, entity)
│   ├── find(entityType, id)
│   ├── findAll(entityType, filters)
│   └── delete(entityType, id)
│
├── SyncQueueManager
│   ├── enqueue(operation)
│   ├── dequeue()
│   ├── getPendingCount()
│   └── clearSynced()
│
├── SyncEngine
│   ├── sync()
│   ├── syncEntity(entityType, entityId)
│   ├── detectConflict(local, remote)
│   └── resolveConflict(conflict)
│
└── ConflictResolver
    ├── resolveByTimestamp(local, remote)
    ├── resolveByVersion(local, remote)
    └── flagForManualResolution(conflict)
```

#### Sync Queue Structure

```typescript
interface SyncOperation {
  id: string;
  entityType: 'CONSULTATION' | 'VITALS' | 'PRESCRIPTION' | 'PATIENT' | 'CONSENT';
  entityId: string;
  operation: 'CREATE' | 'UPDATE' | 'DELETE';
  payload: any;
  localVersion: number;
  remoteVersion: number | null;
  deviceId: string;
  timestamp: Date;
  status: 'PENDING' | 'SYNCING' | 'SYNCED' | 'FAILED' | 'CONFLICT';
  retryCount: number;
  lastError?: string;
  syncedAt?: Date;
}
```

#### Conflict Resolution Strategies

**Strategy 1: Last Write Wins (Timestamp)**
- Compare `updatedAt` timestamps
- Keep the most recent version
- Use for: Non-critical data (preferences, settings)

**Strategy 2: Version-Based**
- Compare version numbers
- Merge if versions are compatible
- Use for: Structured data (patient info)

**Strategy 3: Manual Resolution**
- Flag conflict for user review
- Store both versions
- Use for: Critical data (prescriptions, vitals)

**Strategy 4: Field-Level Merge**
- Merge non-conflicting fields
- Flag conflicting fields
- Use for: Complex entities (consultations)

#### Sync Triggers

1. **Automatic**:
   - On connectivity restore
   - Periodic (every 5 minutes when online)
   - After local write operation (if online)

2. **Manual**:
   - User-triggered sync button
   - Pull-to-refresh gesture

3. **Priority-Based**:
   - Critical operations first (prescriptions, vitals)
   - Then regular operations
   - Finally, background sync (logs, analytics)

#### Local Database Schema (SQLite)

```sql
-- Entities table (generic)
CREATE TABLE entities (
  id TEXT PRIMARY KEY,
  type TEXT NOT NULL,
  data TEXT NOT NULL, -- JSON
  version INTEGER DEFAULT 1,
  updated_at INTEGER NOT NULL,
  synced_at INTEGER,
  INDEX idx_type (type),
  INDEX idx_synced (synced_at)
);

-- Sync queue
CREATE TABLE sync_queue (
  id TEXT PRIMARY KEY,
  entity_type TEXT NOT NULL,
  entity_id TEXT NOT NULL,
  operation TEXT NOT NULL,
  payload TEXT NOT NULL, -- JSON
  status TEXT DEFAULT 'PENDING',
  retry_count INTEGER DEFAULT 0,
  created_at INTEGER NOT NULL,
  INDEX idx_status (status)
);

-- Conflict log
CREATE TABLE conflicts (
  id TEXT PRIMARY KEY,
  entity_type TEXT NOT NULL,
  entity_id TEXT NOT NULL,
  local_data TEXT NOT NULL, -- JSON
  remote_data TEXT NOT NULL, -- JSON
  resolution_strategy TEXT,
  resolved_at INTEGER,
  created_at INTEGER NOT NULL
);
```

---

### F5: Consent & Privacy (DPDP Act Compliance)

#### Component Architecture

```
ConsentPrivacyService
├── ConsentManager
│   ├── requestConsent(patientId, consentType, grantedTo)
│   ├── verifyConsent(consentId, method, token)
│   ├── revokeConsent(consentId)
│   └── getConsentStatus(patientId, consentType)
│
├── AccessControl
│   ├── checkPermission(userId, resource, action)
│   ├── enforceRBAC(userId, resource)
│   └── auditAccess(userId, resource, action)
│
├── EncryptionService
│   ├── encryptData(data, keyId)
│   ├── decryptData(encryptedData, keyId)
│   └── rotateKeys()
│
└── AuditLogger
    ├── logAccess(userId, resource, action, result)
    ├── logConsent(consentId, action)
    └── generateAuditReport(filters)
```

#### Consent Types

```typescript
enum ConsentType {
  DATA_SHARING = 'DATA_SHARING', // Share data with doctor
  RECORDING = 'RECORDING', // Record consultation
  PHARMACY_SHARING = 'PHARMACY_SHARING', // Share prescription with pharmacy
  GOVT_SCHEME = 'GOVT_SCHEME', // Share with government schemes
  RESEARCH = 'RESEARCH', // Use data for research (anonymized)
  ANALYTICS = 'ANALYTICS' // Use for analytics (anonymized)
}
```

#### Verification Methods

**OTP Verification**:
1. Generate 6-digit OTP
2. Send via SMS
3. Patient enters OTP
4. Verify and grant consent
5. Expires in 10 minutes

**Biometric Verification**:
1. Request biometric (fingerprint/face)
2. Capture biometric data
3. Verify against stored template
4. Grant consent if match
5. Store verification record

**Signature Verification**:
1. Display consent terms
2. Patient signs on screen
3. Store signature image
4. Grant consent
5. Signature is legally binding

#### RBAC Model

```typescript
enum Role {
  PATIENT = 'PATIENT',
  SAHAYAK = 'SAHAYAK',
  DOCTOR = 'DOCTOR',
  ADMIN = 'ADMIN',
  PHARMACY = 'PHARMACY',
  GOVT_SCHEME = 'GOVT_SCHEME'
}

interface Permission {
  resource: string; // e.g., 'patient.data', 'prescription'
  actions: string[]; // e.g., ['read', 'write']
  conditions?: { // Optional conditions
    patientId?: string; // Only for specific patient
    activeOnly?: boolean; // Only active consultations
  };
}

const RolePermissions: Record<Role, Permission[]> = {
  PATIENT: [
    { resource: 'own.data', actions: ['read', 'write'] },
    { resource: 'own.prescription', actions: ['read'] }
  ],
  SAHAYAK: [
    { resource: 'patient.data', actions: ['read'], conditions: { activeOnly: true } },
    { resource: 'vitals', actions: ['create'] }
  ],
  DOCTOR: [
    { resource: 'patient.data', actions: ['read'], conditions: { patientId: 'assigned' } },
    { resource: 'prescription', actions: ['create', 'read'] }
  ],
  // ... other roles
};
```

#### Access Control Flow

```
Access Request
    ↓
[Check User Role]
    ↓
[Check Resource Permissions]
    ↓
[Check Consent] (if required)
    ↓
[Check Conditions] (activeOnly, patientId, etc.)
    ↓
[Log Access] (audit trail)
    ↓
Grant/Deny Access
```

---

### F6: Visual Diagnosis Aid

#### Component Architecture

```
VisualDiagnosisService
├── ImageProcessor
│   ├── uploadImage(file, patientId)
│   ├── validateImage(file)
│   ├── compressImage(image)
│   └── storeImage(imageId, url)
│
├── CVModel
│   ├── analyzeImage(imageUrl)
│   ├── classifyCondition(image)
│   ├── calculateConfidence(predictions)
│   └── generateReport(analysis)
│
└── ReportGenerator
    ├── createReport(analysis, patientInfo)
    ├── suggestFollowUp(condition)
    └── compareImages(before, after)
```

#### Supported Conditions

**Phase 1**:
- Skin rashes
- Wounds (healing assessment)
- Burns
- Infections
- Allergic reactions

**Phase 2**:
- Dermatitis
- Psoriasis
- Eczema
- Fungal infections
- Vitiligo

#### CV Model Output

```typescript
interface ImageAnalysis {
  imageId: string;
  conditions: ConditionPrediction[];
  confidence: number; // Overall confidence (0-1)
  recommendations: string[];
  severity: 'MILD' | 'MODERATE' | 'SEVERE';
  requiresUrgentCare: boolean;
}

interface ConditionPrediction {
  condition: string;
  confidence: number; // 0-1
  description: string;
  treatmentSuggestions: string[];
}
```

#### Image Requirements

- **Format**: JPG, PNG
- **Size**: Max 10MB
- **Resolution**: Min 640x480
- **Compression**: Automatic compression to < 2MB
- **Privacy**: Images encrypted at rest

---

### F7: Government Scheme Integration

#### Component Architecture

```
GovtSchemeService
├── ABHAService
│   ├── validateABHA(abhaId)
│   ├── verifyIdentity(abhaId, otp)
│   └── fetchHealthRecords(abhaId)
│
├── EligibilityService
│   ├── checkEligibility(abhaId, scheme)
│   ├── getCoverageDetails(abhaId)
│   └── calculateCoPayment(consultation, coverage)
│
├── ClaimService
│   ├── submitClaim(consultationId, abhaId)
│   ├── trackClaim(claimId)
│   └── getClaimHistory(abhaId)
│
└── CacheManager
    ├── cacheEligibility(abhaId, data, ttl)
    ├── getCachedEligibility(abhaId)
    └── invalidateCache(abhaId)
```

#### ABHA Integration

**API Endpoints** (Government):
```
POST /abha/validate
  Request: { abhaId: string }
  Response: { valid: boolean, name: string }

POST /abha/verify
  Request: { abhaId: string, otp: string }
  Response: { verified: boolean, token: string }

GET /abha/health-records
  Headers: { Authorization: Bearer <token> }
  Response: { records: HealthRecord[] }
```

#### Eligibility Check Flow

```
Patient provides ABHA ID
    ↓
[Validate ABHA ID]
    ↓
[Check Cache] (if cached and valid)
    ↓
[Call Govt API] (if not cached)
    ↓
[Check Eligibility]
    ↓
[Calculate Coverage]
    ↓
[Cache Result] (24 hours)
    ↓
[Display to Patient/Doctor]
```

#### Supported Schemes

**Phase 1**:
- Ayushman Bharat Pradhan Mantri Jan Arogya Yojana (AB-PMJAY)

**Phase 2**:
- State-specific health schemes
- Employee State Insurance (ESI)
- Central Government Health Scheme (CGHS)

---

### F8: WhatsApp Bot for Follow-ups

#### Component Architecture

```
WhatsAppBotService
├── MessageHandler
│   ├── receiveMessage(phoneNumber, message)
│   ├── sendMessage(phoneNumber, message)
│   ├── sendReminder(phoneNumber, type, data)
│   └── handleQuery(phoneNumber, query)
│
├── ReminderScheduler
│   ├── scheduleReminder(patientId, type, time)
│   ├── cancelReminder(reminderId)
│   └── getUpcomingReminders(patientId)
│
├── NLPEngine
│   ├── understandIntent(message)
│   ├── extractEntities(message)
│   └── generateResponse(intent, context)
│
└── AppointmentManager
    ├── bookAppointment(patientId, preferredTime)
    ├── confirmAppointment(appointmentId)
    └── cancelAppointment(appointmentId)
```

#### Reminder Types

```typescript
enum ReminderType {
  MEDICATION = 'MEDICATION', // Take medicine
  APPOINTMENT = 'APPOINTMENT', // Upcoming appointment
  FOLLOW_UP = 'FOLLOW_UP', // Follow-up consultation
  TEST = 'TEST', // Lab test reminder
  HEALTH_TIP = 'HEALTH_TIP' // Preventive care tips
}
```

#### Message Templates

**Medication Reminder** (Hindi):
```
नमस्ते {patientName},

यह आपकी दवा का समय है:
- दवा: {medicineName}
- मात्रा: {dosage}
- समय: {time}

कृपया दवा लें और "हाँ" लिखकर पुष्टि करें।

धन्यवाद,
GramSeva Health
```

**Appointment Reminder** (English):
```
Hello {patientName},

Your appointment is scheduled for:
- Date: {date}
- Time: {time}
- Doctor: {doctorName}

Reply "CONFIRM" to confirm or "RESCHEDULE" to change.

Thank you,
GramSeva Health
```

#### NLP Intent Classification

**Intents**:
- `MEDICATION_QUERY`: Questions about medicines
- `APPOINTMENT_BOOKING`: Book/reschedule appointment
- `SYMPTOM_QUERY`: Ask about symptoms
- `PRESCRIPTION_QUERY`: Questions about prescription
- `GENERAL_QUERY`: General health questions
- `GREETING`: Greetings
- `THANK_YOU`: Thank you messages

**Entity Extraction**:
- Medicine names
- Dates/times
- Symptoms
- Appointment IDs

---

## API Specifications

### Common API Patterns

#### Authentication
All APIs require JWT token in header:
```
Authorization: Bearer <token>
```

#### Response Format
```typescript
interface APIResponse<T> {
  success: boolean;
  data?: T;
  error?: {
    code: string;
    message: string;
    details?: any;
  };
  timestamp: Date;
}
```

#### Pagination
```typescript
interface PaginatedResponse<T> {
  data: T[];
  pagination: {
    page: number;
    pageSize: number;
    total: number;
    totalPages: number;
  };
}
```

#### Error Codes
- `400`: Bad Request
- `401`: Unauthorized
- `403`: Forbidden (no consent/permission)
- `404`: Not Found
- `409`: Conflict (sync conflict)
- `429`: Rate Limited
- `500`: Internal Server Error
- `503`: Service Unavailable

---

## Data Flow Diagrams

### Consultation Flow (Detailed)

```
[Patient] → [Sahayak App]
    ↓
[Sahayak registers/authenticates patient]
    ↓
[Patient describes symptoms] → [AI Service]
    ↓
[AI translates & analyzes] → [Severity Score]
    ↓
[Consultation created] → [Queue Service]
    ↓
[Doctor notified] → [Doctor App]
    ↓
[Doctor accepts] → [Video Room created]
    ↓
[All parties join] → [Media Server]
    ↓
[Sahayak uploads vitals] → [Real-time sync]
    ↓
[Doctor sees vitals] → [Consultation proceeds]
    ↓
[Doctor generates prescription] → [Prescription Service]
    ↓
[Prescription sent to patient] → [Notification Service]
    ↓
[Patient orders medicines] → [Pharmacy Service]
    ↓
[Order tracked] → [WhatsApp reminders scheduled]
```

---

## Error Handling

### Error Categories

1. **Network Errors**:
   - Connection timeout → Retry with exponential backoff
   - No connectivity → Queue for offline sync
   - Slow connection → Degrade quality

2. **Authentication Errors**:
   - Token expired → Refresh token
   - Invalid token → Redirect to login
   - No permission → Show error, log access attempt

3. **Data Errors**:
   - Validation failure → Show field-specific errors
   - Conflict → Trigger conflict resolution
   - Not found → Show user-friendly message

4. **Service Errors**:
   - External API failure → Use circuit breaker
   - Service unavailable → Show maintenance message
   - Rate limited → Queue request, retry later

### Error Recovery Strategies

- **Automatic Retry**: Network errors, transient failures
- **Manual Retry**: User-triggered for failed operations
- **Graceful Degradation**: Fallback to basic features
- **Offline Mode**: Queue operations, sync later
- **User Notification**: Clear error messages with actions

---

This detailed specification document provides comprehensive technical details for each feature, enabling development teams to implement the features accurately and consistently.
