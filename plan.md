# Student Early-Warning & Guidance System — Implementation Plan

## Scope and product approach

Build a complete, database-backed academic support platform rather than a static dashboard. The delivered product will cover the requested end-to-end workflow:

`marks + attendance + assignments → transparent risk calculation → early-warning alert → teacher review → intervention → follow-up → improvement tracking`

The first working version will use a transparent rule-based risk engine with configurable thresholds. It will not require an external AI/ML service. The data model and service boundaries will leave room for a future predictive model without hiding the reasons behind a student's risk status.

The application will use the explicitly requested stack: React + Vite + TypeScript, Tailwind CSS, React Router, Axios, Recharts, Lucide React, Node.js + Express REST APIs, JWT sessions, bcrypt password hashing, Zod validation, Nodemailer when configured, and MySQL only. The managed MySQL resource provided by the project will be the source of truth for all dashboards, charts, alerts, interventions, and demo data.

## Architecture

- **Frontend:** Vite React TypeScript app in `client/`, with route guards and role-aware layouts for `/admin/*`, `/teacher/*`, and `/student/*`.
- **Backend:** Express TypeScript API in `server/`, separated into routes, controllers, services, middleware, validators, database access, and utilities.
- **Database:** MySQL schema and migrations in `database/`, accessed with `mysql2/promise`. Foreign keys, unique constraints, indexes, and timestamps will be used throughout.
- **Authentication:** Email or ID plus password login. Passwords are hashed with bcrypt. Access tokens are JWTs and are stored in an `httpOnly` cookie named `sweg_session`; the API also accepts a Bearer token for API clients. Cookies use `SameSite=None; Secure` for public HTTPS Preview compatibility, while local HTTP keeps secure handling separate. Token expiry returns a clean 401 and the frontend logs out automatically.
- **Authorization:** A middleware validates the JWT, loads the user, and enforces role permissions. Teacher queries always scope to `teacher_student_assignments`; students can only access their own student record. Admin-only mutations write audit logs.
- **Risk engine:** A service computes attendance, academic decline, assignment completion, and failed-assessment points from actual records. It persists each assessment, generates human-readable reasons and recommended actions, and creates de-duplicated alerts for assigned mentors when a risk state requires attention.
- **Notifications:** In-app notifications are always persisted. Nodemailer email is attempted only when SMTP environment variables are present; email failure never blocks risk calculation or the core application.

## Data model

The normalized MySQL schema will include:

- `users`, `students`, `teachers`, `departments`, `classes`, `subjects`
- `teacher_student_assignments`, `teacher_subjects`
- `marks`, `attendance`, `assignments`
- `risk_assessments`, `alerts`, `interventions`, `notifications`, `audit_logs`
- `academic_semesters` and `warning_thresholds` to support the requested semester and configurable threshold administration

All academic write operations record the actor and timestamp. Foreign keys use safe delete behavior, and list/detail queries have indexes for student, teacher, subject, risk, status, dates, and assignment relationships.

## Key API surface

- `POST /api/auth/login`, `POST /api/auth/logout`, `GET /api/auth/me`
- `GET /api/admin/dashboard`, `GET/POST/PUT /api/admin/departments|classes|subjects|teachers|students|thresholds`
- `GET /api/admin/analytics`, `GET /api/admin/alerts`, `GET /api/admin/audit-logs`
- `GET /api/teacher/dashboard`, `GET /api/teacher/students`, `GET /api/teacher/students/:id`
- `GET/POST/PUT /api/students/:id/marks`, `/attendance`, `/assignments`
- `GET /api/students/:id/risk`, `POST /api/students/:id/recalculate-risk`
- `GET /api/alerts`, `PUT /api/alerts/:id/acknowledge`
- `GET/POST/PUT /api/interventions`, `GET /api/interventions/:id`
- `GET /api/student/dashboard`, `GET /api/student/profile`, `PUT /api/student/profile`
- `GET /api/notifications`, `PUT /api/notifications/:id/read`

Responses omit password hashes and internal credentials. Zod schemas validate all meaningful request bodies and query filters. Errors are converted to safe, user-facing messages.

## Frontend routes and screens

- `/login` — role-neutral login form with demo account hints and friendly error states.
- `/admin/dashboard` — institution KPIs, risk distribution, attendance/marks trends, department performance, alert volume, and quick links.
- `/admin/students`, `/admin/teachers`, `/admin/departments`, `/admin/classes`, `/admin/subjects`, `/admin/thresholds`, `/admin/alerts`, `/admin/audit` — management workflows backed by REST mutations.
- `/teacher/dashboard` — assigned-student KPI cards, alert/intervention summaries, searchable/filterable table, and follow-ups.
- `/teacher/students/:id` — academic profile, risk reasons, marks/attendance/assignment charts, alert acknowledgement, intervention creation/editing, and before/after improvement comparison.
- `/student/dashboard` — personal metrics, current risk status, transparent reasons, teacher guidance, trend charts, intervention history, and upcoming follow-up.
- A shared not-found, loading, empty, error, toast, modal, and confirmation system will be used across roles.

## Design direction

### Design movement
**Editorial data-dashboard / calm institutional modernism:** a serious academic operations tool with a warm, human support tone rather than a punitive compliance dashboard.

### Core principles
1. **Clarity before density:** surface the handful of metrics that change decisions, then provide drill-down context.
2. **Supportive, not punitive:** use "Needs Support", "Early Warning", and "Requires Attention" instead of labeling students as failures.
3. **Evidence is visible:** every risk state includes a score, reasons, source metrics, and a recommended next action.
4. **Operational momentum:** interventions and follow-ups are treated as first-class work items, not buried in an activity log.

### Color philosophy
Use a deep ink/navy foundation for trust and legibility, a distinctive indigo-violet accent for action and focus, and restrained semantic colors: sage green for healthy/improving, amber for attention, and coral red for high-risk states. Red is reserved for urgency indicators and never for student identity or negative moral judgment. Light surfaces use a soft blue-gray tint instead of stark white to make long dashboard sessions comfortable.

### Layout paradigm
A persistent left rail and slim contextual top bar frame a wide, asymmetrical workspace. The main content uses a large narrative panel for the primary action or student story, with smaller evidence cards and a right-side alert/follow-up rail. Tables become horizontal scroll regions or stacked cards on mobile rather than shrinking into unreadable columns.

### Signature elements
- **Signal rail:** slim vertical risk/status accent on student rows and profile panels.
- **Evidence chips:** compact metric pills such as `Attendance 64%`, `Marks ↓ 21%`, and `2 missing` that make reasons scannable.
- **Support path:** intervention timeline with connected nodes for alert, guidance, follow-up, and outcome.

### Interaction philosophy
Every mutation gives immediate, specific feedback and updates the relevant data from the API. Risk explanations are expandable but never hidden. Filters preserve context, and destructive or permission-sensitive operations require a clear confirmation dialog. Empty states always explain the next useful action.

### Animation
Use restrained transitions only: 150–200ms opacity/translate transitions for panels, skeleton shimmer for loading, and a subtle chart reveal on first render. No decorative continuous animation and no motion that changes meaning.

### Typography system
Use Inter for interface text and Fraunces for high-level narrative headings where available through a durable web font import; fall back to system sans/serif stacks. Use compact uppercase labels for metadata, 14–16px body copy, and a strong 28–36px heading scale for dashboard narratives.

### Brand essence
**A clear, compassionate signal system that helps educators act before a setback becomes a pattern.** Personality: observant, grounded, encouraging.

### Brand voice
Headlines are direct and calm; CTAs are action-oriented but humane; microcopy explains what the system knows and what to do next.

Example lines:
- “See the students who need a clearer next step.”
- “The signal is early. Your support can be, too.”

### Wordmark and logo concept
A simple wordmark paired with a three-line ascending signal mark: three rounded bars that form a subtle open book/arrow. It represents data becoming guidance without suggesting surveillance.

### Signature brand color
**Signal Indigo `#5B5CE2`** — an ownable, optimistic focus color used for primary actions, active navigation, and the support-path timeline.

## Demo data and accounts

The seed script will generate:

- 1 admin, 5 teachers, 50 students
- Multiple departments, classes, semesters, and subjects
- Healthy, improving, declining, high-attendance/low-marks, low-attendance/good-marks, and combined-risk student profiles
- Marks, attendance, assignments, risk assessments, alerts, notifications, and interventions

Demo passwords will be safe, documented development credentials only, for example `DemoPass!2026`, and will never be used as production secrets.

## Implementation phases

1. Replace the empty workspace with the React/Vite/Express TypeScript toolchain, Tailwind setup, env example, route manifest, and health endpoint.
2. Add the MySQL schema, migration runner, seed script, and database service.
3. Implement authentication, JWT cookie/Bearer parsing, bcrypt login, role guards, and frontend auth context.
4. Implement shared application shell and polished responsive design system.
5. Implement teacher dashboard and student detail vertical slice, including real CRUD for marks, attendance, assignments, risk recalculation, alerts, and interventions.
6. Implement student dashboard and self-service limited profile update.
7. Implement admin management screens, threshold configuration, analytics, and audit views.
8. Add email notification integration, loading/error/empty states, route synchronization, README, and deployment configuration.
9. Run typecheck, build, API/database smoke checks, inspect the finished code with a read-only requirement review, fix confirmed issues, commit to the managed canonical repository, and verify Preview readiness.

## Material constraints and assumptions

- MySQL is the only database; no MongoDB or Oracle will be introduced.
- External paid services are not required for the core application. Email is optional and disabled gracefully when SMTP is absent.
- The app will use the user-requested credential/JWT model rather than Manus OAuth because the specification explicitly requires email/ID + password, bcrypt, and JWT. The implementation will still preserve the project’s Preview cookie compatibility requirements.
- The managed project database is shared across development and published deployment; migrations and seeding will be additive and idempotent, and no destructive reset will be run.
- The first release will prioritize the end-to-end academic support workflow and the requested role dashboards over nonessential institution features.
