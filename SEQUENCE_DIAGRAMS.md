# Sequence Diagrams - Smart Hospital ICU Management System

> **⚠️ Core Requirements**: Each sequence diagram maps to the core requirements defined in [KEY_REQUIREMENTS.md](./KEY_REQUIREMENTS.md).

## Table of Contents
1. [Bed Allocation & Reservation Flow](#bed-allocation--reservation-flow)
2. [Emergency Override (Code Blue)](#emergency-override-code-blue)
3. [Patient Discharge & Bed Cleaning](#patient-discharge--bed-cleaning)
4. [IoT Vitals Alert & Auto-Transfer](#iot-vitals-alert--auto-transfer)

---

## Bed Allocation & Reservation Flow

**Requirement**: Priority-Based Auto-Allocation Implementation
**Use Case**: UC-1 (Bed Allocation)

```mermaid
sequenceDiagram
    autonumber
    participant Nurse as Ward Nurse (UI)
    participant API as API Gateway
    participant Auth as Auth Service
    participant BedSvc as Bed Mgmt Service
    participant AllocSvc as Allocation Engine
    participant DB as Database (PostgreSQL)
    participant Notif as Notification Service

    Nurse->>API: GET /beds?type=ICU&capability=VENTILATOR
    API->>Auth: validateToken(jwt)
    Auth-->>API: Token Valid (Role: NURSE)
    API->>BedSvc: searchBeds(criteria)
    BedSvc->>DB: SELECT * FROM beds WHERE type='ICU' AND status='AVAILABLE'
    DB-->>BedSvc: List[Bed]
    BedSvc-->>API: 200 OK (List of Beds)
    API-->>Nurse: Display Bed List

    Nurse->>API: POST /reservations {patientId, bedId}
    API->>BedSvc: createReservation(dto)
    
    BedSvc->>DB: BEGIN TRANSACTION
    BedSvc->>DB: SELECT status, version FROM beds WHERE id=bedId FOR UPDATE
    
    alt Bed is Available
        DB-->>BedSvc: status='AVAILABLE'
        BedSvc->>AllocSvc: validateRules(patient, bed)
        AllocSvc-->>BedSvc: Validation Passed
        
        BedSvc->>DB: INSERT INTO reservations (...)
        BedSvc->>DB: UPDATE beds SET status='RESERVED', version=v+1
        BedSvc->>DB: COMMIT
        DB-->>BedSvc: Success
        
        BedSvc->>Notif: sendConfirmation(nurseId, bedId)
        Notif-->>Nurse: Push "Reservation Confirmed"
        BedSvc-->>API: 201 Created
        API-->>Nurse: Show Success Modal
    else Bed was taken (Race Condition)
        DB-->>BedSvc: status='RESERVED'
        BedSvc->>DB: ROLLBACK
        BedSvc-->>API: 409 Conflict "Bed already taken"
        API-->>Nurse: Error "Please select another bed"
    end
```

---

## Emergency Override (Code Blue)

**Requirement**: Emergency Priority Handling
**Use Case**: UC-2 (Emergency Override)

```mermaid
sequenceDiagram
    autonumber
    participant Doc as Doctor (Mobile)
    participant API as API Gateway
    participant BedSvc as Bed Mgmt Service
    participant Audit as Audit Service
    participant DB as Database
    participant Push as Push Notification System

    Doc->>API: POST /allocations/override {bedId, reason="Code Blue"}
    API->>BedSvc: processOverride(dto)
    
    BedSvc->>BedSvc: checkPermission(User.Role == DOCTOR)
    
    BedSvc->>DB: SELECT * FROM reservations WHERE bed_id=bedId
    DB-->>BedSvc: existingReservation (Patient B)
    
    par Cancel Old Reservation
        BedSvc->>DB: UPDATE reservations SET status='CANCELLED' WHERE id=resId
        BedSvc->>Push: notify(StaffB, "Reservation Cancelled: Emergency Override")
    and Create New Allocation
        BedSvc->>DB: UPDATE beds SET status='OCCUPIED', patient_id=PatientA
    and Log Audit
        BedSvc->>Audit: logEvent("OVERRIDE", {actor: Doc, reason: "Code Blue"})
        Audit->>DB: INSERT INTO audit_logs
    end

    BedSvc-->>API: 200 OK
    API-->>Doc: "Bed Override Successful"
```

---

## Patient Discharge & Bed Cleaning

**Requirement**: Intelligent Patient Flow
**Use Case**: UC-4 (Mark Bed Clean)

```mermaid
sequenceDiagram
    autonumber
    participant HIS as Legacy HIS (ADT)
    participant Listener as HL7 Listener
    participant BedSvc as Bed Mgmt Service
    participant DB as Database
    participant Porter as Housekeeping App (Mobile)

    HIS->>Listener: HL7 ADT_A03 (Discharge)
    Listener->>Listener: Parse Message (PatientID, BedID)
    Listener->>BedSvc: handleDischarge(bedId)
    
    BedSvc->>DB: UPDATE beds SET status='CLEANING', current_patient=NULL
    BedSvc->>Porter: dispatchJob({type: "Terminal Clean", bedId})
    
    Note over Porter: Porter receives alert,<br/>cleans the room
    
    Porter->>API: POST /beds/{id}/clean-complete
    API->>BedSvc: markClean(bedId)
    BedSvc->>DB: UPDATE beds SET status='AVAILABLE', last_cleaned=NOW()
    
    BedSvc->>Push: notifyAdmissionDesk("Bed Ready")
```

---

## IoT Vitals Alert & Auto-Transfer

**Requirement**: Smart ICU Integration
**Use Case**: UC-5 (Real-Time Acuity Update)

```mermaid
sequenceDiagram
    autonumber
    participant Device as Bedside Monitor
    participant IoT as IoT Gateway
    participant BedSvc as Bed Mgmt Service
    participant Doc as ICU Doctor
    participant DB as Database

    loop Every 1 second
        Device->>IoT: MQTT: telemetry/vitals {hr: 140, spo2: 88}
    end

    IoT->>IoT: Analyze Stream (Rule: SpO2 < 90 for 10s)
    
    IoT->>BedSvc: triggerAlert({severity: HIGH, type: "Desaturation"})
    BedSvc->>DB: UPDATE patient_acuity SET score='CRITICAL'
    
    BedSvc->>Doc: Send Pager Alert "Bed 4: Patient Destabilizing"
    
    opt Auto-Suggestion
        BedSvc->>BedSvc: checkResources(Bed 4)
        alt Bed 4 has no Ventilator
            BedSvc->>Doc: Suggest Transfer -> "Bed 8 (Has Ventilator)"
        end
    end
```

---

**Last Updated**: January 2026
**Version**: 1.0
