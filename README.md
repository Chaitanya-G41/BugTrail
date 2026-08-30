# BugTrail — Modern Engineering Defect Tracking & Workflow Platform

BugTrail is an enterprise-grade engineering defect tracking system that provides full Bugzilla workflow state machine parity, multi-tenant workspace isolation, server-side role-based access control, cryptographic audit logging, and automated QA triage integrations.

---

## 1. Executive Overview

Legacy issue tracking tools like Bugzilla offer robust workflow rules but suffer from outdated user interfaces, fragmented access control, and slow manual triage. BugTrail modernizes defect management by combining strict QA state machines with contemporary collaboration tools, cryptographic audit chains, and automated AI assistance.

Key System Capabilities:
- Full Bugzilla workflow state machine parity with formal state transitions and resolution codes.
- Multi-tenant workspace onboarding with unique join codes and team role selection.
- Server-side Role-Based Access Control (RBAC) enforcing permission boundaries across five user roles.
- SHA-256 cryptographic audit trail generating tamper-evident hash chains for all defect mutations.
- Dual datalist comboboxes allowing users to pick existing products/components or type custom names on the fly.
- Native operating system file picker for uploading diagnostic logs and code patches (.patch / .diff).
- Automated stale defect whining engine with custom alert rules and digest generation.
- Interactive drag-and-drop Kanban workflow board with real-time state machine validation.
- Google Gemini AI auto-triage recommendations and real-time duplicate defect detection.
- Real-time Server-Sent Events (SSE) pipeline for live cross-user browser updates.

---

## 2. Authentication, Workspaces & Persona Management

BugTrail supports both traditional user registration and fast evaluation persona switching:

Real Authentication Engine:
- Secure password hashing using bcryptjs.
- Persistent session management via HTTP-only session cookies.
- Account signup, login, and logout API endpoints.

Quick Evaluation Demo Accounts:
To facilitate fast testing by judges and team members, the sign-in page provides one-click credentials for preset team roles:
- Alice Vance (Lead Architect): ADMIN role with full administrative power.
- Bob Martinez (Bug Triager): TRIAGER role managing defect severities, priorities, and assignments.
- Chaitanya (Core Developer): DEVELOPER role executing bug fixes, resolution state changes, and patch uploads.
- Eva Lin (Frontend Specialist): DEVELOPER role focusing on frontend component defects.
- Community Reporter: REPORTER role for external defect submitters.

Multi-Tenant Workspace Onboarding:
- Create Workspace: Generates a unique shareable join code (such as BT-8X9K2L) and sets the creator as ADMIN.
- Join Workspace: Allows new users to enter a join code and select their team role (DEVELOPER, TRIAGER, QA/TESTER, REPORTER). To preserve security, joining users cannot claim the ADMIN role.
- Workspace Switcher: Enables users belonging to multiple teams to switch between active workspaces cleanly from the navigation header.

---

## 3. Server-Side Role-Based Access Control (RBAC)

BugTrail enforces strict role permissions both in the user interface and on the server:

Roles and Permissions Breakdown:
- ADMIN: Full system access, including product creation, team member management, and workflow overrides.
- TRIAGER: Manages defect severities, priorities, developer assignments, whining rules, and state transitions.
- DEVELOPER: Self-assigns defects, uploads code patches (.patch / .diff), and transitions bugs to RESOLVED with Bugzilla resolution codes.
- QA/TESTER: Files defect reports, tests resolved bugs, transitions RESOLVED defects to VERIFIED or CLOSED, or REOPENS failed fixes.
- REPORTER: Files new defects, posts comments, and watches issue updates. State transitions and bug assignments are locked for this role.

Server-Side Permission Verification:
All workflow state transition endpoints inspect the active session token using the server-side hasPermission validator. If an unauthorized user (such as a REPORTER) attempts a state transition via direct API request, the server rejects the action with an HTTP 403 Forbidden response.

---

## 4. Bugzilla Workflow Engine & Resolution Parity

BugTrail implements the standard Bugzilla state machine to enforce formal QA lifecycle discipline:

Allowed Workflow Transitions:
- UNCONFIRMED -> NEW or RESOLVED
- NEW -> ASSIGNED, RESOLVED, or UNCONFIRMED
- ASSIGNED -> RESOLVED or NEW
- RESOLVED -> VERIFIED, CLOSED, or REOPENED
- VERIFIED -> CLOSED or REOPENED
- CLOSED -> REOPENED
- REOPENED -> ASSIGNED, RESOLVED, or NEW

Bugzilla Resolution Codes:
When a defect is moved to the RESOLVED state, the user must specify a resolution code:
- FIXED: Problem has been identified and corrected.
- INVALID: Problem described is not a bug.
- WONTFIX: Problem will not be fixed.
- DUPLICATE: Problem is a duplicate of an existing bug.
- WORKSFORME: All attempts to reproduce the problem failed.
- INCOMPLETE: Bug report has insufficient description or reproduction steps.

Product and Component Structure:
Defects are categorized hierarchically under Products and architectural Components. Custom products and components can be added dynamically during defect creation.

---

## 5. Drag-and-Drop Kanban Workflow Board

The Kanban board provides a visual representation of defect pipelines across six status columns: UNCONFIRMED, NEW, ASSIGNED, RESOLVED, VERIFIED, and CLOSED.

Kanban Board Features:
- Card Drag-and-Drop: Move defects between status columns with mouse drag.
- Real-Time Validation: Dropping a card onto an invalid status column triggers an alert displaying allowed state transitions according to Bugzilla workflow rules.
- Resolution Prompt: Dragging a bug into the RESOLVED column opens a prompt requesting the Bugzilla resolution code (FIXED, INVALID, WONTFIX, etc.).
- Role Protection: Users signed in with the REPORTER role are prevented from dragging cards.

---

## 6. SHA-256 Cryptographic Audit Chain

To guarantee data integrity and prevent unauthorized back-door database modifications, BugTrail logs every defect change in a cryptographic audit chain.

Cryptographic Audit Architecture:
- Each audit log entry includes the actor ID, action type, field changed, previous value, new value, timestamp, and a SHA-256 hash digest.
- Every new audit block includes the hash of the preceding block (prevHash), forming an immutable hash chain.
- The defect detail page features a SHA-256 Audit Verification badge that verifies the integrity of all blocks in the log. If any record is tampered with directly in the database, the hash verification fails and displays an alert banner.

---

## 7. Automated Whining Engine for Stale Defects

BugTrail includes a customizable automated whining engine to prevent defects from sitting idle in triage pipelines.

Whining Engine Capabilities:
- Custom Whining Rules: Define alert criteria based on status, inactivity threshold in days, and severity levels.
- Automated Digest Generation: Scans active defects against configured rules and compiles stale defect digests listing affected bug keys, titles, assignees, and last update timestamps.
- Manual and Scheduled Execution: Whining digests can be triggered manually from the UI or run via scheduled API endpoint execution.

---

## 8. Flexible Datalist Comboboxes & Native File Uploads

Filing Defect Reports:
The File Bug modal utilizes native datalist combobox fields for Product, Component, Severity, and Priority inputs:
- Users can click the input field to select an existing option from the dropdown list.
- Users can type custom text directly into the same input field.
- If a custom Product or Component name is entered, the backend creates the new record on the fly and registers it for future dropdown selections.

Native File Attachments:
- The attachment tab uses the native operating system file selection window to allow users to attach diagnostic logs, crash reports, and code patches.
- Files ending in .patch or .diff are automatically flagged with a PATCH badge and provided with direct preview and download links.

---

## 9. Google Gemini AI Triage & Real-Time Duplicate Detection

AI Auto-Triage:
Using Google Gemini API (@google/genai), BugTrail analyzes bug titles and descriptions to suggest recommended Severity and Priority ratings along with a short rationale.

Real-Time Duplicate Scanning:
While a user types a bug title and description in the File Bug modal, an automated debounced search checks existing defects for semantic similarities. If potential duplicates are detected, a warning banner displays matching bug keys, titles, and match percentages before submission.

Heuristic Fallback:
If no GEMINI_API_KEY environment variable is configured, the system falls back gracefully to a heuristic triage calculator, ensuring the application functions without API dependencies.

---

## 10. Database Architecture & Production Deployment

Database Support:
BugTrail uses Prisma ORM configured for PostgreSQL databases (such as Supabase or Neon) in production environments and SQLite for local development.

Production Deployment Features:
- Non-Interactive Schema Pushing: Build scripts execute prisma db push --accept-data-loss and prisma generate to apply database schema updates automatically during Vercel deployment.
- Auto-Seeding Engine: On fresh or empty database deployments, the login API automatically initializes demo accounts and workspace structures on the first request.
- Cross-Machine Join Code Resolution: When a team member enters a join code created on another machine, the join API auto-provisions the workspace record locally to ensure seamless joining.

---

## 11. Local Setup Instructions

Prerequisites:
- Node.js version 18 or higher
- npm package manager

Installation Steps:
1. Clone the repository:
   git clone https://github.com/Chaitanya-G41/BugTrail.git
   cd BugTrail

2. Install dependencies:
   npm install

3. Configure environment variables in a .env file:
   DATABASE_URL="postgresql://postgres.your-ref:YOUR_PASSWORD@aws-0-ap-southeast-1.pooler.supabase.com:6543/postgres?pgbouncer=true"
   DIRECT_URL="postgresql://postgres.your-ref:YOUR_PASSWORD@db.your-ref.supabase.co:5432/postgres"
   GEMINI_API_KEY="your-optional-api-key"

4. Push the Prisma database schema:
   npx prisma db push

5. Start the development server:
   npm run dev

6. Access the application at http://localhost:3000

---
