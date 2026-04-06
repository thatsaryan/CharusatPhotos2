================================================================================
CHARUSAT PLACEMENT SETU --- INTERVIEW PREPARATION GUIDE
================================================================================

This file explains the COMPLETE project in simple English. Use this to
prepare for interviews. Read section by section.

================================================================================ 1.
PROJECT OVERVIEW
================================================================================

What is this project? - A full-stack mobile app + backend for managing
university placements - Used by CHARUSAT University to handle placement
drives, attendance, forms

What does it solve? - Before this: spreadsheets, WhatsApp groups, manual
attendance sheets - After this: one app handles everything --- drives,
attendance, forms, notifications

Who uses it? (5 user roles) - STUDENT → Browse drives, apply, attend
seminars, fill forms - STUDENT COORD. → Everything a student does +
manage department + create announcements + create drives - FACULTY
COORD. → Manage drives, attendance, students for their department - TPC
→ Full admin control over all departments - FACULTY → Create and manage
attendance sessions

How is it built? - Backend: Node.js + Express + PostgreSQL - Mobile App:
Flutter (Dart) - Both talk through REST APIs

================================================================================
2. TECH STACK (WITH REASONS --- VERY IMPORTANT FOR INTERVIEW)
================================================================================

BACKEND: \-\-\-\-\-\-\-- Technology Why Used \-\-\-\-\-\-\-\-\-\--
\-\-\-\-\-\-\-- Node.js + Express Fast, non-blocking I/O. Perfect for
API servers. Express gives clean routing and middleware support.

PostgreSQL Relational data (students, drives, applications have
relationships). Supports arrays (used for departments\[\],
semesters\[\]). ACID compliant --- important for placement data
integrity.

pg (node-postgres) Direct SQL with connection pooling (max 10
connections). No ORM overhead --- full control over queries. All queries
use \$1, \$2 parameterization (prevents SQL injection).

JWT (jsonwebtoken) Stateless authentication. Token sent with every
request. Configured with HS256 algorithm lock (prevents \'none\'
attack). Includes issuer claim for extra validation.

Argon2id (argon2) Password hashing. Better than bcrypt because:  -
Argon2i = side-channel resistant  - Argon2d = GPU attack resistant  -
Argon2id = HYBRID of both (best choice) OWASP-compliant config: 19MB
memory, 2 iterations.

Nodemailer Sending emails (OTP, drive notifications, announcements).
Uses pooled SMTP transport (5 connections, rate limited). Gmail
integration via App Password.

ExcelJS Server-side Excel generation for attendance export. Creates
styled .xlsx files with headers and formatting.

Helmet Auto-sets security HTTP headers (CSP, X-Frame-Options, HSTS). One
line of code, massive security improvement.

HPP Prevents HTTP Parameter Pollution attacks.

express-rate-limit Rate limiting. API: 200 req/15min. Auth: 10
req/15min. Prevents brute-force attacks on login/OTP endpoints.

express-validator Input validation + sanitization (email normalization,
XSS escaping).

cors Allows cross-origin requests from mobile app.

dotenv Loads environment variables from .env file. Keeps secrets
(JWT_SECRET, DB password) out of code.

FRONTEND (MOBILE APP): \-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--
Technology Why Used \-\-\-\-\-\-\-\-\-\-- \-\-\-\-\-\-\-- Flutter
Cross-platform (Android + iOS from one codebase). Fast development with
hot reload. Rich widget library for Material Design.

Provider State management. Simple, lightweight, official recommendation.
Used for sharing auth state across screens.

http package Makes REST API calls to backend. Sends Bearer token in
headers for authentication.

SharedPreferences Local key-value storage on device. Stores JWT token
and user data for persistent login.

Geolocator Gets GPS coordinates for attendance verification. Checks if
student is inside the hall boundaries.

device_info_plus Gets unique device ID for device binding feature.
Prevents credential sharing among students.

permission_handler Handles runtime permissions (location, storage).
Required for Android 10+ and iOS.

excel (Dart) Client-side Excel generation for form response export.
Platform-aware: saves to Downloads (Android) or Documents (iOS).

Google Fonts Custom typography (not default system fonts).

url_launcher Opens external URLs in browser.

share_plus Share files/content to other apps.

video_player Plays video content within app.

open_file Opens generated Excel files with device\'s default app.

path_provider Gets platform-specific directories (Downloads, Documents).

================================================================================
3. FRONTEND EXPLANATION (FLUTTER APP)
================================================================================

HOW THE APP STARTS: \-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-- 1. main.dart
runs → MaterialApp created 2. First screen: SplashScreen 3. SplashScreen
checks SharedPreferences for saved token 4. If token exists → get saved
user → route to correct home screen based on role 5. If no token → go to
LoginScreen

ROLE-BASED ROUTING (very important):
\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--
After login, the app checks user.role and routes to the correct home
screen:

role = \'student\' → StudentHomeScreen (4 tabs: Home, Attendance, Forms,
Profile) role = \'student_coordinator\' → ScHomeScreen (5 tabs: Home,
Drives, Seminars, Students, Forms) role = \'fc\' → FcHomeScreen (menu:
SC Management, Drives, Forms, Announcements) role = \'tpc\' →
TpcPlacementScreen (full admin view of all drives and data)

The RoleRouterScreen widget handles this routing with a simple switch
statement.

APP STRUCTURE (Clean Architecture):
\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--
lib/ ├── core/ │ ├── constants/api_constants.dart → All 80+ API endpoint
URLs in one place │ ├── routes/app_routes.dart → Named route strings (15
routes for 5 roles) │ └── utils/ → Shared utility functions │ ├── data/
│ ├── models/ → 10 data models (classes that hold data) │ │ ├──
user_model.dart → User with role helpers (isStudent, isTpc, isFc, etc.)
│ │ ├── company_model.dart → Company + drive data with eligibility check
│ │ ├── application_model.dart → Job application status tracking │ │ ├──
form_model.dart → Form questions + responses │ │ ├──
notification_model.dart → In-app notification data │ │ ├──
leave_model.dart → Leave request data │ │ ├── calendar_event_model.dart
→ Placement calendar events │ │ ├── sc_student_model.dart → Student data
for SC management │ │ └── student_profile_model.dart → Detailed student
profile │ │ │ └── services/ → 10 API service classes (talk to backend) │
├── auth_service.dart → Login, signup, OTP, logout, token management │
├── company_service.dart → Get drives, apply, unapply, applicant data │
├── student_service.dart → Student profile CRUD │ ├── sc_service.dart →
SC management (assign/remove coordinators) │ ├── form_service.dart →
Create forms, submit responses, view responses │ ├──
announcement_service.dart → Create and view announcements │ ├──
calendar_service.dart → Calendar event CRUD │ ├── leave_service.dart →
Request leave, approve/reject │ ├── notification_service.dart → Fetch
and mark notifications as read │ └── export_service.dart → Client-side
Excel export for form responses │ ├── features/ → Role-specific feature
modules │ ├── fc/ → Faculty Coordinator screens │ ├── sc/ → Student
Coordinator specific screens │ └── student/ → Student-specific screens
(profile, companies) │ └── presentation/ ├── screens/ → 36 screens
across 9 feature groups │ ├── auth/ → login, signup, OTP, forgot
password, splash, role router │ ├── home/ → student_home, sc_home,
fc_home (role-specific dashboards) │ ├── attendance/ → student
attendance, faculty attendance, TPC attendance │ ├── placement/ → create
drive, applicants, rounds, placed students │ ├── forms/ → form builder,
form fill, form list, form responses │ ├── announcements/→ create + view
announcements │ ├── notifications/→ notification list │ ├── calendar/ →
placement calendar │ └── leaves/ → student leave, TPC leave management │
└── widgets/ → Reusable UI components

HOW API CALLS WORK: \-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-- 1. User does
something (tap Apply button) 2. Screen calls service method
(companyService.applyToCompany(driveId)) 3. Service reads token from
SharedPreferences 4. Service makes HTTP POST with Bearer token in
headers 5. Backend processes request, returns JSON response 6. Service
parses JSON into a Dart model 7. Screen updates UI using setState()

Example flow: User taps \"Apply\" → \_applyToDrive() →
CompanyService.applyToCompany() → HTTP POST to /api/applications with
{drive_id: \"123\"} → Backend checks auth, checks eligibility, inserts
into db → Returns {success: true} → UI shows \"Applied ✅\" snackbar,
refreshes drive list

STATE MANAGEMENT: \-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-- - Uses setState()
for local screen state (loading, lists, errors) - Uses SharedPreferences
for persistent data (token, user JSON) - Uses Provider for sharing auth
state globally - Each screen loads its own data in initState()

NAVIGATION: \-\-\-\-\-\-\-\-\-\-- - Named routes defined in AppRoutes
class - Most navigation uses MaterialPageRoute with Navigator.push() -
Some use Navigator.pushReplacementNamed() (login → home, replaces
stack) - Logout uses Navigator.pushAndRemoveUntil() (clears entire
navigation stack)

IMPORTANT FRONTEND FEATURES:
\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-- - Student
Home has pagination (shows 2 drives, \"Show More\" loads 3 more) - SC
Home polls seminars every 15 seconds for live attendance count - GPS
attendance requests HIGH accuracy location with 10-second timeout -
Drive cards show dynamic badges (Eligible/Applied/Not Eligible/Closed) -
Already-placed students get blocked from applying (ALREADY_PLACED
check) - Apply confirmation dialog warns about commitment to all
rounds - Pull-to-refresh on all home screens using RefreshIndicator -
Excel export handles Android vs iOS storage paths differently

================================================================================
4. BACKEND EXPLANATION
================================================================================

ENTRY POINT --- server.js:
\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-- This is where
everything starts: 1. Creates Express app 2. Applies security middleware
in order:  - helmet() → Security headers  - cors() → Cross-origin access
 - express.json(50KB) → Parse JSON body, limit size  - apiLimiter → Rate
limit all routes 3. Registers all module routes: /api/auth → auth_routes
/api/students → student_routes /api/companies → company_routes +
drive_round_routes /api/applications → application_routes
/api/attendance → attendance_routes /api/announcements →
announcement_routes /api/notifications → notification_routes /api/forms
→ form_routes /api/calendar → calendar_routes /api/leaves → leave_routes
/api/sc → sc_routes /api/placement → placement_routes 4. Starts
background job: every 5 minutes, auto-close expired attendance sessions
5. Starts server on port from .env

ARCHITECTURE PATTERN --- MODULAR MVC:
\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--
Each feature is a separate folder (module) with 3 files:

module/ ├── routes.js → Defines URL paths + which middleware to apply
├── controller.js → Handles request/response (thin layer) └── service.js
→ Business logic + database queries

Example: auth module auth_routes.js → POST /login calls
authController.login auth_controller.js → Extracts email/password from
req.body, calls authService.login() auth_service.js → Verifies password
with Argon2, checks device binding, signs JWT

REQUEST LIFECYCLE: \-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-- Every API
request goes through this pipeline:

 1. REQUEST comes in 2. helmet() adds security headers 3. cors() checks
origin 4. express.json() parses body (max 50KB) 5. apiLimiter checks
rate limit (200 req/15min) 6. Router matches URL to correct module 7.
protect() middleware:  - Reads token from Authorization header  -
Verifies JWT (checks algorithm = HS256, issuer =
charusat-placement-setu)  - Queries database for user by ID from token
 - Checks if account is active (is_active = true)  - For students:
checks device_id matches token\'s deviceId  - Attaches user object to
req.user 8. restrictTo(\'tpc\', \'fc\') middleware:  - Checks if
req.user.role is in the allowed list  - If not → 403 Forbidden 9.
Controller function runs 10. Service function does business logic + DB
query 11. RESPONSE sent back as JSON

DATABASE CONNECTION --- config/db.js:
\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-- -
Uses pg.Pool (connection pooling) - Max 10 connections, 5 second
timeout, 30 second idle timeout - All queries use parameterized syntax:
pool.query(\'SELECT \* FROM users WHERE id = \$1\', \[id\]) - This
prevents SQL injection (the #1 web vulnerability)

12 BACKEND MODULES EXPLAINED:
\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--

1\. AUTH MODULE (auth/)  - Signup: validate email domain
(@charusat.edu.in for students, \@charusat.ac.in for staff)  - OTP
verification: 6-digit code, stored in memory, 10-min expiry  - Password
hashing: Argon2id with OWASP config  - Login: verify password → check
device binding → sign JWT → create session  - Device binding: first
login binds device, different device = rejected  - Password reset:
OTP-based reset flow

2\. STUDENTS MODULE (students/)  - Get/update student profile (CGPA,
marks, backlogs, phone)  - List all students (with department filter)  -
Assign/remove SC role  - Reset device binding (admin only)

3\. COMPANIES MODULE (companies/)  - Create companies  - Create drives
with eligibility criteria: \* Min CGPA (auto-normalizes between 4.0 and
10.0 scales) \* Min 10th/12th marks \* Max active backlogs \* Education
type filter (B.Tech, M.Tech, MCA, etc.) \* Target departments and
institutes  - Sends email blast + in-app notification when drive created
 - Get eligible drives (filtered per student\'s profile)  - Get all
drives (for TPC/FC/SC)

4\. DRIVE ROUNDS MODULE (companies/drive_round_routes.js)  - Create
round for a drive  - Select students for a round (with database
transaction)  - Send personalized email to each selected student  -
Create in-app notification for each selected student  - View round
announcements and selected students

5\. APPLICATIONS MODULE (applications/)  - Apply to drive (checks if
already applied, already placed)  - View my applications  - Withdraw
application

6\. ATTENDANCE MODULE (attendance/)  - Create session (with target
departments + semesters)  - Open/close attendance windows (time-limited)
 - Self-mark with GPS verification: \* Gets student\'s lat/lng \*
Compares with hall\'s bounding box (min_lat, max_lat, min_lng, max_lng)
\* If outside → rejected  - Manual mark/unmark by faculty  - Live
attendance count  - Export to Excel (styled .xlsx with present/absent
sheets)  - Auto-close expired sessions (background job every 5 min)

7\. FORMS MODULE (forms/)  - Create form with multiple question types:
\* short_text, long_text, multiple_choice, checkbox, date, linear_scale
 - Target specific institutes, departments, semesters  - Submit
responses (ON CONFLICT prevents duplicates)  - View individual and
aggregate responses  - Close forms to stop submissions  - Email +
notification sent on form creation

8\. ANNOUNCEMENTS MODULE (announcements/)  - Create announcement
(department-scoped)  - Email blast to targeted students  - View and
delete announcements

9\. NOTIFICATIONS MODULE (notifications/)  - Get user\'s notifications
 - Mark individual notification as read  - Mark all as read  - Internal
functions: createNotification(), createBulkNotifications() (Called by
other modules, not exposed as API)

10\. CALENDAR MODULE (calendar/)  - Create/view/delete placement events
 - Accessible by SC, FC, TPC

11\. LEAVES MODULE (leaves/)  - Backend routes exist but functionality
is scaffold (TODO)  - Mobile app service is fully built and ready

12\. PLACEMENT MODULE (placement/)  - View all drives with applicant
counts  - View drive applicants  - Update application status (applied →
placed)  - Update drive status  - View all placed students

SC MODULE (sc/)  - Assign student as SC  - Remove SC role  - View SC
list  - View department students

================================================================================
5. END-TO-END FLOWS (USE THESE IN INTERVIEW)
================================================================================

FLOW 1: USER SIGNUP AND LOGIN
\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-- 1. Student
opens app → SplashScreen → no token → LoginScreen 2. Taps \"Sign Up\" →
SignupScreen 3. Enters email (must be \@charusat.edu.in) 4. App calls
POST /api/auth/send-signup-otp 5. Backend checks email not already
registered, generates 6-digit OTP, emails it 6. Student enters OTP → App
calls POST /api/auth/signup with all details + OTP + deviceId 7. Backend
verifies OTP, hashes password with Argon2id, inserts user + student
profile 8. Backend signs JWT token, creates active session in DB 9. App
saves token + user in SharedPreferences 10. App routes to
StudentHomeScreen based on role

FLOW 2: STUDENT APPLIES TO A DRIVE
\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-- 1.
StudentHomeScreen loads → calls GET /api/companies/drives/student 2.
Backend runs eligibility query:  - Joins drives with student profile  -
Compares student\'s CGPA, marks, backlogs against drive criteria  -
Returns only eligible drives (marked as is_eligible: true) 3. Student
sees drive card with \"Eligible\" badge 4. Taps \"Apply\" → confirmation
dialog appears (warns about commitment) 5. Taps \"Yes, Apply\" → App
calls POST /api/applications with {drive_id} 6. Backend checks: already
applied? already placed? drive still active? 7. If OK → inserts
application, returns success 8. App shows \"Applied ✅\" snackbar,
refreshes both drives and applications list 9. Drive card now shows
\"Applied\" badge

FLOW 3: GEO-FENCED ATTENDANCE MARKING
\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-- 1.
Faculty creates session: POST /api/attendance/sessions  - Specifies:
title, hall (with lat/lng bounds), target departments/semesters  - Email
blast sent to targeted students 2. Faculty opens window: POST
/api/attendance/sessions/:id/window/open  - Sets duration (e.g., 10
minutes) 3. Student opens attendance tab → sees active session with
\"Mark Present\" button 4. Taps \"Mark Present\" → app requests GPS
permission 5. Geolocator gets current position (high accuracy, 10-sec
timeout) 6. App sends POST /api/attendance/sessions/:id/self-mark with
{lat, lng} 7. Backend checks:  - Is session active? Is window open? Is
window expired?  - Is student in target department/semester?  - Is
student\'s lat/lng INSIDE the hall\'s bounding box? (lat \>= min_lat AND
lat \<= max_lat AND lng \>= min_lng AND lng \<= max_lng) 8. If
everything passes → INSERT INTO attendance (ON CONFLICT DO NOTHING) 9.
Student sees \"Marked present ✅\" 10. Faculty sees live count update
(SC screen polls every 15 seconds)

================================================================================
6. SECURITY IMPLEMENTATION (VERY IMPORTANT FOR INTERVIEW)
================================================================================

AUTHENTICATION: \-\-\-\-\-\-\-\-\-\-\-\-\-\-\-- - Passwords hashed with
Argon2id (NOT bcrypt) WHY Argon2id? It\'s a HYBRID:  - Argon2i =
resistant to side-channel attacks  - Argon2d = resistant to GPU cracking
 - Argon2id = combines BOTH protections Config: 19MB memory, 2
iterations, 1 parallelism (OWASP minimum)

\- JWT tokens with HS256 algorithm  - Algorithm explicitly LOCKED to
HS256 during sign AND verify  - This prevents the \"none algorithm
attack\" (a real vulnerability where attacker sends unsigned token with
algorithm: \"none\")  - Issuer claim set to \"charusat-placement-setu\"
--- verified on every request  - 7-day expiry

\- OTP system: 6-digit code, 10-minute expiry, stored in-memory map Used
for: email verification during signup, password reset

AUTHORIZATION: \-\-\-\-\-\-\-\-\-\-\-\-\-\-- - 5-tier RBAC (Role-Based
Access Control) Roles: student, student_coordinator, fc, tpc, faculty

\- Two middleware functions compose together: protect() → Verifies JWT,
loads user from DB restrictTo(\'tpc\', \'fc\') → Checks if user role is
in allowed list

\- Department scoping: FC and SC can only see data from their own
department This is enforced at the SQL level, not just the UI

\- Account deactivation: is_active flag checked in middleware Blocked at
API level, not just login

DEVICE SECURITY: \-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-- - Device binding for
students: \* First login: device_id saved to user record \* Next login
from SAME device: allowed \* Next login from DIFFERENT device: error
\"DEVICE_CHANGED\" \* Admin can reset binding (POST
/api/auth/reset-device/:userId)

WHY? Prevents credential sharing among students (common in universities)

\- Single active session: \* On new login: DELETE all previous sessions,
INSERT new one \* Only ONE device can be logged in at a time

API SECURITY: \-\-\-\-\-\-\-\-\-\-\-\-\-- - Rate limiting: \* General
API: 200 requests per 15 minutes \* Auth endpoints (login, OTP): 10
requests per 15 minutes (stricter) \* Different limits for dev vs
production

\- Helmet: sets 15+ security headers automatically \*
Content-Security-Policy, X-Frame-Options, Strict-Transport-Security,
etc.

\- HPP: prevents HTTP Parameter Pollution \* Attacker can\'t send
duplicate query params to confuse the server

\- JSON body limit: 50KB max \* Prevents payload-based DoS attacks

\- Error masking: in production, internal errors return generic
\"Internal server error\" \* Prevents leaking stack traces or DB schema
to attackers

INPUT VALIDATION: \-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-- -
express-validator middleware for auth routes: \* Email: isEmail() +
normalizeEmail() \* Password: isLength({min: 8}) + trim() \* Name:
notEmpty() + trim() + escape() (XSS prevention) \* Role:
isIn(\[\'student\', \'tpc\', \'fc\', \'sc\'\])

\- Email domain lock: \* Students must use \@charusat.edu.in \* Staff
must use \@charusat.ac.in \* Cross-checks domain against selected role

\- All SQL queries use parameterized syntax (\$1, \$2, \$3\...) \* ZERO
string concatenation in SQL \* Makes SQL injection IMPOSSIBLE

WEAK AREAS (mention in interview to show awareness):
\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-- -
OTP stored in-memory (should use Redis for production/scalability) - No
refresh token rotation (single long-lived JWT) - No CSRF protection (OK
for mobile API, but needed if web client added) - No request
logging/audit trail for admin actions - Email passwords in .env (could
use cloud secrets manager)

================================================================================
7. FILE-BY-FILE SUMMARY
================================================================================

BACKEND FILES: \-\-\-\-\-\-\-\-\-\-\-\-\--

server.js → Main entry point. Sets up Express, security middleware,
registers all routes. → Also starts background job to auto-close expired
attendance sessions.

config/db.js → Creates PostgreSQL connection pool with 10 max
connections. → Exports pool for all modules to use.

middlewares/auth_middleware.js → protect(): Verifies JWT token, loads
user from DB, checks is_active and device_id. → restrictTo(): Checks
user role against allowed roles.

middlewares/rateLimiter.js → apiLimiter: 200 req/15min for all routes. →
authLimiter: 10 req/15min for auth routes (login, OTP). → Different
limits for dev vs production.

middlewares/roleGuard.js → Another role-checking middleware (similar to
restrictTo in auth_middleware).

middlewares/validators/authValidator.js → Validates signup (email,
password, name, role) and login (email, password). → Uses
express-validator with sanitization.

utils/jwt_helper.js → signToken(): Creates JWT with HS256, issuer, 7-day
expiry. → verifyToken(): Verifies JWT with algorithm lock (only HS256
allowed).

utils/email_helper.js → Pooled SMTP transport (5 connections, rate
limited 5/sec). → sendOTPEmail(): Professional HTML email template for
OTP. → sendBulkEmail(): Sends to all matching students in parallel. →
getTargetEmails(): Queries students by institute + department +
semester. → driveEmailTemplate(): HTML template for new drive
notifications. → announcementEmailTemplate(): HTML template for
announcements. → formEmailTemplate(): HTML template for new form
notifications.

utils/otp_store.js → In-memory Map to store OTPs with 10-min expiry. →
generateOTP(): Random 6-digit code. → saveOTP(): Stores email → OTP
mapping with expiry timer. → verifyOTP(): Checks code matches and not
expired.

modules/auth/auth_routes.js → 7 routes: send-signup-otp, signup, login,
logout, me, send-reset-otp, reset-password. → Public routes use
authLimiter. Protected routes use protect().

modules/auth/auth_controller.js → Thin layer: extracts data from
req.body, calls service, sends response. → Validates email domain
(@charusat.edu.in vs \@charusat.ac.in).

modules/auth/auth_service.js → Core auth logic: \* sendSignupOTP: Check
email not taken, generate OTP, send email. \* signup: Verify OTP, hash
password (Argon2id), create user + student profile, sign JWT. \* login:
Verify password, check device binding, sign JWT, manage sessions. \*
resetDeviceBinding: Clear device_id + delete sessions. \* sendResetOTP /
resetPassword: OTP-based password reset.

modules/companies/company_routes.js → Company and drive CRUD. → Smart
eligibility queries with CGPA normalization. → Application management
(apply, status update, placed tracking).

modules/companies/drive_round_routes.js → Drive round management with
database transactions (BEGIN/COMMIT/ROLLBACK). → Sends bulk email +
in-app notifications to selected students. → Uses email_helper and
notification_controller internally.

modules/attendance/attendance_routes.js → 700+ lines. Most complex
module. → Session CRUD, window management, geo-fenced self-marking. →
Manual marking, live count, history, Excel export. → Hall management and
department/semester filtering.

modules/forms/form_routes.js → Dynamic form builder with 6 question
types. → Targeted distribution (institute, department, semester). → ON
CONFLICT for idempotent submissions.

modules/announcements/announcement_routes.js → Create/list/delete
announcements. → Department-scoped (SC sees own department, TPC sees
all).

modules/notifications/notification_controller.js → Fetch, mark read,
bulk create. → Internal createBulkNotifications() used by other modules.

modules/calendar/calendar_routes.js → Simple CRUD for placement events.

modules/students/student_routes.js → Profile management, student
listing, SC role assignment.

modules/sc/sc_routes.js → SC assignment and department student
management.

modules/applications/application_routes.js → Apply, view mine, withdraw.

modules/placement/placement_routes.js → All drives view, applicant
management, status updates.

modules/leaves/leave_routes.js → Scaffold --- routes exist but logic is
TODO.

MOBILE APP FILES: \-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--

main.dart → App entry point. Sets up MaterialApp with theme. → Initial
route: SplashScreen.

core/constants/api_constants.dart → Single source of truth for ALL 80+
API endpoint URLs. → Uses static methods for dynamic URLs (e.g.,
unapply(driveId)). → Base URL points to ngrok tunnel (for development).

core/routes/app_routes.dart → 15 named route constants organized by role
(auth, student, tpc, fc, sc, shared).

data/models/user_model.dart → User data class with role helpers
(isStudent, isTpc, isFc, isStudentCoordinator). → Placement eligibility
check: semester \>= 6. → Display helpers: initials, roleLabel. →
fromJson/toJson for serialization. → copyWith for immutable updates.

data/models/company_model.dart → Company/drive data with eligibility
fields. → Includes isEligible flag and isDeadlinePassed check.

data/services/auth_service.dart → Login/signup: sends HTTP request,
saves token + user to SharedPreferences. → refreshUser(): calls GET /me,
updates local cache. → Auto-detects SC role: if student + is_sc flag →
changes role to student_coordinator. → Handles rate limit (429)
responses. → logout(): clears token + user from SharedPreferences.

data/services/company_service.dart → getEligibleCompanies(),
getMyApplications(), applyToCompany(), unapplyFromDrive().

data/services/export_service.dart → Client-side Excel export for form
responses. → Platform-aware: Android → /Download/, iOS → Documents
directory. → Creates styled headers (black background, white text). →
Opens file after creation using open_file package.

data/services/leave_service.dart → Full CRUD: requestLeave, getMyLeaves,
getPendingLeaves, getAllLeaves, updateLeaveStatus. → Ready for when
backend completes leave module.

presentation/screens/auth/splash_screen.dart → Checks for saved session,
redirects to login or home.

presentation/screens/auth/login_screen.dart → Email + password login.
Gets deviceId using device_info_plus.

presentation/screens/auth/signup_screen.dart → Multi-field signup with
OTP verification.

presentation/screens/home/student_home_screen.dart → 1147 lines. Main
student dashboard. → 4 tabs: Home (drives + applications), Attendance,
Forms, Profile. → Drive pagination (show 2, load 3 more). →
Apply/unapply with confirmation dialogs. → Hero stats card (Drives,
Applied, Eligible, Placed counts).

presentation/screens/home/sc_home_screen.dart → 2166 lines. Most complex
screen in the app. → 5 tabs: Home, Drives, Seminars, Students, Forms. →
Local data models (ScDriveModel, ScApplicantModel, etc.). → Live seminar
polling every 15 seconds. → GPS self-marking with full permission
handling. → Create sessions, manage windows, view attendance. →
Department student list with search.

presentation/screens/home/fc_home_screen.dart → FC menu: SC Management,
Placement Drives, Forms, Announcements, Attendance, Leaves. → Navigation
to feature-specific screens.

presentation/screens/attendance/student/student_attendance_screen.dart →
View attendance sessions and mark present.

presentation/screens/attendance/faculty/faculty_attendance_screen.dart →
Create sessions, open/close windows, manual marking.

presentation/screens/forms/form_builder_screen.dart → Dynamic form
creator supporting 6 question types.

presentation/screens/forms/form_fill_screen.dart → Form filling
interface for students.

presentation/screens/forms/form_responses_screen.dart → View responses +
export to Excel.

presentation/screens/placement/create_drive_screen.dart → Create new
drives with all eligibility criteria.

features/student/profile/student_profile_screen.dart → View and edit
student profile.

features/sc/student_detail_screen.dart → View detailed student info (for
SC management).

features/fc/screens/sc_management_screen.dart → Assign/remove Student
Coordinators.

================================================================================
8. KEY POINTS TO SPEAK IN INTERVIEW
================================================================================

When asked \"Tell me about your project\":
\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--
\"I built a full-stack placement management platform for CHARUSAT
University. It has a Node.js + Express backend with PostgreSQL and a
Flutter mobile app. The system serves 5 different user roles with
role-specific dashboards. Key features include GPS-verified attendance,
smart drive eligibility matching, dynamic form builder, and a
comprehensive security implementation with Argon2id password hashing,
JWT authentication, and rate limiting.\"

When asked about ARCHITECTURE:
\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-- \"The
backend follows a modular MVC pattern. Each feature like auth,
attendance, and companies is an isolated module with its own routes,
controller, and service. The Flutter app uses clean architecture with
core, data, and presentation layers. All API calls go through service
classes, and data is modeled in separate Dart classes.\"

When asked about SECURITY:
\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-- \"I used Argon2id
instead of bcrypt because it\'s a hybrid that resists both side-channel
and GPU attacks. JWT tokens are locked to HS256 algorithm to prevent the
\'none algorithm attack\'. I implemented device binding to prevent
credential sharing --- each student can only log in from one registered
device. Rate limiting is applied globally (200/15min) and strictly on
auth routes (10/15min). All SQL queries use parameterized syntax, making
SQL injection impossible.\"

When asked about CHALLENGES:
\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-- \"The biggest
challenge was implementing GPS-based attendance with geo-fencing. I had
to handle varying GPS accuracy, permission flows on both Android and
iOS, and define bounding boxes for each hall. I also had to handle edge
cases like permission denied forever, location services disabled, and
GPS timeout. Another challenge was building the smart eligibility engine
that normalizes CGPA between 4.0 and 10.0 scales and matches against
multiple criteria simultaneously.\"

When asked about DATABASE DESIGN:
\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-- \"I
used PostgreSQL for its relational capabilities and array support. For
multi-select targeting (departments, semesters), I used native
PostgreSQL arrays instead of junction tables --- this simplifies queries
for this use case. I used ON CONFLICT DO NOTHING for idempotent
operations like attendance marking, preventing duplicate records without
application-level checks. All queries use connection pooling (max 10)
and parameterized syntax.\"

When asked about STATE MANAGEMENT:
\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--
\"I used Provider for global auth state and setState() for local screen
state. User session data (JWT token and user profile) is persisted in
SharedPreferences. On app startup, the token is checked and the user is
routed to their role-specific home screen. If the token is expired (401
response), auto-logout triggers.\"

When asked \"What would you improve?\":
\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--
\"First, I\'d replace the in-memory OTP store with Redis for horizontal
scalability. Second, I\'d add refresh token rotation with short-lived
access tokens. Third, I\'d add push notifications via FCM for real-time
alerts. Fourth, I\'d add Docker containerization for easier deployment.
Fifth, I\'d add comprehensive audit logging for admin actions.\"

When asked about SCALABILITY:
\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-- \"The app
is ready for scale in several ways: - Connection pooling (10 max
connections) prevents DB exhaustion - Pooled SMTP transport (5
connections) handles bulk emails efficiently - Rate limiting protects
against abuse - Modular architecture means new features can be added as
separate modules What I\'d add for production scale: Redis for
OTP/session storage, WebSocket for real-time updates instead of polling,
and database connection limits per module.\"

When asked about TESTING approach:
\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--
\"The architecture supports testability --- services are separate from
controllers, so business logic can be unit tested independently. The
clean architecture on Flutter side means services can be mocked for
widget testing. I\'d add: - Jest for backend unit tests - Supertest for
API integration tests - Flutter widget tests with mock services\"

================================================================================
END OF INTERVIEW PREP
================================================================================
