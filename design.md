# AI PCB Diagnozer — Design

## 1. High-Level Architecture
The solution is designed as a cloud-backed mobile-first application with an AI analysis service and databases for boards, components, and repair intelligence.

### 1.1 Components
- Mobile App (Flutter)
- Web Dashboard (ReactJS) (optional for prototype)
- API Backend (Django/FastAPI)
- AI Analysis Engine (Python service/module)
- Databases:
  - Board & Component DB (metadata + availability flags)
  - Repair Intelligence DB (patterns, rules, historical cases)
- File Storage:
  - Schematics, BIOS, Datasheets (stored securely, downloadable via signed URLs)

## 2. Data Flow
1) User logs in → receives JWT token  
2) User selects Laptop/Desktop and selects symptom  
3) User enters board number or uploads photo  
4) Backend performs:
   - OCR (if image) → extracts board/IC text
   - Board lookup → returns model/brand + resource availability  
5) User enters impedance values  
6) AI engine analyzes inputs:
   - Range checks
   - Pattern matching vs known faults
   - Confidence scoring  
7) Backend returns:
   - Fault recommendations + confidence
   - PCB region IDs to highlight  
8) App displays:
   - Red square overlay on PCB image
   - Suggested repair steps  
9) History record saved and accessible later.

## 3. AI / Logic Design (Prototype Friendly)
This system can be implemented without heavy training initially, using a hybrid approach:

### 3.1 Rule + Pattern Hybrid
- Range-based anomaly detection:
  - Each rail has expected impedance ranges (board-specific if known; generic otherwise).
- Severity classification:
  - Normal, Suspect, Short (very low impedance), Open/High (e.g., PLTRST high impedance).
- Pattern matching:
  - Map combinations of abnormal rails to likely board regions:
    - CSR abnormal → PCH/chipset zone
    - RAM/VTT abnormal → RAM power/termination zone
    - Core coils abnormal → CPU VRM zone
    - GPU coil abnormal → GPU VRM/BGA zone
    - 3V/5V rails abnormal → main power management zone

### 3.2 Confidence Score
A simple scoring approach (prototype):
- Start at 50%
- Add points for strong matches (e.g., “short” class)
- Add points when multiple rails support the same region
- Cap at 95%
This yields an interpretable confidence score.

### 3.3 Future AI Upgrade
- Train ML models using historical labeled cases:
  - Inputs: symptoms + impedance vector + board family
  - Output: fault region(s) + probability
- Add image-based detection:
  - Identify zone/region from PCB image and known layout templates

## 4. PCB Highlighting Design
### 4.1 Region Mapping
For each supported board/model, store a set of regions:
- region_id (e.g., CPU_VRM, PCH_ZONE, GPU_VRM, RAM_ZONE)
- bounding boxes (x, y, w, h) in normalized coordinates (0..1)
- optional label and icon

### 4.2 Overlay Rendering
- The mobile app loads PCB reference image for the board
- For each region_id returned by backend, draw a red square/rectangle overlay
- Allow tap-to-view details: why flagged + what to check next

## 5. Backend API Design (Suggested Endpoints)
### Auth
- POST /api/auth/login → {token}
- GET /api/auth/me → profile

### Board / IC Lookup
- GET /api/boards/lookup?board_no=...  
- POST /api/boards/ocr → upload image → extracted text + match candidates
- GET /api/components/lookup?ic_part_no=...

### Diagnosis
- POST /api/diagnosis/run
  - input: device_type, symptoms[], board_id(optional), impedance_values{...}
  - output: faults[], confidence, regions[], steps[], resources_available

### Resources
- GET /api/resources/schematics/{board_id}
- GET /api/resources/bios/{board_id}
- GET /api/resources/datasheet/{component_id}

### History
- POST /api/history/save
- GET /api/history/my?search=&date_from=&date_to=
- GET /api/history/{history_id}

## 6. Database Design (Simplified)
### 6.1 Tables
- users(id, name, role, email, password_hash, created_at)
- boards(id, board_no, model, brand, manufacturer, color, device_type)
- board_resources(board_id, schematics_url, bios_url, status_flags)
- components(id, part_no, name, category, manufacturer, package, datasheet_url, pinout_image_url)
- impedance_profiles(board_id, rail_name, min_value, max_value, unit)
- fault_patterns(id, device_type, symptom_tag, rule_json, region_id, recommended_steps)
- history(id, user_id, board_id, symptoms_json, impedance_json, result_json, created_at)

### 6.2 Storage
- Use PostgreSQL/MySQL for structured data
- Use S3 (or equivalent) for files: schematics, BIOS, datasheets

## 7. UI Screen Map (Mobile)
1. Login
2. Disclaimer
3. Choose Device (Laptop/Desktop)
4. Fault Symptom Selection (dropdown + list)
5. Board Identification (type board no / upload photo)
6. Board Info (auto-filled availability status)
7. Impedance Input (forms for CSR/BAT/3V/5V/RAM/VTT/Core1/2/3/GPU/PLTRST)
8. AI Diagnose (button) + Result
9. Recommended Fault Areas (red box overlay)
10. Schematics/BIOS/Datasheet download & viewer
11. User History (personal history list + filters)
12. Data Not Found screen (try later + upload clear photo)

## 8. Security & Access Control
- JWT authentication for all APIs
- Resource URLs delivered via signed links (time-limited)
- Role-based permission for admin upload/management
- Audit: store downloads and diagnosis logs per user

## 9. Deployment (Prototype)
- Backend + AI service containerized using Docker
- Deploy on AWS (EC2/ECS) or similar
- RDS for database, S3 for file storage
- CI/CD optional: GitHub Actions

## 10. Testing Plan (Prototype)
- Unit tests: impedance parsing, rule evaluation, scoring logic
- API tests: auth, lookup, diagnosis, history
- UI tests: validation, error handling, step navigation
- Performance checks: lookup latency, diagnosis latency

## 11. Roadmap
- Add more board templates + region maps
- Train ML model using real repair cases
- Add camera live OCR and auto-board detection
- Integrate IoT-enabled measurement device for automatic impedance capture
- Build nationwide repair intelligence network with anonymized insights
