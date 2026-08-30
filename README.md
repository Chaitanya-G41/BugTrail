# BugTrail

## Overview

BugTrail is a modern, enterprise-grade engineering defect tracking and workflow management platform that brings the full power of Bugzilla-style quality assurance pipelines into a clean, contemporary interface. Built on Next.js 16 App Router with Prisma ORM, Supabase PostgreSQL, and Google Gemini AI, BugTrail goes beyond simple issue tracking by introducing cryptographic audit integrity, automated AI-assisted triage, formal Bugzilla workflow enforcement, multi-tenant workspace isolation, and a fully server-side enforced role-based permission system.

---

## Problem Understanding & Core Functionality

Software teams need structured, disciplined defect pipelines with clear ownership, strict state transitions, and reliable audit trails. BugTrail addresses this with:

- Full Bugzilla Workflow State Machine: Formal transitions across UNCONFIRMED, NEW, ASSIGNED, RESOLVED, VERIFIED, and CLOSED states with strict allowed-transition rules.
- Bugzilla Resolution Codes: On resolving a defect, developers must specify FIXED, INVALID, WONTFIX, DUPLICATE, WORKSFORME, or INCOMPLETE. Resolution is stored and displayed as part of the defect record.
- Product and Component Hierarchy: Defects are scoped under Products and architectural Components, mirroring real engineering team structures.
- Dual-Mode Datalist Comboboxes: Every product, component, severity, and priority field accepts both selection from existing options and free-text entry in the same input field. New values typed during bug filing are automatically registered and available for future use.
- Saved Query Filters: Users can save complex defect filter combinations (status, severity, priority, product, assignee) and restore them instantly.
- CC List and Notifications: Users can add themselves to the CC list of any defect to receive in-app notifications on updates.
- Review Flag Matrix: Formal code review and QA approval flags (review+, review-, feedback) per defect, tracking sign-off chains.
- Native OS File Attachments: Upload actual files from the local file system including diagnostic logs, crash dumps, and code patches. Files ending in .patch or .diff are automatically flagged and styled distinctly.

---

## Innovation & Meaningful Differentiation

### SHA-256 Cryptographic Audit Chain

Every state transition, field update, comment, and assignment produces an immutable audit log block. Each block stores:
- The actor ID and action type
- The previous and new field values
- A SHA-256 hash computed over the previous block's hash, the actor, the timestamp, and the new value

This forms a tamper-evident cryptographic chain. Any modification to a stored audit record breaks the hash continuity, which is instantly detected and displayed as an integrity failure badge on the defect detail page. This goes beyond standard activity logs by making the entire mutation history cryptographically verifiable.

### Automated Whining Engine

BugTrail includes a configurable stale defect alert system inspired by Bugzilla's whining daemon. Teams can define custom rules specifying which status values, severity levels, and inactivity thresholds trigger a digest report. When the engine runs, it scans all open defects against active rules and compiles a structured digest grouping stale defects by rule, listing bug keys, titles, current status, assignees, and days since last activity.

### Google Gemini AI Auto-Triage

When filing a defect, users can request AI-assisted triage by providing a title and description. The system sends the content to Google Gemini 2.0 Flash, which returns a recommended severity level, priority rating, and a concise rationale explaining the classification. The response is parsed and pre-populated into the form fields, reducing the manual effort of categorizing incoming defects. The system includes a built-in heuristic fallback that estimates severity and priority locally when AI is unavailable, ensuring the feature remains functional in all environments.

### Real-Time Duplicate Defect Detection

As a user types a new bug title and description, BugTrail runs a debounced background similarity check against the existing defect database. If semantically similar defects are found, a warning panel appears before submission listing matching bug keys, titles, and similarity scores. This prevents duplicate defect creation without requiring manual search.

### Server-Side Role-Based Access Control

BugTrail enforces a five-tier permission model at the API level, not just in the UI:

- ADMIN: Full access to all features including product creation, team member management, and workflow overrides across all defects.
- TRIAGER: Controls defect severity, priority, and assignment. Can drive state transitions across the full workflow.
- DEVELOPER: Executes fixes, transitions defects to RESOLVED with a resolution code, and uploads code patch files.
- QA/TESTER: Verifies fixed defects, transitions RESOLVED defects to VERIFIED or CLOSED, and reopens defects that fail verification.
- REPORTER: Files new defects and posts comments. Workflow state transitions and bug assignment are server-side restricted for this role.

All transition endpoints inspect the active session token and return HTTP 403 Forbidden if the requesting user's role lacks the required permission, regardless of what the UI displays.

### Multi-Tenant Workspace Isolation

Each team operates in its own isolated workspace. Creating a workspace generates a unique shareable join code. Users who join via the code select their role during onboarding. Defects, products, and team members are fully scoped to their workspace. Users belonging to multiple workspaces can switch between them from the navigation header, with each workspace maintaining its own role assignment for that user.

### Drag-and-Drop Kanban Board with State Machine Enforcement

The Kanban board renders six columns corresponding to Bugzilla workflow states. Defects can be dragged across columns, but the state machine rules are enforced on drop. If a drag targets an invalid destination state, the system rejects the transition with a message listing the allowed destination states. Moving a card to RESOLVED opens a resolution code prompt before the transition is committed. REPORTER-role users are blocked from initiating drags entirely.

### Server-Sent Events Real-Time Pipeline

A persistent SSE connection is established per browser session. When any defect is modified, a new comment is posted, a status is transitioned, or an attachment is uploaded, the event is broadcast over the SSE stream to all connected clients. Defect list pages and the Kanban board automatically refresh their data on receiving relevant events without requiring manual page reloads.

### Global Command Palette

Pressing Cmd+K (or Ctrl+K on Windows) opens an instant search palette. Users can search defects by key or title, filter by product or status, and navigate to any defect detail page without using the mouse. The palette also exposes quick actions like opening the File Bug modal.

---

## Technical Architecture

| Layer | Technology | Responsibility |
| :--- | :--- | :--- |
| Frontend | Next.js 16 + React 19 + Tailwind CSS v4 | UI, Kanban board, command palette, real-time SSE subscription |
| API Engine | Next.js App Router Route Handlers | REST endpoints, RBAC enforcement, SSE streaming |
| ORM & Database | Prisma 5.22.0 + Supabase PostgreSQL | Relational modeling, workspace isolation, connection pooling |
| AI Integration | Google Gemini 2.0 Flash (@google/genai) | Severity/priority classification, duplicate detection |
| Audit Layer | SHA-256 Hash Chaining | Tamper-evident block-based mutation logging |
| Authentication | bcryptjs + HTTP-only Cookie Sessions | Secure password hashing, persistent session management |

---

## How to Run Locally

Requirements:
- Node.js v18 or higher
- npm

Steps:

1. Clone the repository:
   ```bash
   git clone https://github.com/Chaitanya-G41/BugTrail.git
   cd BugTrail
   ```

2. Install dependencies:
   ```bash
   npm install
   ```

3. Create a `.env` file:
   ```env
   DATABASE_URL="postgresql://postgres.your-ref:YOUR_PASSWORD@aws-0-ap-southeast-1.pooler.supabase.com:6543/postgres?pgbouncer=true"
   DIRECT_URL="postgresql://postgres.your-ref:YOUR_PASSWORD@db.your-ref.supabase.co:5432/postgres"
   GEMINI_API_KEY="your-api-key"
   ```

4. Push the database schema:
   ```bash
   npx prisma db push
   ```

5. Start the development server:
   ```bash
   npm run dev
   ```

Open [http://localhost:3000](http://localhost:3000) in your browser.

---

## Acknowledgments

- Bugzilla project for establishing the formal defect state machine standards that BugTrail builds upon.
- Next.js and Vercel for the App Router architecture and Turbopack build tooling.
- Prisma and Supabase for the ORM and PostgreSQL infrastructure.
- Google AI team for the Gemini API that powers AI triage.