# GramSeva Health - Features Documentation

> **⚠️ Core Requirements**: Features F1-F5 are the 5 critical requirements that form the foundation. See [KEY_REQUIREMENTS.md](./KEY_REQUIREMENTS.md) for the complete reference.

## Table of Contents
1. [Core Features](#core-features)
2. [Additional Features](#additional-features)
3. [Feature Dependencies](#feature-dependencies)
4. [Feature Prioritization](#feature-prioritization)
5. [User Stories](#user-stories)
6. [Acceptance Criteria](#acceptance-criteria)

---

## Core Features

> **Note**: Features F1-F5 correspond to the 5 core requirements:
> - **F1** = Assisted Tele-Consultation Platform
> - **F2** = AI-Powered Triage & Translation
> - **F3** = Prescription & Fulfillment Engine
> - **F4** = Offline Sync & Storage
> - **F5** = Consent & Privacy (DPDP Act Compliance)

### F1: Assisted Tele-Consultation Platform

#### Description
A video consultation module optimized for low bandwidth, allowing the Sahayak (intermediary) to upload vitals from IoT devices (BP monitor, Oximeter) directly to the doctor's screen.

#### Functional Requirements
- **FR1.1**: Support multi-party video calls (Patient, Sahayak, Doctor)
- **FR1.2**: Optimize video quality based on available bandwidth (adaptive bitrate)
- **FR1.3**: Allow Sahayak to upload vitals from IoT devices during consultation
- **FR1.4**: Display vitals in real-time on doctor's screen
- **FR1.5**: Support screen sharing for medical records/images
- **FR1.6**: Record consultation sessions (with patient consent)
- **FR1.7**: Handle connection drops gracefully with reconnection
- **FR1.8**: Support audio-only mode for very low bandwidth scenarios

#### Non-Functional Requirements
- **NFR1.1**: Video call should work with minimum 256 Kbps bandwidth
- **NFR1.2**: Latency should be < 500ms for real-time communication
- **NFR1.3**: Support up to 3 concurrent participants
- **NFR1.4**: Video quality should auto-adjust based on network conditions
- **NFR1.5**: Consultation recordings should be encrypted and stored securely

#### Technical Components
- WebRTC for peer-to-peer video communication
- Media server (Janus/Kurento) for multi-party calls
- Adaptive bitrate streaming
- IoT device integration layer
- Real-time data synchronization

#### User Stories
- **US1.1**: As a Doctor, I want to see patient vitals in real-time during consultation so I can make informed decisions
- **US1.2**: As a Sahayak, I want to upload vitals from IoT devices so doctors have accurate patient data
- **US1.3**: As a Patient, I want to have video consultation even with poor internet so I can access healthcare
- **US1.4**: As a Doctor, I want to see consultation recordings so I can review complex cases

---

### F2: AI-Powered Triage & Translation

#### Description
A GenAI bot that listens to the patient's symptoms in local dialect, translates them to English medical summaries for the doctor, and suggests a preliminary severity score.

#### Functional Requirements
- **FR2.1**: Accept symptom input in multiple formats (voice, text, structured form)
- **FR2.2**: Detect and identify local dialect/language automatically
- **FR2.3**: Translate patient symptoms from local dialect to English
- **FR2.4**: Extract medical symptoms and conditions from translated text
- **FR2.5**: Generate structured medical summary for doctor
- **FR2.6**: Calculate severity score (1-10 scale)
- **FR2.7**: Suggest triage priority (LOW, MEDIUM, HIGH, CRITICAL)
- **FR2.8**: Provide confidence score for translation and analysis
- **FR2.9**: Support multiple Indian languages/dialects (Hindi, Telugu, Tamil, Bengali, etc.)

#### Non-Functional Requirements
- **NFR2.1**: Translation should complete within 5 seconds
- **NFR2.2**: Accuracy should be > 85% for common symptoms
- **NFR2.3**: Support at least 10 major Indian languages
- **NFR2.4**: Severity score should be consistent across similar symptoms
- **NFR2.5**: System should handle ambiguous symptoms gracefully

#### Technical Components
- Speech-to-Text service (Google/AWS)
- Language detection model
- Translation service (custom model or API)
- LLM service (GPT-4/Claude) for medical summarization
- Severity scoring ML model
- Triage recommendation engine

#### User Stories
- **US2.1**: As a Patient, I want to describe symptoms in my local language so I can communicate effectively
- **US2.2**: As a Doctor, I want to see translated and summarized symptoms so I can understand patient conditions quickly
- **US2.3**: As a System, I want to prioritize consultations based on severity so critical cases are handled first
- **US2.4**: As a Sahayak, I want the system to translate patient speech so I can help bridge communication gaps

---

### F3: Prescription & Fulfillment Engine

#### Description
Digital prescription generation that integrates with local pharmacy supply chains for medicine delivery.

#### Functional Requirements
- **FR3.1**: Generate digital prescriptions with medicine details, dosage, frequency
- **FR3.2**: Support multiple medicines in a single prescription
- **FR3.3**: Include doctor's instructions and follow-up date
- **FR3.4**: Check medicine availability with pharmacy partners
- **FR3.5**: Allow patient to place order for medicines
- **FR3.6**: Track order status (placed, confirmed, dispatched, delivered)
- **FR3.7**: Support multiple pharmacy partners
- **FR3.8**: Generate prescription PDF for patient records
- **FR3.9**: Send prescription to patient via SMS/WhatsApp
- **FR3.10**: Support prescription refills for chronic conditions

#### Non-Functional Requirements
- **NFR3.1**: Prescription generation should be < 2 seconds
- **NFR3.2**: Medicine availability check should be < 3 seconds
- **NFR3.3**: Support integration with at least 5 pharmacy partners
- **NFR3.4**: Prescription should be tamper-proof (digital signature)
- **NFR3.5**: Order tracking should update in real-time

#### Technical Components
- Prescription generation service
- Pharmacy API integration layer
- Order management system
- PDF generation service
- Notification service (SMS/WhatsApp)
- Digital signature service

#### User Stories
- **US3.1**: As a Doctor, I want to generate digital prescriptions so patients can easily access medicines
- **US3.2**: As a Patient, I want to order medicines online so I don't have to visit pharmacy
- **US3.3**: As a Patient, I want to track my medicine order so I know when to expect delivery
- **US3.4**: As a Pharmacy, I want to receive prescription orders so I can fulfill them

---

### F4: Offline Sync & Storage

#### Description
"Store and Forward" architecture to cache patient data locally when the internet is down and sync when connectivity is restored.

#### Functional Requirements
- **FR4.1**: Store all patient data locally on device (SQLite)
- **FR4.2**: Queue all write operations when offline
- **FR4.3**: Automatically sync when connectivity is restored
- **FR4.4**: Detect and resolve data conflicts
- **FR4.5**: Show sync status to users
- **FR4.6**: Allow manual sync trigger
- **FR4.7**: Support incremental sync (only changed data)
- **FR4.8**: Handle sync failures with retry mechanism
- **FR4.9**: Maintain data integrity during sync
- **FR4.10**: Support partial sync (priority-based)

#### Non-Functional Requirements
- **NFR4.1**: Local database should support at least 1000 patient records
- **NFR4.2**: Sync should complete within 5 minutes for typical data volume
- **NFR4.3**: Conflict resolution should be automatic for 90% of cases
- **NFR4.4**: Data should be encrypted in local storage
- **NFR4.5**: Sync should work in background without blocking UI

#### Technical Components
- Local database (SQLite)
- Sync queue manager
- Conflict resolution engine
- Background sync service
- Data encryption service
- Sync status tracker

#### User Stories
- **US4.1**: As a Sahayak, I want to capture patient data offline so I can work even without internet
- **US4.2**: As a System, I want to sync data automatically so all information is up-to-date
- **US4.3**: As a User, I want to see sync status so I know when data is synchronized
- **US4.4**: As a System, I want to resolve conflicts automatically so data integrity is maintained

---

### F5: Consent & Privacy (DPDP Act Compliance)

#### Description
Strict role-based access control ensuring Sahayaks can only see data for active patients, with explicit consent management (OTP/Biometric) for sharing records.

#### Functional Requirements
- **FR5.1**: Implement role-based access control (RBAC)
- **FR5.2**: Request explicit consent for data sharing
- **FR5.3**: Support multiple consent types (data sharing, recording, pharmacy sharing, govt scheme)
- **FR5.4**: Verify consent using OTP or Biometric
- **FR5.5**: Restrict Sahayak access to active patients only
- **FR5.6**: Allow patients to revoke consent
- **FR5.7**: Log all data access for audit
- **FR5.8**: Encrypt sensitive patient data
- **FR5.9**: Support consent expiration and renewal
- **FR5.10**: Provide consent history to patients

#### Non-Functional Requirements
- **NFR5.1**: Consent verification should complete within 30 seconds
- **NFR5.2**: All data access should be logged within 1 second
- **NFR5.3**: Encryption should use AES-256 standard
- **NFR5.4**: Consent should be stored immutably
- **NFR5.5**: Access control checks should add < 100ms latency

#### Technical Components
- RBAC service
- Consent management service
- OTP service
- Biometric authentication service
- Audit logging service
- Encryption service
- Access control middleware

#### User Stories
- **US5.1**: As a Patient, I want to control who can access my data so my privacy is protected
- **US5.2**: As a Sahayak, I want to access only active patient data so I can assist effectively
- **US5.3**: As a System, I want to log all data access so compliance is maintained
- **US5.4**: As a Patient, I want to revoke consent so I have control over my data

---

## Additional Features

### F6: Visual Diagnosis Aid

#### Description
Computer Vision tools for skin conditions/wounds analysis to assist general practitioners in preliminary assessment.

#### Functional Requirements
- **FR6.1**: Accept image uploads (skin conditions, wounds, rashes)
- **FR6.2**: Analyze images using computer vision models
- **FR6.3**: Identify potential conditions/diseases
- **FR6.4**: Provide confidence scores for identifications
- **FR6.5**: Generate preliminary assessment report
- **FR6.6**: Suggest follow-up actions or tests
- **FR6.7**: Support multiple image formats (JPG, PNG)
- **FR6.8**: Allow image annotation by doctor
- **FR6.9**: Store images securely with patient records
- **FR6.10**: Support image comparison (before/after treatment)

#### Non-Functional Requirements
- **NFR6.1**: Image analysis should complete within 10 seconds
- **NFR6.2**: Accuracy should be > 80% for common skin conditions
- **NFR6.3**: Support images up to 10MB
- **NFR6.4**: Images should be compressed for storage
- **NFR6.5**: Analysis should work offline (on-device model)

#### Technical Components
- Image upload service
- Computer vision model (TensorFlow/PyTorch)
- Image processing service
- Condition classification engine
- Report generation service
- Image storage service

#### User Stories
- **US6.1**: As a Doctor, I want to analyze skin condition images so I can make preliminary assessments
- **US6.2**: As a Patient, I want to upload images of my condition so doctor can see it clearly
- **US6.3**: As a Doctor, I want AI suggestions for conditions so I can consider all possibilities
- **US6.4**: As a System, I want to store images securely so patient privacy is maintained

---

### F7: Government Scheme Integration

#### Description
API integration with Ayushman Bharat (ABHA IDs) to verify insurance eligibility and fetch past health records.

#### Functional Requirements
- **FR7.1**: Validate ABHA ID with government systems
- **FR7.2**: Check insurance eligibility under Ayushman Bharat
- **FR7.3**: Fetch patient's past health records
- **FR7.4**: Display insurance coverage details
- **FR7.5**: Calculate co-payment amount (if any)
- **FR7.6**: Submit claims to insurance (if applicable)
- **FR7.7**: Support multiple government schemes (not just Ayushman Bharat)
- **FR7.8**: Cache eligibility data for offline access
- **FR7.9**: Handle API failures gracefully
- **FR7.10**: Maintain audit trail of all API calls

#### Non-Functional Requirements
- **NFR7.1**: ABHA validation should complete within 5 seconds
- **NFR7.2**: System should handle API timeouts gracefully
- **NFR7.3**: Support retry mechanism for failed API calls
- **NFR7.4**: Cache eligibility data for 24 hours
- **NFR7.5**: All API calls should be logged for audit

#### Technical Components
- Government API integration service
- ABHA validation service
- Insurance eligibility checker
- Health record fetcher
- Claim submission service
- API gateway with circuit breaker
- Caching service

#### User Stories
- **US7.1**: As a Patient, I want to use my ABHA ID so I can access government benefits
- **US7.2**: As a Doctor, I want to see patient's past health records so I can provide better care
- **US7.3**: As a System, I want to verify insurance eligibility so patients know their coverage
- **US7.4**: As a Patient, I want to see my insurance coverage so I can plan my expenses

---

### F8: WhatsApp Bot for Follow-ups

#### Description
Automated vernacular reminders for medication dosage and follow-up appointment scheduling.

#### Functional Requirements
- **FR8.1**: Send medication reminders in patient's preferred language
- **FR8.2**: Support multiple reminder types (medication, appointment, follow-up)
- **FR8.3**: Allow patients to confirm/acknowledge reminders
- **FR8.4**: Support two-way communication (patient can ask questions)
- **FR8.5**: Schedule follow-up appointments via WhatsApp
- **FR8.6**: Send health tips and preventive care information
- **FR8.7**: Handle patient queries using NLP
- **FR8.8**: Escalate complex queries to human support
- **FR8.9**: Support multiple Indian languages
- **FR8.10**: Track medication adherence

#### Non-Functional Requirements
- **NFR8.1**: Reminders should be sent at scheduled times (±5 minutes)
- **NFR8.2**: Bot should respond within 2 seconds for simple queries
- **NFR8.3**: Support at least 1000 concurrent conversations
- **NFR8.4**: Messages should be delivered reliably (99% success rate)
- **NFR8.5**: Bot should handle typos and informal language

#### Technical Components
- WhatsApp Business API integration
- NLP service for query understanding
- Reminder scheduler
- Appointment booking service
- Multi-language support service
- Conversation state manager
- Escalation service

#### User Stories
- **US8.1**: As a Patient, I want to receive medication reminders so I don't miss doses
- **US8.2**: As a Patient, I want to schedule appointments via WhatsApp so it's convenient
- **US8.3**: As a Patient, I want to ask health questions via WhatsApp so I can get quick answers
- **US8.4**: As a System, I want to track medication adherence so I can improve patient outcomes

---

## Feature Dependencies

### Dependency Graph

```
F1 (Tele-Consultation)
  ├── Depends on: F4 (Offline Sync), F5 (Consent)
  └── Enables: F3 (Prescription)

F2 (AI Triage & Translation)
  ├── Depends on: F4 (Offline Sync)
  └── Enables: F1 (Tele-Consultation), Queue prioritization

F3 (Prescription & Fulfillment)
  ├── Depends on: F1 (Tele-Consultation), F5 (Consent)
  └── Enables: F8 (WhatsApp Bot)

F4 (Offline Sync)
  └── Enables: F1, F2, F3, F6, F7 (all features work offline)

F5 (Consent & Privacy)
  └── Required by: F1, F3, F6, F7 (all data access features)

F6 (Visual Diagnosis)
  ├── Depends on: F1 (Tele-Consultation), F5 (Consent)
  └── Can work standalone

F7 (Govt Scheme Integration)
  ├── Depends on: F5 (Consent)
  └── Can work standalone

F8 (WhatsApp Bot)
  ├── Depends on: F3 (Prescription)
  └── Can work standalone
```

### Critical Path

**Phase 1 (Foundation)**:
1. F4 (Offline Sync) - Foundation for all features
2. F5 (Consent & Privacy) - Required for data access

**Phase 2 (Core Consultation)**:
3. F2 (AI Triage & Translation) - Enables smart queuing
4. F1 (Tele-Consultation) - Core consultation feature

**Phase 3 (Completion & Enhancement)**:
5. F3 (Prescription & Fulfillment) - Completes consultation flow
6. F6 (Visual Diagnosis) - Enhances diagnosis
7. F7 (Govt Scheme Integration) - Adds value
8. F8 (WhatsApp Bot) - Improves follow-up

---

## Feature Prioritization

### Priority Levels

#### P0 - Critical (Must Have)
- **F4**: Offline Sync & Storage
- **F5**: Consent & Privacy
- **F1**: Assisted Tele-Consultation Platform
- **F2**: AI-Powered Triage & Translation

**Rationale**: These form the core functionality. Without these, the platform cannot function in rural settings.

#### P1 - High (Should Have)
- **F3**: Prescription & Fulfillment Engine

**Rationale**: Completes the consultation flow and provides end-to-end value.

#### P2 - Medium (Nice to Have)
- **F6**: Visual Diagnosis Aid
- **F7**: Government Scheme Integration

**Rationale**: Enhances the platform but not essential for MVP.

#### P3 - Low (Future Enhancement)
- **F8**: WhatsApp Bot for Follow-ups

**Rationale**: Improves user experience but can be added post-MVP.

---

## User Stories

### Epic 1: Patient Consultation Journey

**As a Patient**, I want to:
- Register at a kiosk with my basic information
- Describe my symptoms in my local language
- Have a video consultation with a doctor
- Receive a digital prescription
- Order medicines online
- Track my medicine delivery
- Receive follow-up reminders

**Acceptance Criteria**:
- Patient can complete full journey from registration to medicine delivery
- All steps work offline (except video call)
- Patient can access consultation history
- Patient receives notifications at each step

### Epic 2: Sahayak Workflow

**As a Sahayak**, I want to:
- Register new patients
- Capture patient vitals using IoT devices
- Upload vitals to doctor's screen
- Assist patients during consultation
- View only active patient data
- Work offline when internet is unavailable

**Acceptance Criteria**:
- Sahayak can complete all tasks offline
- Vitals are uploaded in real-time during consultation
- Sahayak can only access authorized patient data
- Data syncs automatically when online

### Epic 3: Doctor Consultation Experience

**As a Doctor**, I want to:
- See prioritized patient queue
- View translated and summarized symptoms
- See real-time patient vitals
- Conduct video consultations
- Generate digital prescriptions
- Review consultation history
- Analyze patient images for diagnosis

**Acceptance Criteria**:
- Doctor can see all patient information in one place
- Consultations are prioritized by severity
- Prescriptions are generated quickly
- Video quality adapts to bandwidth

---

## Acceptance Criteria

### General Acceptance Criteria

1. **Performance**:
   - All API calls should respond within 3 seconds (95th percentile)
   - Video calls should work with minimum 256 Kbps
   - Offline operations should queue successfully
   - Sync should complete within 5 minutes for typical data

2. **Security**:
   - All data must be encrypted at rest and in transit
   - Access control must be enforced at all endpoints
   - All data access must be logged
   - Consent must be verified before data sharing

3. **Reliability**:
   - System should handle 99.5% uptime
   - Graceful degradation when services are down
   - Automatic retry for failed operations
   - Data should not be lost during sync

4. **Usability**:
   - Interface should support multiple languages
   - Should work on low-end devices
   - Should work with poor internet connectivity
   - Error messages should be clear and actionable

5. **Compliance**:
   - Must comply with DPDP Act
   - Must maintain audit trails
   - Must support consent management
   - Must support data deletion requests

### Feature-Specific Acceptance Criteria

#### F1: Tele-Consultation
- ✅ Video call connects within 10 seconds
- ✅ Vitals appear on doctor's screen within 2 seconds of upload
- ✅ Call quality adapts to bandwidth automatically
- ✅ Reconnection works within 5 seconds after drop

#### F2: AI Triage & Translation
- ✅ Translation completes within 5 seconds
- ✅ Severity score is calculated accurately (>85% accuracy)
- ✅ Supports at least 10 Indian languages
- ✅ Medical summary is clear and actionable

#### F3: Prescription & Fulfillment
- ✅ Prescription generates within 2 seconds
- ✅ Medicine availability check completes within 3 seconds
- ✅ Order tracking updates in real-time
- ✅ Prescription PDF is tamper-proof

#### F4: Offline Sync
- ✅ All data saves locally when offline
- ✅ Sync starts automatically when online
- ✅ Conflicts resolve automatically (90% of cases)
- ✅ Sync status is visible to users

#### F5: Consent & Privacy
- ✅ Consent verification completes within 30 seconds
- ✅ Access is denied without valid consent
- ✅ All access is logged within 1 second
- ✅ Patients can revoke consent anytime

#### F6: Visual Diagnosis
- ✅ Image analysis completes within 10 seconds
- ✅ Condition identification is >80% accurate
- ✅ Images are stored securely
- ✅ Reports are generated automatically

#### F7: Govt Scheme Integration
- ✅ ABHA validation completes within 5 seconds
- ✅ Eligibility check works offline (cached)
- ✅ API failures are handled gracefully
- ✅ All API calls are logged

#### F8: WhatsApp Bot
- ✅ Reminders sent at scheduled times (±5 minutes)
- ✅ Bot responds within 2 seconds
- ✅ Supports multiple languages
- ✅ Handles typos and informal language

---

## Feature Metrics & KPIs

### Success Metrics

1. **Adoption Metrics**:
   - Number of active patients per month
   - Number of consultations per day
   - Number of Sahayaks using the platform
   - Number of doctors on the platform

2. **Performance Metrics**:
   - Average consultation time
   - Video call success rate
   - Sync success rate
   - API response times

3. **Quality Metrics**:
   - Translation accuracy
   - Severity score accuracy
   - Visual diagnosis accuracy
   - Prescription accuracy

4. **User Satisfaction Metrics**:
   - Patient satisfaction score
   - Doctor satisfaction score
   - Sahayak satisfaction score
   - Net Promoter Score (NPS)

5. **Business Metrics**:
   - Medicine order conversion rate
   - Average order value
   - Patient retention rate
   - Cost per consultation

---

## Feature Roadmap

### MVP (Minimum Viable Product) - 3 Months
- F4: Offline Sync & Storage
- F5: Consent & Privacy
- F1: Assisted Tele-Consultation (basic)
- F2: AI Triage & Translation (basic)
- F3: Prescription & Fulfillment (basic)

### Phase 2 - 6 Months
- F1: Enhanced Tele-Consultation (recording, screen sharing)
- F2: Enhanced AI (more languages, better accuracy)
- F6: Visual Diagnosis Aid
- F7: Government Scheme Integration

### Phase 3 - 9 Months
- F8: WhatsApp Bot for Follow-ups
- Advanced analytics and reporting
- Mobile apps (iOS/Android)
- Admin dashboard

### Phase 4 - 12 Months
- AI-powered health recommendations
- Chronic disease management
- Telemedicine for specialized care
- Integration with more government schemes

---

This features document provides a comprehensive breakdown of all features, their requirements, dependencies, and implementation roadmap. Each feature can be further detailed into technical specifications during the design phase.
