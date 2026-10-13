# Signalpath · Student Early-Warning & Guidance System

Signalpath is a full-stack academic support platform that helps educators notice performance changes early, understand why they matter, and follow through with guidance. It analyzes marks, attendance, assignment performance, and academic trends to create transparent early-warning alerts for assigned mentors.

## Honest readiness status

**Current status: operational prototype / pilot candidate — not approved for production student data yet.** The core dashboards, authentication, database-backed records, explainable risk engine, alerts, interventions, and seeded demo environment work end to end. Before a real institution launch, complete the launch gates below: versioned migrations, roster and subject/assignment administration, transactional writes, production SMTP/outbox, automated integration tests, backups/restore testing, audit views, pagination, and institutional privacy/retention review.

The application now refuses weak JWT configuration in production, restricts credentialed CORS to configured origins, checks request origins on cookie-authenticated mutations, and rejects invalid academic score ranges. Demo credentials remain available only as development seed data and must never be used for a real deployment.

## Problem statement

Academic decline is often visible across several small signals before it appears as a final result. When marks, attendance, and assignment completion are reviewed in separate places, it is difficult for teachers to identify students who need a timely conversation. Signalpath brings those signals together and connects them to an intervention and follow-up workflow.

## Objectives

- Detect multi-factor academic risk using transparent rules rather than a single marks cutoff.
- Give teachers an actionable queue of assigned students and explain every warning.
- Keep student access private and read-only for protected academic data.
- Store guidance, follow-up dates, outcomes, and before/after improvement evidence.
- Provide institution-level analytics and configurable risk thresholds for administrators.

## Features

- JWT + bcrypt email/ID/password login with ADMIN, TEACHER, and STUDENT roles.
- MySQL-backed normalized schema with foreign keys, indexes, timestamps, notifications, and audit logs.
- Admin dashboard with institution KPIs, risk distribution, trends, department performance, alerts, and thresholds.
- Teacher dashboard with assigned-student search/filtering, risk sorting, alerts, interventions, and follow-ups.
- Student dashboard with personal metrics, marks trend, risk reasons, teacher guidance, and improvement history.
- Rule-based risk engine using attendance, academic decline, missing assignments, and failed assessments.
- Automatic in-app alerts with cooldown-based duplicate suppression.
- Responsive UI for desktop, tablet, and mobile.

## Technology stack

| Layer | Technology |
| --- | --- |
| Frontend | React, Vite, TypeScript, React Router, Axios, Recharts, Lucide React |
| Styling | Tailwind-compatible project setup plus a focused responsive CSS design system |
| Backend | Node.js, Express, TypeScript, REST API |
| Security | JWT, bcryptjs, httpOnly session cookie, Zod validation, restricted CORS, origin checks |
| Database | MySQL / mysql2 |
| Email | SMTP integration is a launch requirement; the current prototype stores in-app notifications only |

## Architecture

The React client is in `client/` and talks to the Express REST API under `/api`. The server is split into middleware, validators, services, and database access. The risk engine in `server/services/risk.ts` reads source records, persists a new assessment, and creates mentor alerts and notifications when a student reaches MEDIUM or HIGH risk.

The central workflow is:

`academic records → risk assessment → reasons + recommended action → mentor alert → intervention → follow-up → improvement tracking`

## Launch gates for real institutional use

1. **Identity and access:** configure a strong `JWT_SECRET` of at least 32 characters, set explicit `CLIENT_ORIGIN`/`PREVIEW_ORIGIN`, remove demo credentials from the production login experience, add password reset/MFA/credential rotation, and define admin provisioning.
2. **Roster operations:** build or connect an audited workflow for user, department, class, subject, semester, teacher assignment, and student enrollment CRUD. The current admin directory is read-only.
3. **Academic operations:** add teacher-facing marks, attendance, and assignment entry/import screens; verify teacher-subject and student-course authorization, not only teacher-student assignment.
4. **Risk governance:** use comparable semester or rolling windows, ignore assignments that are not yet due, represent insufficient data as “not assessed”, document threshold ownership, and have educators validate scoring before operational use.
5. **Consistency:** use database transactions/idempotency for academic writes plus a durable notification outbox; a risk-write failure must not cause a user retry to duplicate a record.
6. **Migrations:** replace startup schema replay with ordered versioned migrations and a recorded schema version. Use a least-privilege runtime database account and a separate migration account.
7. **Testing:** add unit tests for risk/validators and MySQL integration tests for authentication, role scoping, record writes, alert lifecycle, and migration upgrades. Run them in CI.
8. **Operations:** configure encrypted backups/PITR, retention and restore drills; add structured redacted logs, request IDs, metrics, alerting, and a schema-aware readiness check.
9. **Privacy:** define student-data retention, correction, export, deletion, access review, and case-note visibility. Return only fields needed by each role.
10. **Workflow completion:** expose acknowledge/resolve alerts, notification read state, intervention outcome updates, before/after comparisons, and actual caseload export in the UI.

## Database structure

`database/schema.sql` creates `users`, `students`, `teachers`, `departments`, `classes`, `subjects`, `teacher_student_assignments`, `teacher_subjects`, `marks`, `attendance`, `assignments`, `risk_assessments`, `alerts`, `interventions`, `notifications`, `audit_logs`, `academic_semesters`, and `warning_thresholds`.

## Installation

Requirements: Node.js 22+, pnpm 10+, and a MySQL-compatible database.

```bash
pnpm install
cp .env.example .env
pnpm db:migrate
pnpm seed # development/demo only
pnpm dev
```

Use `pnpm seed` only in a disposable development database. Never run it against production.

## Netlify deployment

This repository includes `netlify.toml` for deploying the Vite frontend. Netlify must use `pnpm run build` and publish `dist/client`; do not use a pnpm workspace filter because this is a single-package repository. The Express API and MySQL database are not hosted by Netlify's static frontend deployment. For a complete full-stack deployment, host the server with the included Dockerfile or another Node-capable service, then set the frontend API base URL and configure CORS accordingly.

## Environment variables

- `PORT` — server port, default `3000`.
- `DATABASE_URL` — MySQL URL.
- `JWT_SECRET` — long random secret; required and at least 32 characters in production.
- `JWT_EXPIRES_IN` — JWT expiry, default `8h`.
- `CLIENT_ORIGIN` — explicit allowed frontend origin.
- `PREVIEW_ORIGIN` — optional managed Preview origin when cross-site iframe cookies are required.
- `SMTP_HOST`, `SMTP_PORT`, `SMTP_USER`, `SMTP_PASS`, `SMTP_FROM` — future/launch SMTP settings; delivery is not yet implemented.

## Demo data and accounts

The seed creates 1 admin, 5 teachers, 50 students, multiple departments/classes/subjects, records for healthy/improving/declining and mixed-signal students, risk assessments, alerts, and interventions.

| Role | Login | Password |
| --- | --- | --- |
| Admin | `admin@example.com` | `DemoPass!2026` |
| Teacher | `teacher1@example.com` | `DemoPass!2026` |
| Student | `student1@example.com` | `DemoPass!2026` |

These accounts are development-only and must not be used for a real institution.

## API overview

- `POST /api/auth/login`, `POST /api/auth/logout`, `GET /api/auth/me`
- `GET /api/admin/dashboard`, `GET/PUT /api/admin/thresholds`, `GET /api/admin/students`
- `GET /api/teacher/dashboard`, `GET /api/teacher/students/:id`
- `POST /api/students/:id/marks`, `/attendance`, `/assignments`
- `GET /api/students/:id/risk`, `POST /api/students/:id/recalculate-risk`
- `GET /api/alerts`, `PUT /api/alerts/:id/acknowledge`
- `GET/POST/PUT /api/interventions`
- `GET /api/student/dashboard`
- `GET /api/notifications`, `PUT /api/notifications/:id/read`

## Validation commands

```bash
pnpm check
pnpm build
curl http://localhost:3000/api/health
curl http://localhost:3000/manus-routes.json
```

## Future enhancements

A future model can estimate decline probability, attendance risk, or intervention priority, while the existing transparent rule reasons remain visible as a human-readable explanation. Other extensions could include scheduled email digests, student self-reflection check-ins, bulk CSV import with audit trails, and richer department-level cohort comparisons.
