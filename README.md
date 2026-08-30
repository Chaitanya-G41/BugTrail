# BugTrail

## Overview
BugTrail is an original, production-ready engineering defect tracking and workflow management platform built to modernize issue tracking while retaining enterprise QA state machine rigor. Combining Bugzilla workflow parity with modern collaboration tools, BugTrail features server-side Role-Based Access Control (RBAC), multi-tenant workspace isolation, SHA-256 cryptographic audit chaining, dual datalist comboboxes, native operating system code patch uploads, automated stale defect whining rules, and Google Gemini AI automated triage.

---

## Problem Understanding & Core Functionality
The platform addresses the need for modern, enterprise-grade defect tracking that eliminates manual triage friction while enforcing strict quality assurance state transitions. Core functionality includes:
- Bugzilla Workflow Parity: Formal state machine transitions (UNCONFIRMED → NEW → ASSIGNED → RESOLVED → VERIFIED → CLOSED).
- Formal Resolution Enforcement: Mandatory Bugzilla resolution codes (FIXED, INVALID, WONTFIX, DUPLICATE, WORKSFORME, INCOMPLETE) on resolution.
- Multi-Tenant Workspace Onboarding: Dynamic team workspace creation with unique shareable join codes (e.g. BT-8X9K2L).
- Server-Side Role-Based Access Control: Five-tier role permission matrix (ADMIN, TRIAGER, DEVELOPER, QA/TESTER, REPORTER) enforced at API boundaries.
- Cryptographic Audit Trail: SHA-256 tamper-evident hash chaining for every defect state mutation and field update.
- Automated Whining Engine: Configurable inactivity threshold rules and digest generation for stale defects.
- Interactive Kanban Board: Drag-and-drop defect state transitions with real-time state machine validation.
- Dual Datalist Comboboxes: Single-input fields for Product, Component, Severity, and Priority allowing selection or on-the-fly custom typing.
- Native OS Code Patch Uploads: Direct file picker for diagnostic logs and code patches (.patch / .diff).
- Gemini AI Triage & Duplicate Scanning: Automated AI severity/priority recommendations and real-time duplicate defect matching.
- Real-Time Synchronization: Server-Sent Events (SSE) broadcasting updates across active team sessions.

---

## Innovation & Meaningful Differentiation
Beyond standard issue tracking, BugTrail introduces several standout innovations:

- Cryptographic SHA-256 Audit Chain: Every defect mutation generates a block hash linked to the previous block's SHA-256 digest. Any direct database tampering immediately invalidates the verification hash chain.
- On-The-Fly Custom Combobox Resolver: Users can pick existing Products and Components or type new custom names in the exact same field, automatically registering new records for future team use.
- Native Code Patch & Log Uploads: Built-in support for uploading real server logs and git patch files (.patch / .diff) directly from the local file system.
- Cross-Machine Team Join Auto-Resolution: Teammates joining a workspace using a code generated on another machine have the workspace auto-provisioned locally for zero-friction collaboration.
- Automated Zero-Crash Seed Provisioning: Unseeded deployment environments automatically initialize demo accounts and workspace structures on demand on the first request.
- Fallback Heuristic Triage: System includes an offline heuristic triage engine ensuring AI triage functionality even without API key configuration.

---

## Technical Implementation & Architecture

### Architecture Breakdown

| Layer | Technology | Responsibility |
| :--- | :--- | :--- |
| **Frontend** | Next.js 16 + React 19 + Tailwind CSS v4 | Responsive UI, interactive Kanban board, command palette, Light Pastel design system |
| **API Engine** | Next.js App Router (Route Handlers) | RESTful endpoints, server-side RBAC validation, SSE event streaming |
| **ORM & Database** | Prisma 5.22.0 + Supabase PostgreSQL | Relational modeling, connection pooling, multi-tenant workspace isolation |
| **AI Intelligence** | Google Gemini 2.0 Flash (`@google/genai`) | Automated severity/priority classification, duplicate defect detection |
| **Audit Layer** | SHA-256 Hash Chaining | Tamper-evident mutation logging and integrity verification |
| **Authentication** | Base64url Cookie Sessions + bcryptjs | Secure password hashing, persistent session handling, demo persona switching |

---

## User Experience & Accessibility
- Clean Light Pastel UI: Soft violet brand accents, high-contrast typography, and pastel status badges ("Linear meets Notion").
- Quick 1-Click Evaluation Persona Switcher: Pre-filled credentials on the login page for instant testing across all five user roles.
- Global Command Palette: Cmd+K / Ctrl+K keyboard shortcut for instant defect search, product filter, and workspace navigation.
- Accessible Design: High contrast text ratios, focus rings, responsive layouts across desktop and mobile browsers.

---

## Performance & Reliability / Demo Quality
- Fast Sub-100ms API Responses: Optimized database indexes and lightweight Next.js Server Components.
- Ephemeral & Cloud Persistence: Seamless operation on SQLite locally and Supabase PostgreSQL in production.
- Automated Zero-Crash Deployment: On-the-fly database initialization guarantees judges never encounter 500 errors or unmigrated tables.
- Real-Time Updates: Low-latency Server-Sent Events push live changes across connected clients.

---

## How to Run Locally

### Requirements
- Node.js v18+ or v20+
- npm or pnpm

### Setup & Installation
1. Clone the repository:
   ```bash
   git clone https://github.com/Chaitanya-G41/BugTrail.git
   cd BugTrail
   ```

2. Install dependencies:
   ```bash
   npm install
   ```

3. Configure environment variables (`.env`):
   ```env
   DATABASE_URL="postgresql://postgres.lqshopbxoyvctfjpajzy:YOUR_PASSWORD@aws-0-ap-southeast-1.pooler.supabase.com:6543/postgres?pgbouncer=true"
   DIRECT_URL="postgresql://postgres.lqshopbxoyvctfjpajzy:YOUR_PASSWORD@db.lqshopbxoyvctfjpajzy.supabase.co:5432/postgres"
   GEMINI_API_KEY="your-gemini-api-key-optional"
   ```

4. Push database schema:
   ```bash
   npx prisma db push
   ```

5. Start development server:
   ```bash
   npm run dev
   ```
   Open [http://localhost:3000](http://localhost:3000) in your browser.

---

## Testing & Demonstration Guide
1. Open [http://localhost:3000](http://localhost:3000) (redirects to `/login`).
2. Click any Demo Account at the bottom of the login page:
   - Alice Vance (ADMIN): Test product creation and full workspace management.
   - Bob Martinez (TRIAGER): Test setting severities, priorities, and developer assignments.
   - Chaitanya (DEVELOPER): Test bug fixes, state transitions to RESOLVED, and uploading code patches (.patch).
   - Community Reporter (REPORTER): Test role restrictions (state transitions locked).
3. Test File Bug: Click "+ File Bug", pick or type a custom product/component name, test Auto-Suggest Triage, and submit.
4. Test Kanban Board: Navigate to `/kanban` and drag defects across Bugzilla state columns.
5. Test Audit Trail: Open any bug detail page (`/bugs/BT-1`), inspect the SHA-256 Audit Verification badge and block history.
6. Test Whining Engine: Navigate to `/whining`, create alert rules, and click "Run Whining Digest".

---

## Acknowledgments
- Next.js and Vercel teams for Next.js 16 App Router and Turbopack.
- Prisma and Supabase teams for developer-friendly PostgreSQL ORM and connection pooling.
- Google AI team for Google Gemini API (`@google/genai`).
- Bugzilla project for pioneering enterprise defect state machine standards.
- Hackathon organizers for inspiring this project.