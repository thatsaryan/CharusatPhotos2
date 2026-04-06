# 🎓 CHARUSAT Placement Setu — Interview Preparation Guide

> **Full-stack placement management platform** built with Node.js + Express + PostgreSQL (backend) and Flutter (mobile app), serving CHARUSAT University's entire placement process across 5 user roles.

---

## 1. Project Overview

### What Does It Solve?

| Before | After |
|--------|-------|
| Spreadsheets & manual sheets | One unified mobile app |
| WhatsApp groups for notifications | In-app + email notifications |
| No eligibility filtering | Smart eligibility engine |
| Paper attendance | GPS-verified digital attendance |
| No form management | Dynamic form builder |

### User Roles

| Role | Access Level | Key Capabilities |
|------|-------------|-----------------|
| **Student** | Personal | Browse drives, apply, mark attendance, fill forms |
| **Student Coordinator (SC)** | Department | All student actions + manage dept + create drives/announcements |
| **Faculty Coordinator (FC)** | Department | Manage drives, attendance, students for own department |
| **TPC** | University-wide | Full admin control over all departments and data |
| **Faculty** | Session-level | Create and manage attendance sessions |

---

## 2. Tech Stack

### Backend

| Technology | Why Used |
|------------|----------|
| **Node.js + Express** | Non-blocking I/O, perfect for API servers; Express gives clean routing and middleware support |
| **PostgreSQL** | Relational data with native array support (`departments[]`, `semesters[]`); ACID compliant for placement data integrity |
| **pg (node-postgres)** | Direct SQL with connection pooling (max 10); parameterized queries (`$1`, `$2`) prevent SQL injection; no ORM overhead |
| **JWT (jsonwebtoken)** | Stateless auth; HS256 algorithm locked (prevents `none` attack); includes issuer claim for extra validation; 7-day expiry |
| **Argon2id** | OWASP-compliant password hashing — hybrid of Argon2i (side-channel resistant) + Argon2d (GPU-attack resistant); 19MB memory, 2 iterations |
| **Nodemailer** | Bulk/OTP emails via pooled SMTP (5 connections, rate-limited); Gmail via App Password |
| **ExcelJS** | Server-side styled `.xlsx` generation for attendance export |
| **Helmet** | Auto-sets 15+ security headers (CSP, X-Frame-Options, HSTS) in one line |
| **HPP** | Prevents HTTP Parameter Pollution attacks |
| **express-rate-limit** | API: 200 req/15min; Auth: 10 req/15min — prevents brute-force |
| **express-validator** | Input validation + sanitization (email normalization, XSS escaping) |
| **cors** | Allows cross-origin requests from mobile app |
| **dotenv** | Keeps secrets (JWT_SECRET, DB password) out of source code |

### Frontend (Flutter Mobile App)

| Technology | Why Used |
|------------|----------|
| **Flutter** | Cross-platform (Android + iOS); hot reload; rich Material Design widget library |
| **Provider** | Lightweight global state management for auth state (official Flutter recommendation) |
| **http** | REST API calls with Bearer token in headers |
| **SharedPreferences** | Persistent local key-value store for JWT token and user profile |
| **Geolocator** | High-accuracy GPS for attendance geo-fencing verification |
| **device_info_plus** | Unique device ID for device binding (prevents credential sharing) |
| **permission_handler** | Runtime permission handling (location, storage) for Android 10+ and iOS |
| **excel (Dart)** | Client-side Excel export; platform-aware path (Downloads on Android, Documents on iOS) |
| **url_launcher** | Opens external URLs in browser |
| **share_plus** | Share files/content to other apps |
| **video_player** | In-app video playback |
| **open_file** | Opens generated Excel files with device's default app |
| **path_provider** | Gets platform-specific file directories |

---

## 3. Frontend Architecture

### App Startup Flow

```
main.dart → MaterialApp created
    └── SplashScreen
            ├── Token found in SharedPreferences
            │       └── Route to role-specific HomeScreen
            └── No token → LoginScreen
```

### Role-Based Routing

```dart
switch (user.role) {
  case 'student'             → StudentHomeScreen   // 4 tabs: Home, Attendance, Forms, Profile
  case 'student_coordinator' → ScHomeScreen        // 5 tabs: Home, Drives, Seminars, Students, Forms
  case 'fc'                  → FcHomeScreen        // Menu: SC Mgmt, Drives, Forms, Announcements
  case 'tpc'                 → TpcPlacementScreen  // Full admin view
}
```

### Clean Architecture (Folder Structure)

```
lib/
├── core/
│   ├── constants/api_constants.dart   ← All 80+ API endpoint URLs
│   ├── routes/app_routes.dart         ← 15 named route constants
│   └── utils/                         ← Shared utility functions
│
├── data/
│   ├── models/                        ← 10 data models (User, Company, Form, etc.)
│   └── services/                      ← 10 API service classes
│       ├── auth_service.dart
│       ├── company_service.dart
│       ├── form_service.dart
│       ├── export_service.dart
│       └── ...
│
├── features/                          ← Role-specific modules (fc/, sc/, student/)
│
└── presentation/
    ├── screens/                       ← 36 screens across 9 feature groups
    └── widgets/                       ← Reusable UI components
```

### API Call Flow

```
User taps "Apply"
    → _applyToDrive() in Screen
    → CompanyService.applyToCompany(driveId)
    → Read token from SharedPreferences
    → HTTP POST /api/applications { drive_id: "123" }
        with Authorization: Bearer <token>
    → Backend: auth check → eligibility check → DB insert
    → { success: true }
    → UI: "Applied ✅" snackbar + refresh drive list
```

---

## 4. Backend Architecture

### Request Lifecycle Pipeline

```
Incoming Request
    1. helmet()          → Security headers
    2. cors()            → Origin check
    3. express.json()    → Parse body (max 50KB)
    4. apiLimiter        → Rate limit check (200 req/15min)
    5. Router            → Match URL to module
    6. protect()         → Verify JWT (HS256 + issuer)
                           → Load user from DB
                           → Check is_active flag
                           → Validate device_id (students)
    7. restrictTo()      → RBAC role check → 403 if denied
    8. Controller        → Extract req.body params
    9. Service           → Business logic + DB query
   10. Response          → JSON sent back
```

### Modular MVC Pattern

Each feature is an **isolated module** with exactly 3 files:

```
module/
├── routes.js      → URL paths + middleware composition
├── controller.js  → Thin layer: extract params, call service, send response
└── service.js     → Business logic + all SQL queries
```

### 12 Backend Modules

| Module | Key Responsibility |
|--------|--------------------|
| **auth** | Signup (OTP), login (device binding), JWT, password reset (Argon2id) |
| **students** | Profile CRUD, student listing, SC role management, device reset |
| **companies** | Drive creation with eligibility criteria, CGPA normalization, email blast |
| **drive_rounds** | Round management with DB transactions, bulk email + in-app notifications |
| **applications** | Apply/withdraw with placed-student blocking |
| **attendance** | GPS geo-fenced self-mark, manual mark, live count, Excel export, auto-close job |
| **forms** | 6 question types, targeted distribution, idempotent submission (ON CONFLICT) |
| **announcements** | Department-scoped create/list/delete with email blast |
| **notifications** | Fetch, mark read, `createBulkNotifications()` (internal use) |
| **calendar** | Placement event CRUD (SC/FC/TPC) |
| **leaves** | Scaffold routes (TODO — Flutter service is complete) |
| **placement** | All drives view, applicant management, status updates, placed students |

---

## 5. End-to-End Flows

### Flow 1 — User Signup & Login

```
1. App opens → SplashScreen → no token → LoginScreen
2. Tap "Sign Up" → enter @charusat.edu.in email
3. POST /api/auth/send-signup-otp
   → Backend checks email not taken → generates 6-digit OTP → emails it
4. Enter OTP → POST /api/auth/signup (details + OTP + deviceId)
   → Verify OTP → Argon2id hash password → INSERT user + student profile
   → Sign JWT → create DB session
5. App saves token + user in SharedPreferences
6. Route to StudentHomeScreen based on role
```

### Flow 2 — Student Applies to a Drive

```
1. GET /api/companies/drives/student
   → Backend joins drives with student profile
   → Compares CGPA, marks, backlogs against criteria
   → Returns only eligible drives (is_eligible: true)
2. Drive card shows "Eligible" badge
3. Tap "Apply" → confirmation dialog (warns about round commitment)
4. POST /api/applications { drive_id }
   → Checks: already applied? already placed? drive active?
5. If OK → INSERT application
6. UI: "Applied ✅" snackbar, drive card flips to "Applied" badge
```

### Flow 3 — GPS Geo-Fenced Attendance

```
1. Faculty: POST /api/attendance/sessions (title, hall lat/lng bounds, dept/semester)
   → Email blast to targeted students
2. Faculty opens window: POST /sessions/:id/window/open (duration: 10 min)
3. Student taps "Mark Present" → app requests GPS permission
4. Geolocator: high accuracy, 10-second timeout
5. POST /api/attendance/sessions/:id/self-mark { lat, lng }
   → Backend checks:
      ✓ Session active? Window open? Not expired?
      ✓ Student in target department/semester?
      ✓ lat >= min_lat AND lat <= max_lat AND lng >= min_lng AND lng <= max_lng?
6. INSERT INTO attendance (ON CONFLICT DO NOTHING)
7. Student: "Marked present ✅"
8. SC screen live-polls count every 15 seconds
```

---

## 6. Security Implementation

### Authentication

| Mechanism | Implementation Detail |
|-----------|----------------------|
| **Password Hashing** | Argon2id — hybrid of Argon2i (side-channel resistant) + Argon2d (GPU resistant); 19MB memory, 2 iterations, 1 parallelism (OWASP min) |
| **JWT** | HS256, algorithm explicitly locked at sign AND verify (prevents `none algorithm attack`); issuer = `charusat-placement-setu`; 7-day expiry |
| **OTP** | 6-digit, 10-min expiry, stored in-memory Map; used for signup verification + password reset |
| **Device Binding** | First login binds `device_id` to user record; different device → `DEVICE_CHANGED` error; admin can reset |
| **Single Session** | On new login: DELETE all previous sessions → INSERT new; only one active device at a time |

### Authorization (RBAC)

```
protect()        → Verifies JWT + loads user from DB + checks is_active + device_id
restrictTo(...)  → Checks req.user.role against allowed roles list → 403 if denied
```

- **5 roles**: `student`, `student_coordinator`, `fc`, `tpc`, `faculty`
- **Department scoping**: FC/SC can only access their own department's data — enforced at **SQL level**
- **Account deactivation**: `is_active` flag checked in middleware, blocked at API level

### API Hardening

| Layer | Protection |
|-------|-----------|
| **Helmet** | 15+ security headers (CSP, X-Frame-Options, HSTS) |
| **Rate Limiting** | General: 200 req/15min; Auth: 10 req/15min (stricter for brute-force prevention) |
| **HPP** | Prevents HTTP Parameter Pollution |
| **Body Limit** | 50KB max JSON payload (prevents DoS) |
| **Parameterized SQL** | All queries use `$1, $2` syntax — SQL injection impossible |
| **Input Validation** | `express-validator`: email normalization, XSS escaping, domain lock |
| **Error Masking** | Production returns generic errors — no stack traces or DB schema leaked |

### Email Domain Lock

| Role | Required Domain |
|------|----------------|
| Student | `@charusat.edu.in` |
| Staff (TPC, FC) | `@charusat.ac.in` |

### Known Weak Areas *(mention these to show maturity)*

- OTP stored in-memory → should use **Redis** for horizontal scalability
- No **refresh token rotation** (single long-lived JWT)
- No **CSRF protection** (acceptable for pure mobile API; needed if web client added)
- No **audit logging** for admin actions
- Email credentials in `.env` → should use a cloud secrets manager in production

---

## 7. Database Design Highlights

- **PostgreSQL arrays** for multi-select fields (`departments[]`, `semesters[]`) — avoids junction tables for this use case
- **ON CONFLICT DO NOTHING** for idempotent operations (attendance, form submissions) — prevents duplicates without app-level checks
- **Connection pooling**: max 10 connections, 5s timeout, 30s idle timeout
- **CGPA normalization**: auto-detects 4.0 vs 10.0 scale during drive creation

---

## 8. Key Interview Answers

### "Tell me about your project"
> *"I built a full-stack placement management platform for CHARUSAT University. It has a Node.js + Express backend with PostgreSQL and a Flutter mobile app serving 5 user roles. Key features include GPS-verified attendance, smart drive eligibility matching, dynamic form builder, and a comprehensive security implementation with Argon2id password hashing, JWT authentication, and rate limiting."*

### "Explain the architecture"
> *"The backend follows modular MVC — each feature (auth, attendance, companies) is an isolated module with routes, controller, and service. The Flutter app uses clean architecture with core, data, and presentation layers. All API calls go through service classes, and data is typed in separate Dart model classes."*

### "What were your challenges?"
> *"The biggest challenge was GPS geo-fencing — handling varying accuracy, permission flows on Android/iOS, and defining bounding boxes per hall. Edge cases like 'permission denied forever', location services off, and GPS timeout all needed explicit handling. Another challenge was the eligibility engine that normalizes CGPA between 4.0 and 10.0 scales and matches against multiple simultaneous criteria."*

### "What would you improve?"
1. Replace in-memory OTP store with **Redis** for horizontal scalability
2. Add **refresh token rotation** with short-lived access tokens
3. Add **push notifications via FCM** for real-time alerts
4. Add **Docker containerization** for easier deployment
5. Add **audit logging** for all admin actions

### "How does it scale?"
> *"Connection pooling prevents DB exhaustion. Pooled SMTP handles bulk email. Rate limiting protects against abuse. Modular architecture means new features are isolated. For production scale: Redis for OTP/session storage, WebSockets instead of polling, and per-module DB connection limits."*

### "How would you test it?"
> *"Services are decoupled from controllers, making business logic independently unit-testable. On the Flutter side, clean architecture allows services to be mocked for widget tests. I'd add Jest + Supertest for backend unit/integration tests and Flutter widget tests with mock services."*

---

## 9. File Reference Summary

### Backend Key Files

| File | Purpose |
|------|---------|
| `server.js` | Entry point — Express setup, security middleware, route registration, background job |
| `config/db.js` | PostgreSQL pool (max 10 connections), exported for all modules |
| `middlewares/auth_middleware.js` | `protect()` + `restrictTo()` |
| `middlewares/rateLimiter.js` | `apiLimiter` (200/15min) + `authLimiter` (10/15min) |
| `utils/jwt_helper.js` | `signToken()` + `verifyToken()` with HS256 lock |
| `utils/email_helper.js` | Pooled SMTP, OTP template, bulk email, drive/form/announcement templates |
| `utils/otp_store.js` | In-memory OTP Map with 10-min expiry |
| `modules/attendance/` | 700+ lines — most complex module; GPS, manual mark, Excel export |
| `modules/forms/` | Dynamic builder, 6 question types, ON CONFLICT idempotency |
| `modules/drive_rounds/` | DB transactions (BEGIN/COMMIT/ROLLBACK), bulk notifications |

### Frontend Key Files

| File | Purpose |
|------|---------|
| `main.dart` | App entry, MaterialApp setup, initial route |
| `core/constants/api_constants.dart` | Single source of truth for 80+ API URLs |
| `data/models/user_model.dart` | Role helpers, placement eligibility (semester ≥ 6), serialization |
| `data/services/auth_service.dart` | Login/signup, SharedPreferences, SC role auto-detection |
| `data/services/export_service.dart` | Platform-aware Excel export (Android/iOS paths) |
| `screens/home/student_home_screen.dart` | 1147 lines — pagination, apply/unapply, hero stats |
| `screens/home/sc_home_screen.dart` | 2166 lines — most complex screen; 15s polling, GPS, session mgmt |
| `screens/forms/form_builder_screen.dart` | Dynamic form creator (6 question types) |
