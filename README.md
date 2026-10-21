# Hospital SaaS — Integrated Clinical Management & Patient Portal System

> **Enterprise Healthcare & Hospital Management SaaS** featuring fully integrated **Doctor** and **Patient** portals, live real-time bidirectional appointment and clinical record synchronization, MongoDB Atlas configuration, and disk-persisted clinical data engine.

---

## 🌟 Key Capabilities & Integrated Portals

### 1. Dual Independent Portals
- **Doctor Portal:** Clinical encounters, appointments calendar with conflict prevention, SOAP notes, digital prescriptions (e-Rx) with signatures, patient EHR dossiers, and practice analytics.
- **Patient Portal:** Personal health overview, vital signs tracking (BP, HR, SpO2, BMI), real-time appointment booking with doctor selection, digital prescription viewer with printable slips, diagnostic laboratory chemistry & radiology reports, billing & payment receipts, doctor review & feedback, and emergency proxy management.

### 2. Cross-Dashboard Bidirectional Synchronization
- **Live Event Bridging:** Appointments requested from the Patient Portal instantly populate the Doctor's Appointments, Schedule, and Analytics.
- **Immediate Doctor Actions:** Confirmations, status changes, and clinical observations from the Doctor Portal reflect instantly on the Patient Portal.
- **Persistent Storage:** Synchronized automatically to `server/data/persisted_db.json` with debounced atomic writes.

### 3. MongoDB Atlas Ready & Configured
- **Configured MongoDB Key:** `al-5CbUfInxkbdbz92xUtUVVPsdj5WnwIbR_eq_Z-gtov9` pre-configured in `server/.env`.
- **Connection Diagnostics:** Accessible live via `GET /api/v1/system/status`.
- **Interactive Cluster Tester:** In **Settings > MongoDB & Real-Time Dashboard Integration**, you can test and connect any standard MongoDB Atlas URI (`mongodb+srv://...`) directly.

---

## 🔐 Demo Credentials & Quick Switch

| Portal | Demo Email | Demo Password | Autofill Tool |
| :--- | :--- | :--- | :--- |
| **Doctor Portal** | `dr.sarah.mitchell@medisync.io` | `DoctorPass123!` | 1-Click "Demo Doctor Credentials" button |
| **Patient Portal** | `munawwarali147@gmail.com` | `PatientPass123!` | 1-Click "Demo Patient Credentials" button |

*Tip: You can seamlessly switch between roles at any time using the role toggle in the top-right header or sidebar.*

---

## 🚀 Quick Start Guide

### Option A: 1-Click Launcher (Windows)
1. Double-click `install.bat` (only needed the first time to install dependencies).
2. Double-click `start.bat` to launch both backend and frontend servers simultaneously.
3. Open `http://localhost:5173` in your browser.

### Option B: Terminal Commands

```bash
# 1. Install dependencies
npm install --prefix server
npm install --prefix client

# 2. Start Backend API Server (Port 5000)
npm run dev:server

# 3. Start Frontend Dashboard (Port 5173) in a second terminal
npm run dev:client
```

- **Frontend URL:** http://localhost:5173
- **Backend API URL:** http://localhost:5000
- **System Health & DB Status:** http://localhost:5000/api/v1/system/status

---

## 📁 Repository Structure

```
hospital SaaS/
├── client/                     # React 19 + Vite Frontend Application
│   ├── src/
│   │   ├── components/         # Doctor UI, Modals, Topbar, Sidebar
│   │   ├── components/auth/    # DoctorLoginPage, PatientLoginPage, AuthGateway
│   │   ├── components/patient/ # PatientSidebar, PatientTopbar
│   │   ├── pages/              # Overview, Appointments, Patients, Settings, Analytics
│   │   ├── pages/patient/      # 10 Dedicated Patient Portal Pages
│   │   ├── services/api.js     # Unified Client API Service
│   │   ├── App.jsx             # Dual-Role Router & Live State Controller
│   │   └── index.css           # CarePulse Clinical Design System Tokens
│   ├── package.json
│   └── vite.config.js
├── server/                     # Node.js + Express Backend REST API
│   ├── .env                    # Environment config with MONGODB_API_KEY
│   ├── src/
│   │   ├── config/db.js        # MongoDB Connection & Status Reporter
│   │   ├── controllers/        # Doctor, Patient, Appointment, Consultation, Rx
│   │   ├── middleware/         # Security, JWT Auth, NoSQL Sanitizer, Rate Limit
│   │   ├── models/             # Mongoose Schemas (Doctor, Patient, Appointment, etc.)
│   │   ├── routes/             # Versioned v1 Express Routes
│   │   ├── services/sharedStore.js # Live Shared In-Memory & Disk Persistence Store
│   │   └── utils/              # Slot generator & appointment validators
│   ├── tests/                  # Verification test suites (Phases 1-8)
│   └── package.json
├── package.json                # Root package with monorepo scripts
├── install.bat                 # 1-Click Windows dependency installer
├── start.bat                   # 1-Click Windows dual-dashboard launcher
└── README.md                   # System documentation
```

---

## 🧪 Automated Test Verification

MediPulse and the integrated Hospital SaaS includes comprehensive test coverage:

```bash
npm test
```

Individual test executions:
```bash
node server/tests/phase3_appointment_tests.js
node server/tests/phase4_consultation_tests.js
node server/tests/phase5_prescription_tests.js
node server/tests/phase6_analytics_tests.js
node server/tests/phase7_security_tests.js
node server/tests/phase8_e2e_verification_tests.js
```
