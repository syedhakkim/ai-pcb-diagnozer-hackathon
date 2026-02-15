# AI PCB Diagnozer — Requirements

## 1. Overview
AI PCB Diagnozer is an AI-assisted diagnostic platform for laptop and desktop motherboard repair. It helps technicians identify likely fault areas using symptom selection, impedance inputs, OCR-based board/IC identification, and a knowledge base of repair patterns. It also provides access to schematics, BIOS files, and IC datasheets.

## 2. Goals
- Reduce diagnosis time from hours to minutes by guiding technicians with structured AI recommendations.
- Improve diagnosis accuracy by comparing measured impedance values against expected ranges and known fault patterns.
- Provide a single place to search board data, download schematics/BIOS, and view IC datasheets/pinouts.
- Maintain per-user diagnostic history for learning and repeatability.

## 3. Users and Roles
### 3.1 Technician (Primary User)
- Login, run diagnostics, view recommendations, download resources, and view personal history.

### 3.2 Admin (Optional)
- Manage board database, upload schematics/BIOS/datasheets, validate crowd-sourced entries, and moderate content.

## 4. Functional Requirements

### 4.1 Authentication
- FR-01: User can login using username and password.
- FR-02: Session/token-based authentication (JWT).
- FR-03: Role-based access (Technician/Admin) (optional for prototype).

### 4.2 Device Selection & Workflow
- FR-04: User can choose device type: Laptop or Desktop.
- FR-05: User can select or enter a fault symptom from a predefined list:
  - No power, Charging issue, Power on no display, Stuck on BIOS,
    Intermediate display, Display only not coming, Audio not working,
    WiFi not working, USB not working, GPU failure, RAM not detected.
- FR-06: User can proceed through the workflow step-by-step.

### 4.3 Board / IC Identification
- FR-07: User can enter board part number manually.
- FR-08: User can upload a clear board/IC photo (prototype: image upload only).
- FR-09: System performs OCR to extract board number / IC part number (if photo is used).
- FR-10: System fetches board metadata from database when found:
  - Model, Brand, Manufacturer, Color
  - Schematics available status
  - BIOS available status

### 4.4 Impedance Input (User-Entered)
- FR-11: User can enter impedance values for rails/points:
  - CSR, BATCOIL, 3V coil, 5V coil, RAM coil, VTT coil,
    Core coil1, Core coil2, Core coil3, GPU coil, PLTRST
- FR-12: Validate inputs (numeric, allow Ω / kΩ, handle blank fields).

### 4.5 AI Diagnosis and Recommendations
- FR-13: System compares entered impedance values with expected ranges for that board/model.
- FR-14: System flags abnormal values and classifies severity (Normal / Suspect / Short).
- FR-15: System produces recommended fault areas with confidence scores.
- FR-16: System highlights suspected regions using red square boxes on a PCB image/map (prototype: use a reference board image).
- FR-17: System shows recommended next checks/repair steps (text checklist).

### 4.6 Resources (Schematics / BIOS / Datasheet)
- FR-18: If available, user can download schematics files.
- FR-19: If available, user can download BIOS files.
- FR-20: User can view IC datasheet and pinout (PDF viewer or in-app image viewer).
- FR-21: Access to resources is permission controlled.

### 4.7 History
- FR-22: System saves each diagnosis result to user history:
  - Date/time, board/model, symptoms, impedance inputs, results, downloads used.
- FR-23: User can search/filter personal history by board/model/symptom/date.
- FR-24: User can open a past record and view the full report.

### 4.8 Not Found Handling
- FR-25: If board number / IC part number is not found, show a “Data Not Found” screen:
  - Suggest re-check entry, upload clearer photo, and try again later.
  - Optional: “Request to add to database”.

## 5. Non-Functional Requirements

### 5.1 Performance
- NFR-01: API responses for board lookup should be < 2 seconds (typical network).
- NFR-02: Diagnosis calculation should complete < 3 seconds for typical inputs.

### 5.2 Security
- NFR-03: Use HTTPS for all communication.
- NFR-04: JWT tokens for API access.
- NFR-05: Secure access to proprietary schematics/BIOS/datasheets.

### 5.3 Reliability & Availability
- NFR-06: Cloud-hosted deployment with backups for database and files.
- NFR-07: Graceful handling of OCR failures and missing data.

### 5.4 Usability
- NFR-08: Mobile-first UI with simple step-by-step flow.
- NFR-09: Clear error messages and validation hints.

### 5.5 Scalability
- NFR-10: Database design must support adding new boards, new fault patterns, and new device categories.

## 6. Assumptions
- Initial prototype supports selected common laptop/desktop board models.
- PCB highlight can use reference images and mapped regions in early versions.
- Impedance expected ranges may be generic per rail type for unknown boards, and board-specific when available.

## 7. Out of Scope (Prototype)
- Automatic component-level thermal/IR integration from external hardware.
- Fully automated probe-driven impedance measurement hardware (future roadmap).


- Full multi-tenant deployment (future roadmap).

## 8. Acceptance Criteria (Prototype)
- User can login and complete the full diagnostic flow end-to-end.
- User can input impedance values and receive fault recommendations + confidence.
- User can view highlighted PCB suspected area (red box overlay).
- User can download/view schematics/BIOS/datasheets when available.
- User history is saved and searchable.
