# API Specification - Smart Hospital ICU Management System

All endpoints are prefixed with `/api/v1`.
Authentication: Bearer Token (JWT).

## 1. Bed Management

### Get Beds
`GET /beds`
Retrieves a list of beds with optional filtering.

**Query Parameters:**
- `ward_id` (UUID): Filter by ward.
- `status` (String): 'AVAILABLE', 'OCCUPIED', etc.
- `capability` (String): 'VENTILATOR', 'ISOLATION'.

**Response:**
```json
[
  {
    "id": "123e4567-e89b-12d3-a456-426614174000",
    "bed_number": "ICU-01",
    "status": "AVAILABLE",
    "capabilities": ["VENTILATOR", "ECG_MONITOR"]
  }
]
```

### Update Bed Status
`PATCH /beds/{id}/status`
Manually update bed status (e.g., Housekeeping marking as Clean).

**Body:**
```json
{
  "status": "AVAILABLE",
  "notes": "Deep cleaned via UV protocol"
}
```

## 2. Allocations & Reservations

### Create Reservation
`POST /reservations`
Reserves a bed for a patient.

**Body:**
```json
{
  "bed_id": "UUID",
  "patient_id": "UUID",
  "priority": "URGENT",
  "duration_hours": 4
}
```

### Emergency Override
`POST /allocations/override`
Forcefully assigns a bed, cancelling any existing future reservations.

**Body:**
```json
{
  "bed_id": "UUID",
  "patient_id": "UUID",
  "reason_code": "CODE_BLUE",
  "override_reason": "Patient going into cardiac arrest"
}
```

## 3. Notifications

### Get Active Alerts
`GET /alerts`
Get alerts relevant to the current user's role/ward.

**Response:**
```json
[
  {
    "id": "alert-99",
    "type": "CLEANING_REQUEST",
    "message": "Bed ICU-04 needs cleaning (Discharge)",
    "timestamp": "2023-10-27T10:00:00Z"
  }
]
```
