# MediPulse — Doctor Management Software (Phase 8: Production Ready)

> **Enterprise Healthcare SaaS for Physician Practice Management, Clinical Encounters, Digital Prescriptions, and Practice Analytics.**  
> Fully tested and production-ready with zero regressions across Phases 1–8.

---

## 1. System Architecture & Directory Structure

MediPulse is organized as a clean, unified healthcare system where both portals share the same backend clinical platform and UI tokens, while maintaining strictly separated interfaces and security perimeters:

```
doctor-saas/
├── client/
│   ├── src/
│   │   ├── doctor-portal/             # Dedicated Physician Application
│   │   │   ├── pages/                 # Overview, Profile, Schedule, Appointments, Patients EHR, e-Rx, Analytics, Settings
│   │   │   ├── components/            # Sidebar, Topbar, Consultations modals, Prescriptions modals
│   │   │   └── DoctorPortal.jsx       # Doctor Portal root orchestrator
│   │   ├── patient-portal/            # Dedicated Patient Application
│   │   │   ├── pages/                 # Overview, Appointments, Care Team, Medical Records, e-Rx & Print, Vitals, Reviews, Settings
│   │   │   ├── components/            # PatientSidebar, PatientTopbar
│   │   │   ├── patient-portal.css     # Tailored calm teal/emerald healthcare styling
│   │   │   └── PatientPortal.jsx      # Patient Portal root orchestrator
│   │   ├── shared/                    # Reusable Architecture
│   │   │   ├── auth/                  # DoctorLoginPage (/doctor/login), PatientLoginPage (/patient/login), PortalGateway (/)
│   │   │   └── components/            # StatCard, Badges, Shared types
│   │   ├── services/
│   │   │   └── api.js                 # Centralized API client (api for doctor, patientApi for patient)
│   │   ├── App.jsx                    # Root router with browser URL sync & role guards
│   │   └── main.jsx
│   └── package.json
└── server/                            # Centralized Backend Clinical Engine
    ├── src/
    │   ├── routes/                    # authRoutes (/auth), patientIntegrationRoutes (/patient-integration), etc.
    │   ├── controllers/               # Clinical business logic & authorization
    │   ├── middleware/                # requireAuth (doctor), requirePatientAuth (patient), security
    │   ├── models/                    # Mongoose schemas (Doctor, Patient, Appointment, Consultation, Prescription, AuditLog)
    │   └── services/                  # Audit trail, PDF stamping, notifications
    └── tests/                         # Master automated test suite (Phases 3–8)
```

---

## 2. Core Functional Modules

1. **Doctor Management Portal (`/doctor/dashboard`):**
   - **Dashboard & Overview:** Real-time KPI statistics (Patients, Appointments, Revenue, Pending Requests), today's schedule, priority reminders, and status badges.
   - **Doctor Profile:** Physician credentials, specialty, verified medical licenses, clinic addresses, contact details, and consultation pricing.
   - **Availability & Schedule:** Configurable weekly intervals, slot durations (15/30/45/60 min), buffer times, and blocked vacation/symposium leaves.
   - **Appointments Management:** Live booking calendar with conflict rejection, state transitions (`Pending` -> `Confirmed` -> `In Progress` -> `Completed`), atomic slot rescheduling, cancellation, and complete status audit trail.
   - **Patient EHR Records & Dossier:** Comprehensive patient directory with search, verified allergy badges (Confirmed, NKDA, Intake Pending), chronic condition histories, and past medications.
   - **Clinical Consultations:** SOAP clinical notes, chief complaints, examination observations, vitals, primary & differential diagnoses, digital signature locks, and clinical amendments.
   - **Digital Prescriptions (e-Rx):** Multi-item medication builder, dosage/frequency/route validator, digital signature stamp, versioned amendment revisions, and void reason tracking.
   - **Practice Analytics:** Filterable KPI summaries (today, 7d, 30d, 90d, custom dates), daily volume trends, consultation modalities breakdown, and patient feedback.

2. **Patient Health Portal (`/patient/dashboard`):**
   - **Health Overview:** Next visit countdown hero banner with 1-click video visit entry, 4 health metric cards, daily medication plan, recent doctor notes.
   - **Appointments:** Filter by upcoming/pending/past, book new in-person or telehealth visits, reschedule, cancel, and interactive Telehealth Video Room with chat simulation.
   - **Care Team:** Dr. Sarah Mitchell credentials, hospital affiliations, weekly clinic hours schedule, and consultation fee breakdown.
   - **Medical Records & EHR:** Verified drug allergies (Penicillin, Sulfa), chronic condition diagnoses with ICD-10 codes, lab reports with biomarker charts, and sanitized consultation notes.
   - **Prescriptions & Pharmacy:** Active digital prescriptions, dosage instructions, **Official Printable Prescription with clinic letterhead & signature**, and 1-click pharmacy refill requests.
   - **Vitals & Biomarkers:** Blood pressure, resting heart rate, SpO2, blood sugar, weight/BMI tracker with daily self-logging modal.
   - **Doctor Reviews:** Interactive 5-star review form with category ratings and verified patient reviews stream.
   - **Account & Settings:** Demographics, notification preferences, and FHIR R4 JSON medical record export.

---

## 3. Quick Start & Local Development

### Prerequisites
- Node.js v18.0.0 or higher
- (Optional) MongoDB 6.0+ local instance or MongoDB Atlas cluster. *(If not configured, the app operates automatically with built-in demonstration stores).*

### Installation
```bash
# Install frontend and backend dependencies
npm install --prefix client
npm install --prefix server
```

### Running Locally
```bash
# Terminal 1: Start Backend API (runs on port 5000)
npm run dev:server

# Terminal 2: Start Frontend Dev Server (runs on port 5173)
npm run dev:client
```
Open **`http://localhost:5173`** in your browser.

---

### Separate Portals & Dedicated Login Routes

MediPulse provides strictly separated login experiences with backend role verification:

#### 1. Portal Gateway & Selector
* **Route:** `/` or `/gateway` (or `?view=gateway`)
* Overview hub presenting both portals with direct entry cards.

#### 2. Doctor Portal Login
* **Route:** `/doctor/login` (or `?view=doctor-login`)
* **Email:** `dr.sarah.mitchell@medisync.io`
* **Password:** `DoctorPass123!` *(or `password`)*
* **Role Guard:** Backend verifies `role === 'doctor'`; patients are prohibited from entering.

#### 3. Patient Portal Login & Self-Registration
* **Route:** `/patient/login` (or `?view=patient-login`)
* **Email / MRN:** `eleanor.vance@example.com` *(or `MRN-2026-0841`)*
* **Password:** `PatientPass123!`
* **1-Click Demo Accounts:** Click on any patient card (*Eleanor Vance*, *Marcus Sterling*, *Sophia Chen*, *David Miller*) to sign in instantly.
* **New Patient Registration:** Includes full patient self-registration modal. Backend enforces `role: 'patient'`, prevents role escalation to doctor/admin, and generates a verified MRN.
* **Role Guard:** Backend verifies `role === 'patient'`; doctors are prohibited from entering as patients.

---

## 4. Automated Testing & Verification Suite

MediPulse includes 107 automated tests across 6 dedicated phase test suites:

```bash
# Run Master Test Runner (all suites)
npm test
```

### Individual Test Suites
```bash
node server/tests/phase3_appointment_tests.js      # Appointments, slots, conflicts
node server/tests/phase4_consultation_tests.js     # Patient records, allergies, consults
node server/tests/phase5_prescription_tests.js     # Multi-medication e-Rx, signature, voids
node server/tests/phase6_analytics_tests.js        # KPI statistics, period deltas, charts
node server/tests/phase7_security_tests.js         # JWT auth, rate limits, NoSQL injection
node server/tests/phase8_e2e_verification_tests.js # End-to-end regression & deployment checks
```

### Client Linting & Production Build
```bash
# Run Oxlint on React client
npm run lint:client

# Compile production client bundle
npm run build:client
```

---

## 5. Security & Privacy Guarantees

- **Authentication:** Signed JSON Web Tokens (JWT) with session revocation on logout.
- **Tenant Isolation:** Cross-physician data access strictly returns `403 Forbidden`.
- **NoSQL Injection Defense:** All incoming keys prefixed with `$` or containing `.` are stripped recursively.
- **HTTP Security Headers:** `X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY`, `X-XSS-Protection: 1; mode=block`, and `X-Powered-By` stripped.
- **Audit Logging:** Tamper-proof, immutable audit logs recorded for every clinical action.
- **Data Protection:** Database connection strings, passwords, and server stack traces are never exposed to the client or returned in error payloads.

---

## 6. Production Deployment

For complete production deployment instructions, environment variables configuration, Nginx reverse proxy setup, and PM2 clustering, refer to:

👉 **[DEPLOYMENT_AND_OPERATIONS_GUIDE.md](file:///c:/Users/HP/OneDrive/Desktop/doctor%20SaaS/DEPLOYMENT_AND_OPERATIONS_GUIDE.md)**
