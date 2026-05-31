# Photo Studio OS Workspace

This directory is the local parent directory for the complete Photo Studio OS
project.

The complete project is the three standalone child repositories together:
`Photo_Studio_OS_Control`, `Photo_Studio_OS_Backend`, and
`Photo_Studio_OS_Frontend`. The top-level `Photo_Studio_OS` repository is only
a local index/container for orientation and cross-repo notes; it is not a
separate product implementation surface.

## Repository Roles

```text
A:\Photo_Studio_OS
├─ Photo_Studio_OS_Control   project control tower
├─ Photo_Studio_OS_Backend   backend engine
└─ Photo_Studio_OS_Frontend  frontend cockpit
```

| Path | Role | Remote |
|---|---|---|
| `Photo_Studio_OS_Control` | Planning, governance, safety gates, release/deploy readiness notes | `https://github.com/JENN2046/Photo_Studio_OS_Control.git` |
| `Photo_Studio_OS_Backend` | NestJS API, Prisma/PostgreSQL state, RBAC, ActivityLog, workflow truth | `https://github.com/JENN2046/Photo_Studio_OS_Backend.git` |
| `Photo_Studio_OS_Frontend` | Vite/React read-only Command Center cockpit and future read-model UI | `https://github.com/JENN2046/Photo_Studio_OS_Frontend.git` |
| top-level `Photo_Studio_OS` | Parent directory and local index only; not a product implementation repo | `https://github.com/JENN2046/Photo_Studio_OS.git` |

## Current Source Of Truth

- Use `Photo_Studio_OS_Frontend` for current frontend work.
- Use `Photo_Studio_OS_Backend` for backend/API/database work.
- Use `Photo_Studio_OS_Control` for project planning, readiness, release, deploy, and boundary decisions.
- Treat the top-level Next.js prototype files as dormant legacy context unless a task explicitly revives them.

The top-level repository currently ignores the three child repositories so each child keeps its own Git history and remote.

## Latest Local Checkpoint

Verified on 2026-05-31 after the Backend T127 read-model branch was
fast-forwarded into `main` and the Backend `operator` read-role compatibility
fix was pushed. No release, deploy, tag, production action, secret change, or
schema change was run during this checkpoint. Local backend Docker dependencies
were started only for read-model e2e, full Backend validation, queue validation,
and backend-read smoke validation.

| Repository | Branch | Local HEAD | Status |
|---|---|---:|---|
| top-level `Photo_Studio_OS` | `main` | `1e22c94` | docs changed; parent directory and local index only |
| `Photo_Studio_OS_Backend` | `main` | `bd97a60` | base aligned with `origin/main`; local contract-check wiring pending commit |
| `Photo_Studio_OS_Frontend` | `main` | `699cf4a` | no source changes in this slice |
| `Photo_Studio_OS_Control` | `main` | `ac37cfb` | planning and decision docs changed |

## Validation Entry Points

Use child repositories for current validation. The complete project is the
combined state of the three child repositories. The top-level Next.js prototype
is dormant legacy context and is not the canonical frontend validation target.

Current known-good local checks from 2026-05-31:

```powershell
cd A:\Photo_Studio_OS\Photo_Studio_OS_Frontend
npm ci
npm run lint

cd A:\Photo_Studio_OS\Photo_Studio_OS_Backend
npm run contract:check
npm run validation:full
node --test tests\request-context-auth.test.js

cd A:\Photo_Studio_OS\Photo_Studio_OS_Frontend
powershell -ExecutionPolicy Bypass -File scripts\qa-backend-read-smoke.ps1 `
  -BackendBaseUrl "http://127.0.0.1:3001/api/v2/read" `
  -AssetInboxExpectedReadModelState partial `
  -QcRetouchExpectedReadModelState empty `
  -DeliveryReadinessExpectedReadModelState empty
```

Notes:

- Backend preflight passed with the expected external-port warning that
  `6005` and `6006` were already listening for VCPToolBox.
- Backend `/api/v2/read` is now implemented in `src/read-models` on Backend
  `main` for Command Center v2, Asset Inbox, QC / Retouch, Review Gallery, and
  Delivery Readiness. The read models use public IDs, route-specific RBAC, and
  persisted backend data. Public review/delivery access remains disabled.
- Backend `operator` is a supported read role for local/staging backend-read
  smoke; unknown runtime roles are denied instead of causing 500 errors.
- Local Backend follow-up wiring adds `npm run contract:check` and includes it
  in `npm run validation:full`; this local wiring has not been pushed yet.
- Local backend Docker validation containers were stopped afterward with
  `npm run infra:down`; validation volumes were not removed.
- The top-level `npm run typecheck` is not a source-of-truth check while the
  dormant legacy Next prototype and ignored child repositories share the same
  parent directory.
- Release, deploy, tag, production changes, and future remote writes remain
  separate explicit actions.

## Port Rules

Do not bind, stop, or repurpose these external-owned ports:

```text
3000  NewAPI / external owned service
6005  VCPToolBox backend
6006  VCPToolBox Admin
6379  default Redis / not Photo Studio validation Redis
```

Known Photo Studio local validation ports:

```text
3001  approved local backend API target in frontend docs
3100  backend runtime candidate
5173  frontend Vite dev server
4173  frontend Vite preview
6380  Photo Studio Redis validation
```

## Common Entry Points

Frontend:

```powershell
cd A:\Photo_Studio_OS\Photo_Studio_OS_Frontend
npm ci
npm run dev
```

Backend:

```powershell
cd A:\Photo_Studio_OS\Photo_Studio_OS_Backend
npm ci
npm run prisma:generate
npm run validation:preflight
```

Control tower:

```powershell
cd A:\Photo_Studio_OS\Photo_Studio_OS_Control
```

Read `PROJECT_MASTER_PLAN.md` before planning release, deploy, backend contract, or cross-repo work.

## Cross-Repo Notes

- Frontend remains mock-first and read-only by default.
- Backend owns durable business truth.
- Control records planning and safety decisions; it is not a runtime package.
- Real backend connection, auth, upload, download, public review, public delivery, release, deploy, push, tag, and production work require explicit approval.

See `docs/06_repository_topology_and_contract_map.md` for the current repository relationship and frontend/backend read-model gap map.
