# Database Schema - Smart Hospital ICU Management System

## Entity Relationship Diagram (ERD)

We use a Relational Database (PostgreSQL) for transactional data ensuring ACID compliance for critical bed allocations.

### Tables

#### `departments`
| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | UUID | PK | Department ID |
| `name` | VARCHAR(100) | NOT NULL | e.g., "Cardiology", "ICU A" |
| `floor` | INT | | Floor number |

#### `wards`
| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | UUID | PK | Ward ID |
| `department_id` | UUID | FK | Refers to departments.id |
| `name` | VARCHAR(100) | | e.g., "Ward 3B" |
| `gender_policy` | ENUM | | 'MALE', 'FEMALE', 'MIXED' |

#### `beds`
| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | UUID | PK | Bed ID |
| `ward_id` | UUID | FK | Refers to wards.id |
| `bed_number` | VARCHAR(20) | NOT NULL | Physical label |
| `type` | ENUM | | 'ICU', 'GENERAL', 'ISOLATION' |
| `capabilities` | JSONB | | e.g., `["VENTILATOR", "OXYGEN"]` |
| `current_status` | ENUM | | 'AVAILABLE', 'OCCUPIED', 'CLEANING', 'RESERVED' |
| `current_patient_id` | UUID | FK, NULL | Nullable, links to active patient |
| `version` | INT | DEFAULT 0 | For Optimistic Concurrency Control |

#### `patients`
| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | UUID | PK | Internal Patient ID |
| `mrn` | VARCHAR(50) | UNIQUE | Medical Record Number (External) |
| `first_name` | VARCHAR(100) | | |
| `last_name` | VARCHAR(100) | | |
| `dob` | DATE | | Date of Birth |

#### `admissions` (The Link Table)
| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | UUID | PK | Admission Event ID |
| `patient_id` | UUID | FK | |
| `bed_id` | UUID | FK | |
| `admitted_at` | TIMESTAMP | | Start time |
| `discharged_at` | TIMESTAMP | | End time (NULL if active) |
| `admitting_doctor_id`| UUID | FK | |

#### `reservations`
| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | UUID | PK | |
| `bed_id` | UUID | FK | |
| `patient_id` | UUID | FK | |
| `reserved_by` | UUID | FK | Staff ID |
| `expires_at` | TIMESTAMP | | Auto-release time |
| `status` | ENUM | | 'ACTIVE', 'FULFILLED', 'EXPIRED' |

#### `audit_logs` (Immutable)
| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | UUID | PK | |
| `entity_type` | VARCHAR | | 'BED', 'ADMISSION' |
| `entity_id` | UUID | | |
| `action` | VARCHAR | | 'STATUS_CHANGE', 'OVERRIDE' |
| `performed_by` | UUID | | Staff ID |
| `timestamp` | TIMESTAMP | | |
| `changes` | JSONB | | Old vs New values |

### Indexes
- `stations_bed_status_idx`: `ON beds (current_status, ward_id)` - fast lookup for available beds in a ward.
- `admissions_active_idx`: `ON admissions (patient_id) WHERE discharged_at IS NULL` - quickly find where a patient is right now.
