# Repository Topology And Contract Map

Last verified: 2026-05-31

## Purpose

This note records the current local workspace topology and the relationship
between the top-level parent directory/index repository and the three standalone
child repositories.

The complete Photo Studio OS project is the combined state of the three child
repositories: Control, Backend, and Frontend. The top-level `Photo_Studio_OS`
directory is a parent directory and local index only.

## Local Repositories

| Repository | Local path | Current role | Latest local HEAD |
|---|---|---|---:|
| Top-level parent/index | `A:\Photo_Studio_OS` | Parent directory and local index only | `1e22c94` |
| Control | `A:\Photo_Studio_OS\Photo_Studio_OS_Control` | Control tower: plans, gates, readiness, release/deploy boundaries | `ac37cfb` |
| Backend | `A:\Photo_Studio_OS\Photo_Studio_OS_Backend` | Backend engine: NestJS, Prisma, PostgreSQL, queues, API truth | `bd97a60` |
| Frontend | `A:\Photo_Studio_OS\Photo_Studio_OS_Frontend` | Frontend cockpit: Vite/React read-only Command Center and read models | `699cf4a` |

## Recommended Ownership

```text
Photo_Studio_OS_Control  = project control tower
Photo_Studio_OS_Backend  = backend engine
Photo_Studio_OS_Frontend = frontend cockpit
top-level Photo_Studio_OS = parent directory and local index only
```

`Photo_Studio_OS` should be treated as the project parent directory, not as a
fourth product repository. Product implementation work belongs in the relevant
child repository.

The top-level Next.js prototype files are dormant legacy context. They currently
reference prototype components and data-entry files that are not present in the
top-level repository, and the top-level TypeScript configuration can also scan
ignored child repository sources. Therefore the top-level `npm run typecheck`
is not a canonical project validation target unless the legacy prototype is
explicitly revived and repaired.

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
- Read-model API under `/api/v2/read`.
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
GET /api/v2/read/command-center/v2
GET /api/v2/read/projects/:projectId/asset-inbox
GET /api/v2/read/projects/:projectId/qc-retouch-queue
GET /api/v2/read/review-sessions/:reviewSessionId/gallery
GET /api/v2/read/deliveries/:deliveryId/readiness
```

## Current Read-model Contract

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

The backend now exposes these `/api/v2/read` route paths on `main`. Current
state:

- `GET /api/v2/read/command-center/v2` is backed by existing dashboard state.
- `GET /api/v2/read/projects/:projectId/asset-inbox` is backed by persisted
  asset, SKU, shot requirement, filename match, and latest QC data for UUID
  project IDs.
- `GET /api/v2/read/projects/:projectId/qc-retouch-queue` is backed by
  persisted latest QC, latest retouch task, latest asset version, SKU, and shot
  requirement data.
- `GET /api/v2/read/review-sessions/:reviewSessionId/gallery` is backed by
  persisted review sessions, review items, asset previews, SKU references, shot
  requirement references, and summary counts. Public review access remains
  disabled.
- `GET /api/v2/read/deliveries/:deliveryId/readiness` is backed by persisted
  delivery package metadata, delivery items, checklist state, blockers, and
  external access status. Public delivery/download access remains disabled.
- Public IDs are accepted for the frontend smoke path; unresolved public IDs
  return guarded 404/empty behavior instead of leaking un-scoped data.
- The Backend `operator` role can read these models in local/staging smoke.

The remaining integration gap is not route existence. The next gap is contract
hardening: examples, drift-control checks, response-shape audit, and explicit
production auth/storage/public-access decisions.

## Approved Integration Direction

Do not connect production or add write behavior.

The approved durable integration direction is:

1. Keep `Photo_Studio_OS_Frontend` mock-first and read-only by default.
2. Keep backend-owned `/api/v2/read` read-model endpoints as the durable
   frontend integration contract.
3. Continue hardening those endpoints in `Photo_Studio_OS_Backend` without
   adding frontend writes, production auth, upload/download, public
   review/delivery, release, or deploy behavior.
4. Use frontend backend-read smoke only against an approved local or staging
   base URL such as `http://127.0.0.1:3001/api/v2/read`.
5. Validate each contract change with focused Backend tests and frontend
   backend-read smoke when a local backend can run safely.

A frontend `/api/v1` compatibility adapter remains allowed only as a short
local smoke workaround if the backend read-model shape is temporarily moving.
It should not become the durable production integration contract.

## Validation Notes

This document was refreshed from local inspection and validation on 2026-05-31.

Validation observed during the 2026-05-31 status review:

```text
Photo_Studio_OS_Frontend: npm ci; npm run lint
Photo_Studio_OS_Backend: npm run contract:check
Photo_Studio_OS_Backend: npm run validation:full
Photo_Studio_OS_Backend: node --test tests\request-context-auth.test.js
Photo_Studio_OS_Frontend: scripts/qa-backend-read-smoke.ps1 against http://127.0.0.1:3001/api/v2/read
```

The Backend read-model e2e now covers public IDs, Command Center, Asset Inbox,
QC / Retouch, Review Gallery, Delivery Readiness, filters, unknown-ID behavior,
and the `operator` read role.

Backend preflight passed with the expected warning that external-owned
VCPToolBox ports `6005` and `6006` were listening.

Local Docker PostgreSQL/Redis dependencies and a temporary backend process on
port `3001` were used only for local validation. The Docker containers were
stopped afterward with `npm run infra:down`; validation volumes were not
removed. Backend commit/push was explicitly approved for `bd97a60`. No release
or deploy was run as part of this topology update. A local follow-up wires
`contract:check` into Backend `validation:full`; that wiring has not been
pushed yet.
