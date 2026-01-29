# Class Diagram - Smart Hospital ICU Management System

> **⚠️ Core Requirements**: Classes are designed around the core requirements defined in [KEY_REQUIREMENTS.md](./KEY_REQUIREMENTS.md).

## Table of Contents
1. [Overview](#overview)
2. [Domain Layer Classes](#domain-layer-classes)
3. [Service Layer Classes](#service-layer-classes)
4. [Infrastructure Layer Classes](#infrastructure-layer-classes)
5. [Complete Class Diagram](#complete-class-diagram)
6. [Class Relationships](#class-relationships)

---

## Overview

The Smart Hospital system follows a **Event-Driven Microservices Architecture**:

- **Domain Layer**: Core business entities (Hospital, Bed, Patient).
- **Service Layer**: Business logic (Allocation, Notifications).
- **Infrastructure Layer**: IoT integration and Data Persistence.

---

## Domain Layer Classes

### Operational Domain

```mermaid
classDiagram
    class Hospital {
        +UUID id
        +String name
        +String licenseNumber
        +Address address
        +List~Department~ departments
        +getGlobalBedCapacity() Integer
        +getEmergencyContact() ContactInfo
    }

    class Department {
        +UUID id
        +UUID hospitalId
        +String name
        +DepartmentType type
        +Integer floorLevel
        +String wingIdentifier
        +List~Ward~ wards
        +getUtilization() Percentage
    }

    class Ward {
        +UUID id
        +UUID departmentId
        +String name
        +GenderPolicy genderPolicy
        +WardType type
        +Staff headNurse
        +List~Bed~ beds
        +getAvailableBeds() List~Bed~
    }

    class Bed {
        +UUID id
        +UUID wardId
        +String bedNumber
        +BedStatus status
        +BedType type
        +List~Capability~ capabilities
        +UUID currentPatientId
        +DateTime lastCleanedAt
        +occupy(patientId) void
        +release() void
        +markDirty() void
        +markMaintenance(reason) void
    }

    class BedCapability {
        +UUID id
        +CapabilityType type
        +String description
        +Boolean isOperational
    }

    Hospital "1" *-- "many" Department
    Department "1" *-- "many" Ward
    Ward "1" *-- "many" Bed
    Bed "1" *-- "many" BedCapability
```

### Patient & Admission Domain

```mermaid
classDiagram
    class Patient {
        +UUID id
        +String mrn
        +String firstName
        +String lastName
        +Date dob
        +BloodType bloodGroup
        +List~Allergy~ allergies
        +ClinicalAcuity currentAcuity
        +getAdmissionHistory() List~Admission~
    }

    class Admission {
        +UUID id
        +UUID patientId
        +UUID bedId
        +AdmissionType type
        +AdmissionStatus status
        +DateTime admittedAt
        +DateTime dischargedAt
        +UUID admittingDoctorId
        +String diagnosis
        +transfer(newBedId) void
        +discharge() void
    }

    class MedicalRecord {
        +UUID id
        +UUID patientId
        +String diagnosisCode
        +String notes
        +UUID recordedBy
        +DateTime timestamp
    }

    class Reservation {
        +UUID id
        +UUID bedId
        +UUID patientId
        +UUID reservedBy
        +ReservationStatus status
        +DateTime createdAt
        +DateTime expiresAt
        +String priorityReason
        +cancel() void
        +fulfill() void
    }

    Patient "1" -- "many" Admission
    Patient "1" -- "many" MedicalRecord
    Admission "1" -- "1" Bed
    Reservation "1" -- "1" Bed
    Reservation "1" -- "1" Patient
```

### Staff & Access Control

```mermaid
classDiagram
    class Staff {
        +UUID id
        +String employeeId
        +String fullName
        +Role role
        +Department assignedDepartment
        +Boolean isActive
        +canOverride() Boolean
        +login() AuthToken
    }

    class AuditLog {
        +UUID id
        +UUID entityId
        +EntityType entityType
        +ActionType action
        +UUID performedBy
        +DateTime timestamp
        +JSON changes
        +String reason
    }

    Staff "1" -- "many" AuditLog : generates >
```

---

## Service Layer Classes

### Core Services

```mermaid
classDiagram
    class BedManagementService {
        -BedRepository bedRepo
        -ReservationRepository reservationRepo
        -EventPublisher eventBus
        +getBeds(filter) List~Bed~
        +updateBedStatus(bedId, status) Bed
        +reserveBed(request) Reservation
        +overrideAllocation(request) Allocation
        +processIoTUpdate(telemetry) void
    }

    class AllocationEngine {
        -StrategyLoader strategy
        -RuleEngine rules
        +findBestBed(patientResults, requirements) Bed
        +validatePolicy(allocation) ValidationResult
        +suggestTransfers(wardId) List~Suggestion~
    }

    class PatientService {
        -PatientRepository patientRepo
        -ExternalAdtClient adtClient
        +syncPatientFromHis(mrn) Patient
        +updateAcuityScore(patientId, vitals) void
        +getPatientDetails(id) Patient
    }

    class NotificationService {
        -PushProvider pushClient
        -SMSProvider smsClient
        -PagerDutyClient pagerClient
        +sendBedReadyAlert(staffId, bedId)
        +sendCodeBlue(location)
        +notifyCleaningRequired(bedId)
    }

    BedManagementService --> AllocationEngine
    BedManagementService --> NotificationService
```

---

## Infrastructure Layer Classes

```mermaid
classDiagram
    class IoTGateway {
        -MqttClient mqtt
        -DeviceRegistry registry
        +subscribeToVitals() Stream
        +parseTelemetry(payload) VitalsData
        +checkDeviceHealth(deviceId) Status
    }

    class HL7Listener {
        -TcpServer server
        -HL7Parser parser
        +listen(port) void
        +onMessageADT_A01(msg) void
        +onMessageADT_A02(msg) void
        +onMessageADT_A03(msg) void
    }

    class GeoLocationService {
        -BeaconRegistry beacons
        +trackAsset(assetTag) Location
        +getNearestPorter(location) Staff
    }

    BedManagementService --> IoTGateway
    PatientService --> HL7Listener
```

---

## Complete Class Diagram

```mermaid
classDiagram
    %% Domain
    class Hospital { +UUID id }
    class Ward { +UUID id }
    class Bed { +UUID id, +Status status }
    class Patient { +UUID id, +MRN mrn }
    class Admission { +UUID id }
    class Reservation { +UUID id }
    class Staff { +UUID id, +Role role }

    %% Services
    class BedMgmtSvc { +reserveBed() }
    class AllocationSvc { +findBestBed() }
    class NotificationSvc { +sendAlert() }

    %% Relationships
    Hospital "1" *-- "*" Ward
    Ward "1" *-- "*" Bed
    Bed "1" -- "0..1" Patient : occupies
    Admission "1" -- "1" Bed
    Reservation "1" -- "1" Bed
    Staff "1" -- "*" Reservation
    BedMgmtSvc --> Bed
    BedMgmtSvc --> Reservation
    AllocationSvc --> Bed
```

---

## Class Relationships

### Key Relationships

| Relationship | Description | Example |
|--------------|-------------|---------|
| **Composition** | Strong ownership | Hospital → Department → Ward |
| **Association** | Operational link | Admission → Bed |
| **Dependency** | Service Usage | BedManagementService → NotificationService |
| **Aggregation** | Grouping | Ward → Staff (Staff can work in multiple wards) |

1.  **Hospital → Department → Ward → Bed**: Hierarchical structure physically modeling the building.
2.  **Admission → Bed**: The core transactional link. An admission locks a bed for a duration.
3.  **Reservation → Bed**: A temporary lock that expires if not converted to an Admission.
4.  **Staff → AuditLog**: Security requirement to trace every override to a specific person.
