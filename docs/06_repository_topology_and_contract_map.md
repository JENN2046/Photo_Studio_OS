# Repository Topology And Contract Map

Last verified: 2026-05-24

## Purpose

This note records the current local workspace topology and the relationship between the outer repository and the three standalone child repositories.

## Local Repositories

| Repository | Local path | Current role | Latest local HEAD |
|---|---|---|---:|
| Outer workspace | `A:\Photo_Studio_OS` | Workspace index and legacy Next prototype context | `8c2f83c` |
| Control | `A:\Photo_Studio_OS\Photo_Studio_OS_Control` | Control tower: plans, gates, readiness, release/deploy boundaries | `d290a19` |
| Backend | `A:\Photo_Studio_OS\Photo_Studio_OS_Backend` | Backend engine: NestJS, Prisma, PostgreSQL, queues, API truth | `2741fa5` |
| Frontend | `A:\Photo_Studio_OS\Photo_Studio_OS_Frontend` | Frontend cockpit: Vite/React read-only Command Center and read models | `699cf4a` |

## Recommended Ownership

```text
Photo_Studio_OS_Control  = project control tower
Photo_Studio_OS_Backend  = backend engine
Photo_Studio_OS_Frontend = frontend cockpit
outer Photo_Studio_OS    = local workspace index and legacy context
```

The outer repository should not become the active frontend implementation while `Photo_Studio_OS_Frontend` is the current cockpit.

## Current Frontend State

`Photo_Studio_OS_Frontend` is the current frontend mainline.

It now includes:

- Vite + React + TypeScript.
- Read-only Command Center cockpit.
- Four read-model workspaces:
  - `#asset-inbox`
  - `#qc-retouch`
  - `#review-gallery`
  - `#delivery-readiness`
- Mock-first data behavior.
- Optional backend read activation through `VITE_BACKEND_API_BASE_URL`.
- Auth/role readiness scaffolding and local QA scripts.
- Backend read smoke scripts for approved local/staging targets.

Blocked without explicit approval:

- production auth
- upload/download
- public review or delivery links
- approval/write actions
- deployment or release

## Current Backend State

`Photo_Studio_OS_Backend` is the current backend mainline.

It provides:

- NestJS API under `/api/v1`.
- Prisma/PostgreSQL persistence.
- RBAC and request context.
- ActivityLog-backed domain state changes.
- Project, SKU, shot requirement, asset, retouch, QC, review, delivery, dashboard, and command-center surfaces.
- Static Review/Delivery OpenAPI contract artifacts.

Known read endpoints include:

```text
GET /api/v1/command-center
GET /api/v1/projects/:projectId/dashboard
GET /api/v1/projects/:projectId/assets
GET /api/v1/projects/:projectId/qc-checks
GET /api/v1/projects/:projectId/review-sessions
GET /api/v1/review-sessions/:reviewSessionId
GET /api/v1/projects/:projectId/deliveries
GET /api/v1/deliveries/:deliveryId
```

## Current Contract Gap

The updated frontend expects a future read-model base URL, commonly documented as:

```text
http://127.0.0.1:3001/api/v2/read
```

Frontend read-model fetchers currently target:

```text
/command-center/v2
/projects/:projectId/asset-inbox
/projects/:projectId/qc-retouch-queue
/review-sessions/:reviewSessionId/gallery
/deliveries/:deliveryId/readiness
```

The backend currently exposes broader `/api/v1` operational and dashboard endpoints, not all of these `/api/v2/read` read-model routes.

## Recommended Next Integration Step

Do not connect production or add write behavior.

The next safe integration step is one of:

1. Add a backend read-model compatibility plan in Control, then implement approved `/api/v2/read` endpoints in Backend.
2. Add a frontend adapter compatibility plan that maps current `/api/v1` backend responses into the frozen frontend read models.

Prefer option 1 when the backend contract should become durable and reusable.
Prefer option 2 only for short local smoke if the backend shape is still moving.

## Validation Notes

This document was created from read-only inspection plus safe `git fetch --prune` and `git pull --ff-only` for the Frontend repository.

No build, test, backend service, database, Docker, browser QA, commit, push, release, or deploy was run as part of this topology update.
