Gentle Path – Full Stack Project Overview
=========================================

Gentle Path is a wellness-focused application built with a **React web frontend** and a **Go (Gin) backend**, using **Firebase Authentication**, **PostgreSQL**, and deployed on **Google Cloud Run (GCP)**.

The backend is complete and live.  
The iOS team will consume the same backend APIs to build a native iOS app.

This document explains:
- What the project does
- Project structure
- How to run frontend and backend
- Backend APIs (what exists and what they do)
- Database schema
- Cloud deployment details
- What the iOS team needs to know

---

1. High-Level Architecture
--------------------------

**Frontend (Web)**
- React + Vite
- Firebase Authentication (email/password)
- Calls backend APIs using Firebase ID token

**Backend (API)**
- Go (Gin framework)
- Firebase Admin SDK for auth verification
- PostgreSQL database
- File storage for PDFs (healing sheets) Bofore we used to have this but later we upgraded to GCS
- Google Cloud Storage (GCS)
- Accessed through a storage abstraction layer (upload_store)
- Deployed on Google Cloud Run

**Auth Flow**
1. User logs in via Firebase (frontend or iOS)
2. Firebase returns ID token
3. Token is sent to backend in `Authorization: Bearer <token>`
4. Backend verifies token and extracts user ID
5. Backend processes request

---

2. Project Structure
--------------------

```text
gentle-path-main/
│
├── backend/
│   ├── main.go                 # App entry point
│   ├── go.mod / go.sum         # Go dependencies
│   │
│   ├── auth/
│   │   ├── firebase.go         # Firebase Admin initialization
│   │   └── middleware.go       # Auth middleware
│   │
│   ├── routes/
│   │   ├── checkins.go         # User daily check-ins
│   │   ├── admin_checkins.go   # Admin views
│   │   ├── protocols.go        # Healing protocols
│   │   ├── protocol_acks.go    # Day / item acknowledgements
│   │   └── uploads.go          # Healing sheet uploads
│   │
│   ├── db/
│   │   ├── db.go               # DB connection
│   │   └── migrations.sql      # Schema
│   │
│   ├── storage/
│   │   └── upload_store.go     # File storage abstraction
│   │
│   ├── Dockerfile              # Cloud Run build
│   └── env.example             # Environment variables
│
├── frontend/
│   ├── index.html
│   ├── src/
│   │   ├── pages/
│   │   │   ├── Login.tsx
│   │   │   ├── Dashboard.tsx
│   │   │   ├── Protocol.tsx
│   │   │   └── Admin.tsx
│   │   │
│   │   ├── lib/
│   │   │   └── api.ts          # Backend API calls
│   │   │
│   │   └── firebase.ts         # Firebase config
│   │
│   ├── vite.config.ts
│   └── package.json
│
└── README.md
```

---

3. Backend Details
------------------

### Tech Stack
- Language: Go
- Framework: Gin
- Auth: Firebase Admin SDK
- DB: PostgreSQL
- Storage: Local (`/tmp`) on Cloud Run
- Google Cloud Storage (GCS)
- Accessed through a storage abstraction layer (upload_store)
- Hosting: Google Cloud Run

### Environment Variables

```ini
DATABASE_URL=postgres://user:pass@host:5432/dbname
FIREBASE_PROJECT_ID=xxxxx
GOOGLE_APPLICATION_CREDENTIALS=/path/to/service-account.json
GIN_MODE=release
```

---

4. Backend APIs
---------------

All APIs are under `/api` and **require Firebase auth** unless noted.

### Health
```bash
GET /health
```

### Check-ins
```bash
POST /api/checkins
GET /api/checkins
GET /api/admin/checkins
```

### Protocols
```bash
GET /api/protocols
GET /api/protocols/:id
```

### Protocol Acknowledgements
```bash
POST /api/protocols/ack
GET /api/protocols/item-acks
POST /api/protocols/item-acks
```

### Uploads
```bash
POST /api/uploads
GET /uploads/:file
```

---

5. Database Schema (Simplified)
-------------------------------

```text
users:
- id (uuid)
- firebase_uid
- email
- created_at

checkins:
- id
- user_id
- mood
- notes
- created_at

protocols:
- id
- title
- description

protocol_days:
- id
- protocol_id
- day_number

protocol_item_acks:
- id
- protocol_item_id
- user_id
- confirmed_at
```

---

6. Frontend Details
-------------------

### Tech Stack
- React
- Vite
- TypeScript
- Firebase Web SDK

### Pages
| Page | Purpose |
|-----|--------|
| Login | User authentication |
| Dashboard | Daily guidance + check-in |
| Protocol | Healing protocol progress |
| Admin | Admin-only views |

### Running Frontend
```bash
cd frontend
npm install
npm run dev
```

```ini
VITE_API_BASE_URL=https://<cloud-run-url>
```

---

7. Cloud Deployment (Backend)
-----------------------------

- Google Cloud Run
- Dockerized deployment

Backend URL example:
```text
https://gentle-path-api-xxxxx-uc.a.run.app
```

---

8. What iOS Team Needs to Know
------------------------------

- Use Firebase iOS SDK
- Send Firebase ID token with every API call
- Same backend as web
- No iOS-specific APIs required

---

9. Common Gotchas
-----------------

- Always refresh Firebase token
- Cloud Run allows writes only to `/tmp`
- All APIs require `Authorization: Bearer <token>`

---

10. Contact / Ownership
-----------------------

Backend completed and maintained by:  
**Kalyan**

Frontend Web: Complete  
iOS App: Implemented by iOS team using existing APIs
