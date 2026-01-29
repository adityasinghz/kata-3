# Actors and Use Cases - Smart Hospital ICU Management System

## Actors

### Primary Actors
1.  **Admission Desk Staff:** Responsible for initial patient entry and bed request creation.
2.  **Triage Nurse (ER):** Assesses patient criticality and requests emergency beds.
3.  **Ward Nurse:** Manages bed status (Occupied, Cleaning, Available) and initiates discharge.
4.  **Doctor / Intensivist:** Decides on medical necessity for ICU, approves transfers, and overrides allocations if needed.
5.  **Environmental Services (Housekeeping):** Cleans beds and marks them as "Ready" in the system.

### Secondary Actors
1.  **Hospital Administrator:** Views high-level dashboards and reports.
2.  **System Administrator:** Manages configurations, user roles, and integrations.
3.  **Automated Systems (IoT):** Vitals monitors, Admission/Discharge/Transfer (ADT) system.

## Use Cases

### UC-1: Bed Allocation & Admission
*   **Actor:** Admission Desk Staff / Triage Nurse
*   **Goal:** Assign a suitable bed to a new patient.
*   **Preconditions:** Patient record exists in ADT.
*   **Main Flow:**
    1.  Actor searches for available beds filtering by Ward Type (ICU, General) and Capability (Ventilator).
    2.  System displays list of available matched beds (sorted by proximity/priority).
    3.  Actor selects a bed and clicks "Reserve".
    4.  System marks bed as "Reserved" and sends notification to Ward Nurse.
*   **Alternate Flow (No Beds):**
    1.  System shows "No Beds Available".
    2.  Actor initiates "Waitlist" or "Overflow Protocol" (UC-6).

### UC-2: Emergency Override
*   **Actor:** Doctor / Intensivist
*   **Goal:** Immediately secure a bed for a life-threatening case.
*   **Preconditions:** "Code Blue" or equivalent emergency.
*   **Main Flow:**
    1.  Actor selects a bed (even if "Reserved" for a lower-priority patient).
    2.  Actor selects "Emergency Override" and provides proper authorization code/reason.
    3.  System re-assigns bed to critical patient.
    4.  System alerts functionality notifies the original reserver and Shift Manager of the override.

### UC-3: Patient Transfer (Inter-Departmental)
*   **Actor:** Ward Nurse (Sending) & Ward Nurse (Receiving)
*   **Goal:** Move patient from ICU to General Ward (step-down).
*   **Main Flow:**
    1.  Sending Nurse initiates "Transfer Request" to General Ward.
    2.  Receiving Nurse views request and allocates a bed.
    3.  System blocks both beds during transit.
    4.  Receiving Nurse confirms arrival; System releases ICU bed for cleaning.

### UC-4: Mark Bed Clean/Ready
*   **Actor:** Environmental Services
*   **Goal:** Make a dirty bed available for new patients.
*   **Main Flow:**
    1.  Actor receives "Cleaning Required" notification on mobile device.
    2.  Actor cleans the room.
    3.  Actor scans QR code on bed/room via mobile app.
    4.  System updates status from "Cleaning" to "Available".

### UC-5: Real-Time Acuity Update
*   **Actor:** IoT Monitor / Doctor
*   **Goal:** Update patient condition to reflect current resource needs.
*   **Main Flow:**
    1.  IoT device detects rapid vitals decline.
    2.  System pushes alert to Central Station.
    3.  Doctor confirms status change to "Critical".
    4.  System flags the bed as "ICU-Level Utilization" (tracking resource usage).

### UC-6: View Hospital Capacity Dashboard
*   **Actor:** Administrator
*   **Goal:** Strategic oversight.
*   **Main Flow:**
    1.  Actor logs in to Dashboard.
    2.  System shows Heatmap of hospital (Red=Full, Green=Empty).
    3.  System displays KPIs: "ER Wait Time," "ICU Occupancy %," "Pending Discharges."
