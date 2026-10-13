# Student Early-Warning & Guidance System — Delivery Outcomes

## 1. Full-stack foundation and project runtime
- The repository contains a separated React/Vite frontend and modular Express backend using TypeScript, Tailwind CSS, React Router, Axios, Recharts, and Lucide React.
- The project listens on the configured runtime port, serves the frontend, exposes an unauthenticated `/api/health` endpoint, and serves a synchronized `/manus-routes.json` route manifest.
- Secrets and API configuration are read from environment variables; `.env.example` contains placeholders and no real credentials.
- The project includes a production build, Dockerfile, README.md, and clear development scripts.

## 2. MySQL database and seed data
- MySQL is the only database and the schema is normalized with foreign keys, indexes, constraints, and timestamps.
- Tables cover users, students, teachers, departments, classes, subjects, teacher/student and teacher/subject assignments, marks, attendance, assignments, risk assessments, alerts, interventions, notifications, audit logs, academic semesters, and configurable warning thresholds.
- Migrations are additive and idempotent; the seed script is safe to run without destroying later user edits.
- Seed data includes 1 admin, 5 teachers, 50 students, multiple departments/classes/subjects, marks, attendance, assignments, risk assessments, alerts, and interventions, including healthy, improving, declining, high-attendance/low-marks, low-attendance/good-marks, and combined-risk profiles.
- Demo accounts are documented with safe development-only passwords.

## 3. Authentication and authorization
- Users can log in using email or ID plus password; passwords are bcrypt-hashed and never returned by APIs.
- JWT authentication supports protected routes, token expiry handling, logout, secure application cookies, and Bearer-token API clients.
- Role-based middleware protects `/admin/*`, `/teacher/*`, and `/student/*` application areas and API operations.
- Teachers can access only students assigned to them; students can access only their own records; students cannot modify marks, attendance, risk scores, teacher feedback, or access admin/teacher APIs.
- Admin-level academic mutations create audit-log records and all user-facing unauthorized/errors are safe and friendly.

## 4. Admin management and analytics
- Admin can manage students, teachers, departments, classes, subjects, teacher/mentor assignments, subject assignments, academic semesters, warning thresholds, account status, and users.
- Admin can view institution-wide totals, at-risk/high-risk counts, open alerts, completed interventions, average attendance/marks, risk distribution, attendance and marks trends, department performance, and alerts by month.
- Admin can view all alerts and audit logs without changing academic data silently.
- Admin tables support search, filters, loading states, empty states, validation, confirmation dialogs, and real backend CRUD.

## 5. Teacher dashboard and student monitoring
- Teacher dashboard shows assigned-student totals, students needing attention, high/medium/low risk counts, new alerts, pending interventions, and upcoming follow-ups from the API/database.
- Assigned-student table supports search by student name and ID plus department, class, semester, risk, attendance range, and performance-trend filters.
- Teacher can open a detailed authorized student profile showing marks, attendance, assignments, trends, risk score, generated reasons, recommended action, alert history, and intervention history.
- Teacher can create/update marks, attendance, and assignment performance for authorized students only.
- Teacher can acknowledge alerts, add guidance/intervention notes, set action plans and follow-up dates, change intervention statuses, and record outcomes: Improved, Partially Improved, No Improvement, or Requires Further Support.

## 6. Transparent early-warning engine and alerts
- Risk calculation uses actual attendance, academic decline versus previous average, assignment completion, and failed assessments; it is not based only on a single marks cutoff.
- The engine calculates `risk_score = attendance_risk + academic_risk + assignment_risk + assessment_risk`, persists the assessment, and maps the result to configurable LOW/MEDIUM/HIGH thresholds.
- Every medium/high assessment shows the score, human-readable reasons grounded in actual data, and recommended actions.
- When risk reaches MEDIUM/HIGH, increases, or has a significant new decline, the system creates a teacher alert and in-app notification; duplicate alerts are prevented with a configurable cooldown.
- Nodemailer email notification is attempted only when configured and never prevents the core application from working when SMTP is absent.

## 7. Student self-service dashboard
- Student dashboard shows the student name/ID, department, semester, attendance percentage, average marks, assignment completion, current risk status, trend, warning messages, teacher guidance, and upcoming follow-up.
- Student charts cover marks over time, attendance over time, subject-wise performance, and assignment completion using API data.
- Student can view intervention history and before/after improvement progress but cannot alter protected academic or teacher-authored data.
- Student can update only limited profile information with validated backend input.

## 8. Intervention and improvement tracking
- The intervention workflow stores type, reason, teacher notes, action plan, follow-up date, status, creator, and timestamps.
- After follow-up, the system compares before/after attendance and average marks and shows improvement status and a comparison chart/timeline.
- The UI makes the support path visible from alert through guidance, intervention, follow-up, and outcome.

## 9. Responsive professional UI and resilience
- The interface uses a responsive sidebar/top navigation layout, cards, tables, charts, modals, toast notifications, accessible forms, consistent typography/spacing, and clear loading/error/empty states.
- The design uses supportive language such as Needs Support, Early Warning, and Requires Attention; green means healthy/improving, amber means attention, red means high risk, and blue/neutral means normal information.
- Desktop, laptop, tablet, and mobile layouts remain usable; dense tables transform to scrollable or stacked layouts at narrow widths.
- No important button is fake: all actions either call a working API or are omitted.

## 10. Documentation, validation, and delivery
- README.md documents the problem, objectives, features, technology stack, architecture, database, environment variables, setup, migration/seed commands, startup, demo accounts, API overview, screenshots section, and future ML enhancements.
- Type checking, production build, backend health check, route-manifest check, and targeted database/API smoke checks pass.
- A read-only requirements review traces the main frontend-to-backend-to-database paths and confirmed issues are fixed before delivery.
- The intended files are committed to the managed canonical repository and the final Preview URL is verified after the server listens.
