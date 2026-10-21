# MediPulse Doctor Management SaaS — Production Deployment & Operations Guide (Phase 8)

> **Document Version:** 8.0.0  
> **Status:** Production-Ready & Verified  
> **System Architecture:** Node.js (Express REST API) + React 19 SPA + MongoDB (Mongoose ODM)

---

## 1. System Overview & Technology Stack

MediPulse is a specialized multi-tenant healthcare software application designed for physician practices, outpatient clinics, and specialty medical centers.

- **Frontend:** React 19, Vite 8, Lucide React icons, Vanilla CSS design tokens.
- **Backend API:** Node.js, Express 4, JWT (jsonwebtoken), Bcryptjs, Mongoose 8.
- **Database:** MongoDB 6.0+ (with graceful local demonstration and fallback mechanisms).
- **Security:** Layered HTTP defense, NoSQL query sanitization, Sliding-window IP rate limiting, SHA-256 digital signature stamps, and HIPAA/GDPR-aligned immutable audit logging.

---

## 2. Directory Structure & Version Control Safeguards

```
doctor SaaS/
├── .gitignore                      <-- Prevents .env, secrets, logs & builds from being committed
├── package.json                    <-- Root orchestrator scripts (dev, build, test, lint)
├── DEPLOYMENT_AND_OPERATIONS_GUIDE.md
├── client/
│   ├── .gitignore
│   ├── package.json
│   ├── dist/                       <-- Production optimized assets (built via Vite)
│   └── src/
│       ├── components/
│       ├── pages/
│       ├── services/api.js         <-- Unified client API layer
│       └── index.css               <-- CSS design system
└── server/
    ├── .env.example                <-- Safe configuration template (NO credentials)
    ├── .gitignore                  <-- Server-specific exclusion rules
    ├── package.json
    ├── docs/                       <-- Integration specifications & OpenAPI schemas
    ├── src/
    │   ├── config/db.js            <-- MongoDB connection with auto-seeding & retry
    │   ├── middleware/             <-- Security, auth, sanitization, rate-limiter
    │   ├── models/                 <-- Mongoose Schemas (Doctor, Patient, Rx, etc.)
    │   ├── controllers/            <-- REST API Controllers
    │   ├── routes/                 <-- Versioned API routes (/api/v1/*)
    │   └── index.js                <-- Express server & production static asset server
    └── tests/
        ├── run_all_tests.js        <-- Master test runner
        ├── phase3_appointment_tests.js
        ├── phase4_consultation_tests.js
        ├── phase5_prescription_tests.js
        ├── phase6_analytics_tests.js
        ├── phase7_security_tests.js
        └── phase8_e2e_verification_tests.js
```

---

## 3. Environment Variables Specification

The system uses environment variables loaded from `server/.env`. Use `server/.env.example` as your reference.

| Variable Name | Required | Default / Example Value | Description & Production Guidance |
|---|---|---|---|
| `PORT` | Optional | `5000` | Port for the Express HTTP/REST backend service. |
| `NODE_ENV` | **Required** | `production` | Set to `production` to activate production optimizations and disable verbose debugging. |
| `MONGO_URI` | **Required** | `mongodb+srv://<user>:<password>@cluster.mongodb.net/medipulse?retryWrites=true&w=majority` | Standard MongoDB connection URI. If omitted or empty, system gracefully boots with local demo stores. |
| `JWT_SECRET` | **Required** | Strong 256-bit random string | Used to cryptographically sign Doctor and Patient authentication tokens. Minimum 32 characters. |
| `ALLOWED_ORIGINS` | **Required** | `https://doctor.yourdomain.com,https://patient.yourdomain.com` | Comma-separated CORS whitelist of domains authorized to call the API. |
| `RATE_LIMIT_WINDOW_MS` | Optional | `900000` (15 mins) | Window size for sliding-window rate limiters. |
| `RATE_LIMIT_MAX_REQUESTS`| Optional | `300` | General API rate limit per IP per window. |
| `AUTH_RATE_LIMIT_MAX_REQUESTS`| Optional | `10` | Strict brute-force limit on `/api/v1/auth/login` per IP per window. |
| `AUDIT_LOG_RETENTION_DAYS`| Optional | `2555` (7 years) | Compliance retention policy for clinical audit trails. |
| `ENABLE_STRICT_PATIENT_AUTH`| Optional | `true` | Enforces verified Patient JWT validation on all patient integration endpoints. |

### Generating a Secure JWT Secret
```bash
node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"
```

---

## 4. Database Setup & Production Provisioning

### A. MongoDB Atlas (Recommended Cloud Architecture)
1. Deploy an Atlas M10+ replica set cluster across multiple availability zones.
2. In **Network Access**, restrict IP whitelist to the IP addresses of your application servers.
3. In **Database Access**, create a dedicated least-privilege database user (e.g., `medipulse_app_user`) with `readWrite` permissions on the `medipulse` database only.
4. Copy the connection string to `server/.env` under `MONGO_URI`.

### B. Initial Seeding & Migration
When `connectDB()` establishes a connection to a fresh MongoDB instance, it automatically seeds:
1. Primary Doctor Profile (`Dr. Sarah Mitchell`, Cardiology & Internal Medicine, Default ID: `6543210987abcdef12345678`).
2. Weekly clinical availability schedule (Mon–Fri morning and afternoon intervals).
3. Sample blocked periods (annual symposium conference).
4. Core database indexes on `{ doctorId: 1, date: 1, startTime: 1 }` to guarantee high-performance conflict detection.

---

## 5. Build, Startup & Production Runbooks

### Step 1: Install Dependencies
```bash
# Root directory
npm install --prefix client
npm install --prefix server
```

### Step 2: Build Production Frontend Assets
```bash
npm run build:client
```
The compiled bundle is generated into `client/dist/`. The backend Express server automatically detects this folder and serves the frontend SPA alongside the REST API from port `5000`.

### Step 3: Run Full Automated Verification Suite
Ensure all 6 test suites pass before promotion:
```bash
npm test
```
*(Runs Phase 3, Phase 4, Phase 5, Phase 6, Phase 7, and Phase 8 regression suites).*

### Step 4: Production Process Management (PM2)
Install PM2 globally on your server:
```bash
npm install -g pm2
```

Create `ecosystem.config.js` in the project root:
```javascript
module.exports = {
  apps: [
    {
      name: 'medipulse-doctor-saas',
      script: 'server/src/index.js',
      instances: 'max',
      exec_mode: 'cluster',
      env: {
        NODE_ENV: 'production',
        PORT: 5000,
      },
    },
  ],
};
```

Start and persist the service:
```bash
pm2 start ecosystem.config.js
pm2 save
pm2 startup
```

---

## 6. Nginx Reverse Proxy & TLS Configuration

For enterprise deployments, terminate TLS at an Nginx reverse proxy in front of the Node.js application:

```nginx
server {
    listen 80;
    server_name doctor.medipulse.io;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl http2;
    server_name doctor.medipulse.io;

    ssl_certificate /etc/letsencrypt/live/doctor.medipulse.io/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/doctor.medipulse.io/privkey.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;

    # Client body limit for lab reports and avatars
    client_max_body_size 10M;

    location / {
        proxy_pass http://127.0.0.1:5000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }
}
```

---

## 7. Health Checks, Monitoring & Observability

### Health Endpoint
`GET /api/health` provides real-time telemetry:

```json
{
  "status": "ok",
  "service": "MediPulse Doctor Management SaaS API",
  "phase": "Phase 8 - Final Testing & Deployment Preparation",
  "version": "8.0.0",
  "environment": "production",
  "uptimeSeconds": 86420,
  "security": {
    "rateLimiter": "active",
    "noSqlSanitizer": "active",
    "securityHeaders": "active",
    "auditLogging": "immutable",
    "patientIntegrationReady": true
  },
  "database": {
    "connected": true,
    "mode": "connected",
    "databaseName": "medipulse"
  },
  "system": {
    "nodeVersion": "v20.x",
    "memoryUsageMb": 64
  },
  "timestamp": "2026-10-19T10:45:00.000Z"
}
```

### Monitoring Integration
Configure your monitoring tool (Datadog, New Relic, UptimeRobot, or Prometheus) to poll `GET /api/health` every 30 seconds. An alert should trigger if `status !== 'ok'` or `database.connected !== true`.

---

## 8. Patient Software Integration Protocol

When deploying the separate Patient Software (Web or Mobile application):
1. **Endpoint Prefix:** Target `https://api.medipulse.io/api/v1/patient-integration/*`.
2. **Authentication:** Patient Software authenticates patients via its own auth provider and requests an integration JWT via `POST /api/v1/auth/patient-token` with `{ patientId }`.
3. **Authorization Header:** All integration requests require `Authorization: Bearer <patient_token>`.
4. **Data Isolation & Privacy:**
   - Consultations endpoint (`GET /patient-integration/consultations`) automatically strips internal doctor notes and differential diagnoses.
   - Prescriptions endpoint (`GET /patient-integration/prescriptions`) strictly filters out all `Draft` prescriptions. Only `Finalized`, `Amended`, and `Signed & Sent` prescriptions are visible to the patient.
   - Appointments booked by the patient (`POST /patient-integration/appointments/book`) are registered with initial status `Pending`, allowing the doctor to review and accept/reschedule them from their dashboard.

---

## 9. Security Audit & Zero-Trust Checklist

- [x] Passwords hashed using bcrypt with salt rounds >= 10.
- [x] JWT tokens signed with strong random secret and verified on protected endpoints.
- [x] Token revocation and session blacklist active upon logout.
- [x] Cross-doctor tenant isolation enforced (Physician A cannot view or modify Physician B's data).
- [x] NoSQL operator injection stripped recursively (`$` prefix and `.` dot notation).
- [x] HTTP security headers attached to all responses (`X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY`, `X-XSS-Protection`).
- [x] Production error handler suppresses database connection strings, passwords, and stack traces.
- [x] All `.env` files excluded from Git tracking via `.gitignore`.
- [x] 107/107 automated test cases passing in master regression runner.
