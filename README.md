# Photo Studio OS Workspace

This directory is the local workspace for the Photo Studio OS project family.
It contains one outer repository plus three standalone child repositories.

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
| outer `Photo_Studio_OS` | Workspace index, legacy Command Center prototype docs, cross-repo orientation | `https://github.com/JENN2046/Photo_Studio_OS.git` |

## Current Source Of Truth

- Use `Photo_Studio_OS_Frontend` for current frontend work.
- Use `Photo_Studio_OS_Backend` for backend/API/database work.
- Use `Photo_Studio_OS_Control` for project planning, readiness, release, deploy, and boundary decisions.
- Treat the outer Next.js prototype files as legacy context unless a task explicitly revives them.

The outer repository currently ignores the three child repositories so each child keeps its own Git history and remote.

## Latest Local Baseline

Verified on 2026-05-24 after `git fetch --prune` and safe fast-forward where available:

| Repository | Branch | Local HEAD | Status |
|---|---|---:|---|
| outer `Photo_Studio_OS` | `main` | `8c2f83c` | workspace index baseline |
| `Photo_Studio_OS_Backend` | `main` | `2741fa5` | clean, aligned with `origin/main` |
| `Photo_Studio_OS_Frontend` | `main` | `699cf4a` | clean, aligned with `origin/main` |
| `Photo_Studio_OS_Control` | `main` | `d290a19` | clean, aligned with `origin/main` |

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
