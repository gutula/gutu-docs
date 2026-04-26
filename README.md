# Gutu — Documentation

The complete end-to-end documentation set for the [Gutu](https://github.com/gutula/gutu)
platform: a contract-first plugin framework for building modular, multi-tenant
admin applications with backend, frontend, and operational concerns all
plugin-driven.

> If you're new here, read in this order:
> 1. **[ARCHITECTURE.md](./docs/ARCHITECTURE.md)** — what Gutu is and how it's built
> 2. **[PLUGIN-DEVELOPMENT.md](./docs/PLUGIN-DEVELOPMENT.md)** — building your first plugin
> 3. **[HOST-SDK-REFERENCE.md](./docs/HOST-SDK-REFERENCE.md)** — every API you can call
> 4. **[UI-UX-GUIDELINES.md](./docs/UI-UX-GUIDELINES.md)** — when you start building pages

---

## For developers

| Doc | Read it when |
|---|---|
| [docs/ARCHITECTURE.md](./docs/ARCHITECTURE.md) | First — the system overview, repo layout, what's a plugin vs what stays in the shell |
| [docs/PLUGIN-DEVELOPMENT.md](./docs/PLUGIN-DEVELOPMENT.md) | You're building a plugin (backend + frontend + UI/UX + testing + distribution) |
| [docs/HOST-SDK-REFERENCE.md](./docs/HOST-SDK-REFERENCE.md) | You need to know what `@gutu-host` exports — every function, every type |
| [docs/UI-UX-GUIDELINES.md](./docs/UI-UX-GUIDELINES.md) | You're building admin pages — design tokens, components, accessibility |
| [docs/TESTING.md](./docs/TESTING.md) | You're writing tests — the four shell suites and plugin-author harness |
| [docs/CONTRIBUTING.md](./docs/CONTRIBUTING.md) | You're sending a PR — code style, commit format, review process |
| [PLUGIN_AUTHORING.md](./PLUGIN_AUTHORING.md) | Quickstart for plugin authors (legacy short version — see PLUGIN-DEVELOPMENT.md for depth) |

## For operators

| Doc | Read it when |
|---|---|
| [DEPLOYMENT.md](./DEPLOYMENT.md) | You're deploying to production — env vars, k8s probes, scaling, backup/restore, security checklist |
| [RUNBOOK.md](./RUNBOOK.md) | You're on call — triaging plugin/worker/audit/rate-limit issues, GDPR fulfilment, plugin enablement |
| [docs/OBSERVABILITY.md](./docs/OBSERVABILITY.md) | You're wiring monitoring — logs, metrics, health probes, leases, alert thresholds |
| [docs/SECURITY.md](./docs/SECURITY.md) | You're hardening or auditing — auth, RBAC, ACL, GDPR, audit hash chain, threat model |

---

## What's documented

### Architecture
- The 30-second mental model
- Shell vs plugin boundary (rule of thumb: if it's installable, it's a plugin)
- Backend `HostPlugin` contract and frontend `AdminUiContribution` contract
- Decentralized plugin discovery via `package.json["gutuPlugins"]`
- Boot sequence (~19 steps, with per-plugin try/catch isolation)
- Lifecycle hooks: `migrate`, `install`, `start`, `stop`, `seed`, `health`, `uninstall`, `exportSubjectData`, `deleteSubjectData`
- Middleware stack: drain · trace · security headers · body cap · rate-limit · metrics · CORS · tenant
- Multi-tenant scoping via AsyncLocalStorage
- Per-tenant plugin enablement
- Cross-plugin contracts (provides/consumes registry pattern)
- Worker patterns (leader election + `withLeadership`)
- Storage adapter registry (local + S3)
- Frontend routing, plugin UI composition, detail-page rails
- Data model: generic `records` table + plugin-owned tables
- ACL layers (RBAC + per-record + per-tenant)
- In-process event bus

### Plugin development (end to end)
- Five-minute walkthrough from `bun run scaffold:plugin` to running
- Folder layout and what lives where
- Every `HostPlugin` field with full type signatures
- Writing routes (Hono, Zod validation, per-tenant gating, per-record ACL)
- Schema (CREATE IF NOT EXISTS conventions, schema evolution via PRAGMA-checked ALTER)
- Frontend `AdminUiContribution`: pages, nav, commands, detail rails, lifecycle hooks
- Available imports from the shell (primitives, admin-primitives, runtime helpers)
- Cross-plugin contracts via `ctx.registries.ns().register()/.lookup()`
- Workers with leader election + idempotency markers
- Permissions declaration and enforcement
- Testing patterns (lifecycle, schema, routes, integration, frontend)
- Distribution: monorepo dev → npm publish → customer install
- 7+ common pitfalls + solutions
- Plugin-author checklist (20+ items)

### UI/UX guidelines
- Design principles
- Page anatomy (header, filter bar, content)
- Detail-page anatomy (`RichDetailPage`)
- Typography tokens
- Color tokens (semantic, theme-safe)
- Density modes (comfortable / compact)
- Motion and animation
- Component patterns: Button, Badge, Card, Dialog, Toast, Input, Switch, Select, Tabs
- State patterns: empty, loading, error, submitting
- Forms: validation, defaults, helper text, save/cancel
- Tables: alignment, pagination, row actions
- Cmd-K palette conventions
- Detail-page rail cards
- Accessibility: keyboard, screen readers, contrast, focus rings, motion preferences
- Performance: data fetching, rendering, bundle size
- Microcopy: labels, confirmations, errors, empty states
- The polish checklist

### Host SDK reference
Every export, every signature:
- `@gutu-host` — db, nowIso, uuid, token, recordAudit, getTenantContext, requireAuth, currentUser, Hono
- `@gutu-host/plugin-contract` — every HostPlugin field documented
- `@gutu-host/leader` — withLeadership, acquireOnce, listLeases
- `@gutu-host/acl` — seedDefaultAcl, effectiveRole, accessibleRecordIds, …
- `@gutu-host/event-bus` — emitRecordEvent, subscribeRecordEvents
- `@gutu-host/field-metadata` — validateRecordAgainstFieldMeta, listFieldMetadata
- `@gutu-host/query` — parseListQuery, listRecords, getRecord, insertRecord, updateRecord, deleteRecord, bulkInsert
- `@gutu-host/storage` — getStorageRegistry, ObjectNotFound, isStorageError
- `@gutu-host/ws` — broadcastResourceChange, registerSocket, unregisterSocket
- `@gutu-host/yjs-room` — yjsOnOpen, yjsOnMessage, yjsOnClose
- `@gutu-host/timeline-helpers` — appendTimelineEvent, readTimeline, startTimelineWriter
- `@gutu-host/awesome-bar` — searchAwesome
- `@gutu-host/webhook-dispatcher` — startWebhookDispatcher
- `@gutu-host/plugin-ui-contract` — defineAdminUi, AdminUiContribution types
- `@gutu-host/tenant-enablement` — pluginGate, isPluginEnabled, setPluginEnabled
- `@gutu-host/permissions` — enforce, registerPluginPermissions, withPluginScope

### Security
- Threat model and trust boundaries
- Auth flow: sign-in, MFA, sessions, API tokens, password reset, email verify
- Authorization: RBAC + per-record ACL + per-tenant gate
- GDPR Article 20 (export) + Article 17 (delete) fan-out
- Audit hash chain + verify endpoint
- Secrets management
- Plugin permissions enforcement modes (enforce / warn / off)
- Network perimeter (CORS, headers, rate limit, body size)
- Tenancy isolation guarantees + failure modes
- Webhooks security (HMAC + retry)
- Storage security
- WebSocket authorization
- Boot-time hardening
- Defense-in-depth checklists for plugin authors AND operators
- Vulnerability reporting

### Observability
- Liveness vs readiness probes (k8s example)
- Structured JSON logs and trace ID propagation
- `/api/_metrics` snapshot shape
- `/api/_plugins` plugin status surface
- `/api/_plugins/_leases` cluster-singleton workers
- Audit chain verification (with cron recipe)
- Common signals + diagnoses
- Recommended dashboards (per-instance / per-tenant / per-plugin)
- Alert thresholds with severities
- End-to-end troubleshooting flow

### Operations
- Production deploy via Docker
- Required + optional environment variables
- Kubernetes probe configuration
- Horizontal scaling (worker leader election + DB-backed rate limit)
- Graceful HTTP drain on SIGTERM
- Backup / restore for SQLite
- Day-to-day on-call: triaging plugin / worker / audit / rate-limit issues
- GDPR fulfilment: Article 20 export + Article 17 erasure
- Per-tenant plugin enablement
- Deploying a new plugin (from npm or local)

---

## How this repo relates to the platform

```
github.com/gutula/
├── gutu                           # Main repo: shell + bundled plugins (admin-panel, plugins/*)
├── gutu-docs                      # ← you are here: documentation
├── gutu-plugin-accounting-core    # First-party domain plugins (60+ repos)
├── gutu-plugin-sales-core
├── gutu-plugin-inventory-core
├── ...
└── gutu-plugin-template-core
```

This `gutu-docs` repo is the **single source of truth for documentation**.
The main `gutu` repo also carries copies for convenience but treat this
repo as canonical — it's what `gutula.dev/docs` (and any future static
site builder) renders from.

---

## Contributing to docs

Docs follow the same workflow as code: see
[docs/CONTRIBUTING.md](./docs/CONTRIBUTING.md). Open a PR against
`main`, CI runs spell-check + link-check, at least one approval,
squash-merge.

---

## License

[MIT](LICENSE) (or whatever the parent project ships under — sync from
the main `gutu` repo).
