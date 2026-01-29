# Features - Smart Hospital ICU Management System

## F1: Interactive Bed Map (The "Air Traffic Control" View)
**Description:** A visual, map-based interface representing the physical layout of the hospital wads.
**Key Capabilities:**
- **Color-Coded Status:** Green (Available), Red (Occupied), Yellow (Reserved), Grey (Cleaning), Blue (Maintenance).
- **Hover Details:** Hovering over a bed shows Patient ID (anonymized), Admission Time, and Acuity Level.
- **Filters:** Filter by Department (ICU, Pediatrics), Equipment (Ventilator, Dialysis), or Gender.
**Technical Note:** Uses Vector Graphics (SVG) on frontend; WebSocket subscription for real-time state changes.

## F2: Smart Allocation Engine
**Description:** Rule-based background service that recommends the best bed.
**Key Capabilities:**
- **Matching Logic:** Matches Patient Needs (Isolation, Negative Pressure) with Bed Capabilities.
- **Proximity Logic:** Prioritizes beds closer to the nursing station for critical patients.
- **Conflict Resolution:** Prevents double-booking by locking records during the reservation transaction (Optimistic Concurrency Control).

## F3: Mobile Task Management for Environmental Services
**Description:** Specialized mobile view for housekeeping staff.
**Key Capabilities:**
- **Job Queue:** List of "Dirty" beds sorted by priority (e.g., ER beds first).
- **One-Tap Completion:** Simple UI to mark "Clean started" and "Clean finished".
- **Turnaround Timer:** Tracks time taken to clean; gamification/metrics for efficiency.

## F4: Automated ADT Integration
**Description:** Middleware to sync with legacy Hospital Information Systems (HIS).
**Key Capabilities:**
- **HL7/FHIR Listener:** Listens for ADT_A01 (Admit), ADT_A02 (Transfer), ADT_A03 (Discharge) messages.
- **Auto-Sync:** Updates local bed database immediately upon receiving HL7 messages.
- **Fallback:** Manual reconciliation interface if HIS message fails.

## F5: Critical Alert Broadcast
**Description:** Multi-modal notification hub.
**Key Capabilities:**
- **Role-Based Routing:** "Code Blue in ICU Bed 4" -> Sent to ICU Team + ER Doctors. "Bed 4 Dirty" -> Sent to Housekeeping.
- **Escalation Policy:** If an alert is not acknowledged within $N$ minutes, call the on-call phone.
- **Audit:** Logs who received the alert and when it was acknowledged.

## F6: Predictive Capacity Analytics
**Description:** AI-driven forecasting module.
**Key Capabilities:**
- **Trend Analysis:** "Fridays at 6 PM usually see a spike in ER admissions."
- **Staffing Recommendations:** "Open Ward B overflow wing based on predicted influx."
- **Reports:** Bottleneck analysis (e.g., "Patients wait 2 hours for Bed Cleaning on Sundays").
