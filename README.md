# BugTrail

## Overview

BugTrail is a modern engineering defect management platform built from the ground up on Next.js 16, Prisma ORM, Supabase PostgreSQL, and Google Gemini AI. The goal was not to recreate Bugzilla's interface, but to understand the underlying problem of structured defect lifecycle management and build a contemporary, intelligent solution with features that go meaningfully beyond what traditional trackers offer.

The result is a platform that combines formal QA workflow enforcement, cryptographic audit integrity, AI-assisted triage, a GitHub webhook integration, real-time event streaming, and a complete multi-tenant workspace model — all in a clean, modern interface.

---

## Problem Understanding & Core Functionality

Defect tracking is fundamentally about structured accountability: every issue must have a clear lifecycle, a responsible owner, and a traceable history. BugTrail enforces this through a formal state machine modeled on Bugzilla's lifecycle, while adding modern tooling around it.

- **Workflow State Machine**: Strict state transitions — UNCONFIRMED → NEW → ASSIGNED → RESOLVED → VERIFIED → CLOSED — enforced server-side on every transition request, including validation that disallowed jumps are rejected with the list of valid destinations.
- **Resolution Enforcement**: Moving a bug to RESOLVED requires one of six formal codes — FIXED, INVALID, WONTFIX, DUPLICATE, WORKSFORME, INCOMPLETE. The resolution is stored and displayed as part of the permanent record.
- **Product & Component Hierarchy**: Defects are scoped under Products and their child Components, mirroring real engineering team architecture.
- **Multi-Tenant Workspaces**: Each team gets an isolated workspace with a shareable join code. Users select their role during onboarding. Defects, products, and members are fully scoped to the workspace.
- **Markdown Comments**: Comment bodies render full Markdown, supporting code blocks, links, and formatting for technical discussions.

---

## Innovation & Meaningful Differentiation

### Cryptographic SHA-256 Audit Chain

Every mutation — status change, field update, comment, assignment, flag — writes a block to the audit log. Each block stores the previous block's hash alongside a SHA-256 digest computed from the actor ID, action, old value, new value, timestamp, and previous hash. This forms a linked chain starting from a genesis root hash.

A dedicated verification endpoint (`GET /api/bugs/[id]/audit-verify`) re-computes every hash in the chain and reports the first index where a mismatch is detected, identifying whether the chain link or the content was tampered. The result is displayed as a live integrity badge on every bug detail page.

### Google Gemini AI Auto-Triage

When filing a defect, users can request AI-assisted triage. The title and description are sent to Gemini 2.5 Flash, which returns a severity (BLOCKER through TRIVIAL), priority (P1–P5), and a short rationale. The response is parsed and pre-fills the form fields instantly. A keyword-based heuristic fallback handles the case where no API key is configured, detecting terms like "crash", "security", "data loss", or "typo" to suggest appropriate classifications.

### Duplicate Detection Before Submission

While a user types a new bug title in the filing modal, a debounced request runs against the live bug database. Existing bugs are scored by word-overlap with the new title. Any match above a 40% threshold appears as a warning panel below the form, listing the matching bug key, title, and similarity percentage — preventing duplicate reports before they are submitted.

### Automated Whining Engine

Teams define configurable alert rules specifying which statuses and severity levels to monitor and an inactivity threshold in days. Running the engine scans all open defects against every active rule and produces a grouped digest — rule by rule — listing stale defect keys, titles, current status, and days since last update. Rules can be toggled active or inactive individually. The engine is exposed as a cron-compatible API endpoint (`GET /api/cron/whining`) and is also triggerable manually from the Whining dashboard.

### GitHub Webhook Integration

A webhook endpoint (`POST /api/integrations/github`) listens for `push` and `pull_request` events from GitHub. Commit messages and PR titles are scanned for Bugzilla-style references — `Fixes BUG-42`, `Closes BUG-7`, `Ref BUG-15`. On a match, the integration automatically posts the commit SHA or PR link as a comment on the corresponding defect. If the keyword is `FIXES`, `RESOLVES`, or `CLOSES`, the defect is also automatically transitioned to RESOLVED (FIXED) and an audit log entry is written. This creates a live bridge between the code repository and the defect tracker.

### Server-Side Role-Based Access Control

Five roles are enforced at the API layer: ADMIN, TRIAGER, DEVELOPER, QA/TESTER, and REPORTER. Transition endpoints return HTTP 403 if the requesting user's session role lacks the required permission — regardless of UI state. The permission matrix is explicit: only ADMIN can create products or manage the team; REPORTER cannot trigger any state transitions.

### Real-Time SSE Event Pipeline

A persistent Server-Sent Events stream (`GET /api/events`) broadcasts every significant mutation — status transitions, new comments, attachments, flag changes — to all connected clients. The defect list and Kanban board subscribe to this stream and refresh their data live, without page reloads.

### Review Flag Matrix

Defects support formal code review and QA flags (`review+`, `review-`, `feedback`) modeled on Bugzilla's flag system. Each flag records the setter, an optional requestee, and writes to the audit chain. The flag matrix is rendered as a dedicated panel on the bug detail page.

### Saved Filter Queries

Users can save any combination of active filters — status, severity, priority, product, assignee, search term — as a named query. Saved queries are stored per user and can be restored instantly from the filter bar dropdown, eliminating the need to re-apply complex filter combinations repeatedly.

### Global Command Palette (⌘K)

Pressing `⌘K` or `Ctrl+K` opens a floating search overlay. Users can search defects by key or title with live results, navigate to any view, or switch active user persona — all from the keyboard without touching the mouse.

---

## Technical Architecture

| Layer | Technology | Responsibility |
| :--- | :--- | :--- |
| Frontend | Next.js 16 App Router + React 19 + Tailwind CSS v4 | UI, Kanban board, command palette, SSE subscription |
| API Layer | Next.js Route Handlers | REST endpoints, RBAC enforcement, SSE streaming, webhook receiver |
| ORM & Database | Prisma 5 + Supabase PostgreSQL | Relational data model, workspace isolation, connection pooling |
| AI Integration | Google Gemini 2.5 Flash (`@google/genai`) | Auto-triage classification, heuristic fallback |
| Audit System | SHA-256 Hash Chain (`crypto` module) | Tamper-evident block-linked mutation log |
| Authentication | bcryptjs + HTTP-only Cookie Sessions | Secure passwords, 7-day session tokens, per-workspace role |

---

## How to Run Locally

**Requirements**: Node.js v18+

```bash
git clone https://github.com/Chaitanya-G41/BugTrail.git
cd BugTrail
npm install
```

Create a `.env` file:
```env
DATABASE_URL="postgresql://..."
DIRECT_URL="postgresql://..."
GEMINI_API_KEY="your-api-key"
```

```bash
npx prisma db push
npm run dev
```

Open [http://localhost:3000](http://localhost:3000). Demo accounts are provisioned automatically on first login.

---

## Demo Accounts

| Email | Role | Capabilities |
| :--- | :--- | :--- |
| alice.admin@bugtrail.org | ADMIN | Full access — products, team, all transitions |
| bob.triager@bugtrail.org | TRIAGER | Severity, priority, assignment, transitions |
| chaitanya.dev@bugtrail.org | DEVELOPER | Fix bugs, resolve with code, upload patches |
| community.reporter@external.io | REPORTER | File bugs and comment only |

Password for all demo accounts: `password123`

---

## Acknowledgments

- Bugzilla project for establishing the formal defect lifecycle model that BugTrail interprets.
- Next.js and Vercel for the App Router and edge runtime architecture.
- Prisma and Supabase for the ORM and PostgreSQL infrastructure.
- Google AI for the Gemini API powering AI triage.