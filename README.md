# Focus — Personal Task Manager

A full-stack Personal Task Manager built as a pnpm monorepo with a Node.js/Express REST API backend, PostgreSQL persistence, and a React + Vite frontend with a corporate SaaS design.

---

## Live Demo

> Deployed link here

---

## Tech Stack

| Layer | Tool | Why |
|---|---|---|
| Runtime | Node.js 24 | LTS with native ESM and `node:test` support |
| Package manager | pnpm workspaces | Fast installs, strict isolation, workspace protocol |
| Language | TypeScript 5.9 | End-to-end type safety across all packages |
| Backend framework | Express 5 | Minimal, battle-tested HTTP framework |
| Database | PostgreSQL | Relational, reliable persistence with full SQL power |
| ORM | Drizzle ORM | Type-safe schema-as-code, zero-overhead queries |
| Validation | Zod (v4) + drizzle-zod | Single source of truth for schema + runtime validation |
| API contract | OpenAPI 3.1 + Orval | Code-generated hooks and Zod schemas from the spec |
| Frontend | React 19 + Vite 7 | Fast HMR, modern React features |
| UI components | shadcn/ui + Radix | Accessible, unstyled primitives with Tailwind styling |
| Styling | Tailwind CSS v4 | Utility-first, design token–driven |
| Data fetching | TanStack Query v5 | Server state management with cache invalidation |
| Logging | Pino + pino-http | Structured JSON logging for production |
| Testing | node:test + supertest | Built-in Node test runner, zero extra dependencies |

---

## How to Run Locally

**Prerequisites:** Node.js 20+, pnpm 10+, a PostgreSQL database.

```bash
# 1. Clone and install
git clone <repo-url>
cd task-manager
pnpm install

# 2. Set the database URL
#    Create a .env file or export directly:
export DATABASE_URL="postgresql://user:password@localhost:5432/tasks"

# 3. Push the database schema
pnpm --filter @workspace/db run push

# 4. Start the API server (port 5000)
pnpm --filter @workspace/api-server run dev

# 5. Start the frontend (separate terminal)
pnpm --filter @workspace/task-manager run dev

# 6. Open http://localhost:5173
```

**Run the test suite:**
```bash
pnpm --filter @workspace/api-server run test
```

**Regenerate API client from OpenAPI spec:**
```bash
pnpm --filter @workspace/api-spec run codegen
```

---

## API Documentation

Base URL: `/api`

### Tasks

| Method | Path | Body | Response |
|---|---|---|---|
| `GET` | `/tasks` | — | `200 Task[]` — sorted by `sortOrder` asc |
| `GET` | `/tasks?status=active` | — | `200 Task[]` — incomplete tasks only |
| `GET` | `/tasks?status=completed` | — | `200 Task[]` — completed tasks only |
| `GET` | `/tasks?search=<q>` | — | `200 Task[]` — title contains query (case-insensitive) |
| `GET` | `/tasks?priority=high` | — | `200 Task[]` — filtered by priority level |
| `POST` | `/tasks` | `{ title*, description?, dueDate?, priority? }` | `201 Task` |
| `GET` | `/tasks/stats` | — | `200 { total, active, completed, overdue, byPriority }` |
| `GET` | `/tasks/export` | — | `200 text/csv` — downloads tasks as CSV |
| `PATCH` | `/tasks/reorder` | `{ order: [{ id, sortOrder }] }` | `200 Task[]` |
| `PATCH` | `/tasks/bulk/complete` | — | `200 { count }` — marks all active tasks complete |
| `DELETE` | `/tasks/bulk/completed` | — | `200 { count }` — deletes all completed tasks |
| `GET` | `/tasks/:id` | — | `200 Task` or `404` |
| `PATCH` | `/tasks/:id` | `{ title?, description?, dueDate?, completed?, priority? }` | `200 Task` or `404` |
| `DELETE` | `/tasks/:id` | — | `204` or `404` |
| `PATCH` | `/tasks/:id/toggle` | — | `200 Task` — toggles `completed` flag |

### Health

| Method | Path | Response |
|---|---|---|
| `GET` | `/healthz` | `200 { status: "ok" }` |

### Error responses

All errors return `{ error: string }` with an appropriate HTTP status code:
- `400` — validation failure (e.g. missing required `title`)
- `404` — task not found
- `500` — unexpected server error

---

## Task Object Shape

```ts
{
  id:          number;          // auto-increment integer primary key
  title:       string;          // required, min 1 character
  description: string | null;   // optional notes
  dueDate:     string | null;   // ISO date string (YYYY-MM-DD)
  completed:   boolean;         // defaults to false
  priority:    "none" | "low" | "medium" | "high";  // defaults to "none"
  sortOrder:   number;          // drag-and-drop position
  createdAt:   string;          // ISO datetime
  updatedAt:   string;          // ISO datetime
}
```

---

## Project Structure

```
task-manager/
├── artifacts/
│   ├── api-server/              # Express 5 REST API
│   │   ├── src/
│   │   │   ├── index.ts         # Server entry point (reads PORT env)
│   │   │   ├── app.ts           # Express app setup (CORS, logging, routes)
│   │   │   ├── routes/
│   │   │   │   ├── index.ts     # Route aggregator
│   │   │   │   └── tasks.ts     # All /api/tasks endpoints
│   │   │   ├── lib/
│   │   │   │   └── logger.ts    # Pino logger singleton
│   │   │   └── tests/
│   │   │       └── tasks.test.ts # Integration tests (node:test + supertest)
│   │   └── package.json
│   │
│   └── task-manager/            # React + Vite frontend
│       ├── src/
│       │   ├── pages/
│       │   │   └── home.tsx     # Main dashboard page
│       │   ├── components/
│       │   │   ├── task-item.tsx # Individual task card
│       │   │   └── task-form.tsx # Create / edit task dialog
│       │   ├── hooks/
│       │   │   └── use-debounce.ts
│       │   └── index.css        # Tailwind + CSS variables (Inter font, blue palette)
│       └── package.json
│
├── lib/
│   ├── db/                      # Drizzle ORM schema + client
│   │   ├── src/schema/tasks.ts  # PostgreSQL table definition with priority enum
│   │   └── drizzle.config.ts    # Migration/push config
│   │
│   ├── api-spec/                # OpenAPI 3.1 specification (source of truth)
│   │   └── openapi.yaml         # All endpoints, schemas, and types
│   │
│   ├── api-zod/                 # Auto-generated Zod validation schemas
│   │   └── src/generated/api.ts # From Orval codegen
│   │
│   └── api-client-react/        # Auto-generated TanStack Query hooks
│       └── src/generated/api.ts # useListTasks, useCreateTask, etc.
│
├── scripts/                     # Utility scripts workspace
├── pnpm-workspace.yaml          # Package catalog and workspace config
├── tsconfig.base.json           # Shared strict TypeScript defaults
└── README.md                    # This file
```

---

## Features

- **CRUD tasks** — create, read, update, delete with full validation
- **Toggle complete** — one-click completion with green checkmark
- **Delete with confirmation** — modal dialog before permanent deletion
- **Filter** — All / Active / Completed tabs
- **Task statistics** — live cards showing Total, Active, Completed, Overdue counts
- **Overdue highlighting** — red left border and OVERDUE badge for past-due tasks
- **Due-today highlighting** — amber left border for tasks due today
- **Task priorities** — High / Medium / Low / None with colour-coded flag badges
- **Search** — debounced real-time search by task title
- **Drag-and-drop reorder** — native HTML5 drag API with optimistic updates
- **Empty states** — contextual messages for each filter with a create prompt
- **CSV export** — download tasks as a spreadsheet with one click
- **Bulk operations** — complete all active tasks or delete all completed tasks
- **Corporate SaaS design** — dark navy sidebar, Inter font, blue `#2563EB` palette
- **Loading states** — spinner while fetching
- **Persistent storage** — PostgreSQL with Drizzle ORM (survives server restarts)

---

## Running Tests

```bash
# Run all API integration tests
pnpm --filter @workspace/api-server run test

# Tests cover:
#   ✅ GET /api/tasks returns 200 array
#   ✅ GET /api/tasks returns tasks sorted by sortOrder
#   ✅ GET /api/tasks?status=active returns only incomplete tasks
#   ✅ GET /api/tasks?status=completed returns only complete tasks
#   ✅ GET /api/tasks/stats returns numeric total/active/completed/overdue
#   ✅ stats total equals active + completed
#   ✅ POST /api/tasks returns 400 if title is missing
#   ✅ POST /api/tasks returns 400 if title is empty string
#   ✅ POST /api/tasks returns 201 with new task
#   ✅ PATCH /api/tasks/:id updates the task
#   ✅ PATCH /api/tasks/:id returns 404 for unknown id
#   ✅ PATCH /api/tasks/:id/toggle flips completed flag
#   ✅ DELETE /api/tasks/:id returns 204 and removes the task
#   ✅ DELETE /api/tasks/:id returns 404 for unknown id
#   ✅ GET /api/tasks/export returns text/csv with correct headers
#   ✅ PATCH /api/tasks/bulk/complete returns { count }
#   ✅ DELETE /api/tasks/bulk/completed returns { count }
```

---

## Next Steps

The following were intentionally omitted to keep scope focused:

- **Authentication** — all tasks are currently shared under one workspace; adding Clerk or Replit Auth would scope tasks per user
- **SQLite fallback** — the app uses PostgreSQL; a SQLite adapter via Drizzle is straightforward to add for local development without a hosted DB
- **Real-time sync** — WebSocket or SSE push to keep multiple browser tabs in sync
- **Recurring tasks** — tasks that automatically re-open on a schedule (daily, weekly, etc.)
- **Task comments / activity log** — audit trail of changes per task
- **File attachments** — attach files or images to tasks via object storage
- **Email reminders** — cron job that emails overdue task summaries
- **Mobile app** — Expo / React Native client using the same API contract
- **End-to-end tests** — Playwright for browser-level testing of the full flow
