# Leonidas - Tech Stack Research

> Companion to `PRD-leonidas.md`. Scope: choose a stack and architecture for an asset management system that has to do **strong, auditable credits/debits of serialized assets across multiple physical locations, many times per day**, with offline-tolerant mobile, consignment-partner reporting, and equity accounting.

---

## 1. The Real Problem in One Sentence

This is **double-entry bookkeeping for physical things**. Every asset movement is a transaction with a source and destination, must be atomic, must never silently overwrite prior state, and must reconcile to a current position that ties out across all locations. The same problem accountants solved 700 years ago with the journal + ledger pattern.

If you treat Leonidas as "a database of where things are right now," you will rebuild Delmar's Excel pain in code: cells get out of sync, no one knows who changed what, and a missed update means a missed PO. If you treat it as **an immutable movement ledger that derives current state**, you get auditability, shortage-detection, equity accrual, and dispute resolution for free.

That architectural decision drives every other choice below.

---

## 2. Architectural Core: The Asset Movement Ledger

### 2.1 Pattern

Two layers:

1. **`asset_movements`** -- append-only, immutable. Every loadout, check-in, transfer, intake, return, scrap, sub-assembly build/break is one row. Never UPDATE, never DELETE. Corrections are *reversing entries* (negate the original, then post the correct one), exactly like accounting.
2. **`asset_state`** -- derived. Current location, status, job assignment, sub-assembly membership for each serial. Either a materialized view refreshed on movement, or a regular table maintained by trigger / stored procedure inside the same transaction as the movement insert.

### 2.2 Movement row -- minimum fields

```
movement_id          uuid pk
ts                   timestamptz                 -- when it physically happened (not when it was logged)
logged_at            timestamptz default now()   -- when it was entered
asset_id             fk -> assets.id (serial)
sub_assembly_id      fk nullable                 -- if part of a frac stack at time of movement
movement_type        enum (loadout, checkin, transfer_out, transfer_in,
                            intake_consignment, intake_owned, return_to_partner,
                            scrap, sub_build, sub_break, adjustment)
from_location_id     fk nullable                 -- null = appearance (new intake)
to_location_id       fk nullable                 -- null = disappearance (scrap/return)
job_id               fk nullable
ticket_id            fk -> loadout/checkin tickets
day_rate_applied     numeric nullable            -- snapshot of rate at time of movement
created_by           fk -> users
device_id            text                        -- which scanner/phone
geo_lat, geo_lng     numeric nullable            -- captured on mobile movements
idempotency_key      text unique                 -- for safe mobile retry
notes                text
reversal_of          fk -> movement_id nullable  -- non-null if this is a correction
```

### 2.3 Why this pattern fits Spartan specifically

| PRD pain / requirement | How the ledger solves it |
|---|---|
| "Missed billable job because consignment came back in but cell stayed green" (§1.3) | A check-in is a row that *cannot* be forgotten without leaving the asset in transit -- planning board surfaces in-transit assets as exceptions |
| Shortage Resolution Waterfall (UF3) | Per-location counts are a `SELECT count(*) FROM asset_state WHERE location_id = $1 AND ownership_class = $2` -- always correct, derived not maintained |
| Equity accrual (§1.5.1) | Per-asset rental-day-revenue = aggregation over loadout/checkin movement pairs. Recomputable, not stored-and-drifted |
| KPA cert packet generation (UF7, F7) | Loadout ticket has exact serials; one query returns the cert artifacts |
| "Ryan called mid-meeting and we went from -2 to -6 valves" (Apr 7 transcript) | Real-time aggregate view recomputes on every movement; planning board reflects new shortage instantly |
| Dispute with partner over month-end statement | Replay the ledger -- *the ledger is the audit trail*, no extra work |

### 2.4 The non-negotiable rule

**Clients never write to `asset_state` directly.** Every state change goes through a single Postgres function (`fn_post_movement(...)`) that:
1. Validates the movement (asset exists, source location matches current state, idempotency key not seen)
2. Inserts the movement row
3. Updates derived state in the same transaction
4. Emits a realtime event

Enforced by RLS / grants: app role only has EXECUTE on the function, no INSERT/UPDATE on the underlying tables. This is what makes the ledger trustworthy under concurrent writes.

---

## 3. Concurrency -- The Actually Hard Part

### 3.1 The race condition

Two yard hands in different districts simultaneously assign serial `MAV-1042` to two different jobs. Without protection, both writes succeed and the asset is "in two places."

### 3.2 Three viable approaches

| Approach | How it works | When to use |
|---|---|---|
| **Pessimistic row lock** | `SELECT ... FROM asset_state WHERE asset_id = $1 FOR UPDATE` inside the movement function. Second writer blocks until the first commits, then re-validates and likely fails the precondition check. | **Recommended for Leonidas.** ~500 assets, contention is rare; lock wait will be milliseconds; logic is dead-simple. |
| **Optimistic version column** | `asset_state` has a `version` int. Movement function does `UPDATE ... SET version = version + 1 WHERE version = $expected`. On conflict, return error and let client retry. | If you ever scale to 50K+ assets where pessimistic locks become contention hotspots. Not now. |
| **Serializable isolation** | Set transaction isolation to SERIALIZABLE; Postgres detects conflicts and aborts the loser. | Cleanest theoretically but harder to debug retry storms. Overkill at this scale. |

### 3.3 Mobile/offline retry

Field scanner posts a movement, gets no response (cell drops). On reconnect, posts again. Without protection, the asset moves twice.

**Fix:** Every mobile mutation includes a client-generated `idempotency_key` (UUID). The server stores it; second post with same key returns the original result, no-op. This is standard fintech practice (Stripe, etc.) and is non-negotiable for offline-tolerant mobile.

---

## 4. Stack Recommendation

### 4.1 Headline pick

> **Postgres (Supabase) + Next.js (App Router, TypeScript) on Vercel + PWA mobile + Tailwind + minimal custom code on top of Supabase RPCs.**

This is the lowest-friction stack that gives Leonidas everything it needs: ACID transactions, RLS for district-scoped users, realtime for the planning board, auth, file storage for cert PDFs, and a fast mobile-installable web app. Alex already has Supabase in the workspace toolchain, which collapses the integration risk.

### 4.2 Component-by-component

| Layer | Pick | Why | Alternatives considered |
|---|---|---|---|
| **Database** | **Postgres 15+ (via Supabase)** | ACID, mature, supports the ledger pattern natively, jsonb for sub-assembly BOMs, generated columns, row-level security | MySQL (weaker jsonb / RLS story), SQLite (no concurrency), DynamoDB / Firebase (wrong shape -- you'd reinvent joins for utilization reports) |
| **Backend / API** | **Supabase RPC (Postgres functions) + PostgREST** | All movement logic lives in `fn_post_movement` and friends -- one source of truth, transactional, no separate API server to deploy. RLS handles authorization. | Custom Node/Rails API: more code, more deploys, more places for state to drift. Worth it later if business logic outgrows SQL. |
| **Auth** | **Supabase Auth** (email + magic link for office, phone OTP for yard) | Built-in, integrates with RLS, supports SSO later | Auth0 / Clerk: nicer DX but extra cost & vendor for no additional capability here |
| **Realtime** | **Supabase Realtime** (Postgres logical replication) | Push movement events to the planning board so Delmar's view updates the moment a yard scanner posts | Pusher / Ably: extra service. Polling: fine fallback if realtime gets flaky. |
| **Frontend (desktop)** | **Next.js 14 App Router + TypeScript + Tailwind** | Server actions map cleanly onto Supabase RPCs; React Server Components keep the planning board fast; existing mockup is HTML/CSS so port path is short | Remix (similar tradeoffs), SvelteKit (smaller ecosystem for the data-grid widgets you'll need), pure HTMX (great for forms, awkward for the live planning board) |
| **Frontend (mobile)** | **Same Next.js app as installable PWA** | One codebase. PWA gives camera (QR scan), geolocation, IndexedDB for offline queue, home-screen install. Field crews don't need an app store. | React Native / Expo: only worth it if you need background scanning or BLE for RFID. Not Phase 1. |
| **Offline queue** | **Custom IndexedDB write-ahead log + idempotency keys** | ~150 lines of TS. Movements queue locally, replay on reconnect, server dedupes by key. | PowerSync / ElectricSQL: powerful CRDT sync, but $$$ and overkill for "queue 20 movements until LTE comes back." Revisit if requirements grow. |
| **QR / barcode scanning** | **`html5-qrcode` or `zxing-js`** in browser | Works in PWA, no native code | Native scanner SDKs: only if you go React Native |
| **PDF (consignment statements, cert packets)** | **Puppeteer on a Vercel function**, or **`@react-pdf/renderer`** | Render the same React component server-side to PDF; statements look identical to on-screen | DocRaptor / PDFShift: $/month for what you can do free |
| **Background jobs** | **Supabase `pg_cron`** for monthly statement runs; Vercel Cron for anything that needs HTTP | No new infra | A worker queue (BullMQ, Inngest): only if jobs get heavy |
| **File storage** | **Supabase Storage** for cert PDFs, partner statements, photos of damaged assets | Same auth, same RLS, S3-compatible | Direct S3: more glue code |
| **Observability** | **Supabase logs + Sentry** for the frontend | Cheap and sufficient at this scale | Datadog: enterprise-class, enterprise-priced |
| **Hosting** | **Vercel (frontend) + Supabase (everything else)** | Two dashboards, two bills, near-zero ops | Self-host on a VPS: cheaper at scale, slower to ship at Phase 1 |

### 4.3 Estimated monthly run cost (Phase 1, ~10 office users + ~30 mobile users)

| Service | Tier | Cost |
|---|---|---|
| Supabase | Pro ($25) | $25/mo |
| Vercel | Pro (likely Hobby suffices initially) | $0-$20/mo |
| Sentry | Free / Team | $0-$26/mo |
| Domain | -- | ~$1/mo |
| **Total** | | **~$25-$75/mo** |

Well inside the "one-time $10K + reasonable subscription" envelope Delmar discussed.

---

## 5. Industry Best Practices for Multi-Location Asset Credit/Debit Systems

What the broader space does, and which patterns Leonidas should adopt vs. ignore.

### 5.1 Patterns to adopt

1. **Append-only event store / movement ledger** -- standard in WMS (SAP EWM, Manhattan, Oracle WMS) and inventory accounting. Adopt fully.
2. **Idempotency keys on every mobile mutation** -- standard in fintech (Stripe, Square). Adopt fully.
3. **Single transactional procedure for state changes** -- prevents the "who else is updating this row" class of bug that plagued Delmar's Excel sheet.
4. **Snapshot rates/terms onto the movement** -- if MAV's rate changes mid-month, prior movements still bill correctly. Standard accounting practice (don't mutate posted entries).
5. **Reversing entries instead of edits/deletes** -- preserves history, gives Delmar the receipts when a partner disputes a number.
6. **Generated/derived current state, never directly written** -- WMS systems call this "perpetual inventory derived from transactions." Same idea.
7. **Per-location ledgers reconcilable to a global position** -- sum of district counts = company count. Build the planning board so Delmar can *see* this reconcile in real time, because right now he can't.
8. **Geocode every mobile movement** -- free truth signal that supplements (and quietly replaces) the failed GeoTrackers Delmar wrote off.
9. **Cycle-count workflow as a first-class feature** -- yard does periodic spot counts; system records discrepancies as `adjustment` movements with required notes. Catches drift before it becomes a missed PO.

### 5.2 Patterns to skip (for now)

1. **RFID / IoT trackers** -- Delmar already burned on GeoTrackers (forklifts knock them off, ~30% loss rate). The mobile-ledger discipline gives 95% of the value at 5% of the cost. Revisit only if discipline alone proves insufficient.
2. **Full WMS suite (SAP / Oracle / Manhattan)** -- $250K+ implementations, 6-12 month deploys, doesn't model consignment partner equity natively. Every dollar spent here is a dollar not spent on the things that actually hurt (consignment reporting, shortage waterfall, mobile loadout).
3. **Off-the-shelf asset SaaS (Asset Panda, EZOffice, Snipe-IT)** -- generic, no consignment economics, no shortage waterfall logic, no oilfield workflows. Building on Postgres + Supabase is faster than bending one of these to fit.
4. **Blockchain / distributed ledger** -- no. The trust boundary is internal + a handful of named partners; a Postgres ledger with audit logs is more than sufficient and infinitely easier to operate.
5. **Microservices** -- one Postgres + one Next.js app is the right size for Phase 1 and probably Phase 3. Don't fragment until you have a reason.

### 5.3 Specialized ledger databases (optional, future)

**TigerBeetle** is a purpose-built financial ledger database (designed for double-entry transactions at fintech-scale). It's the "if we ever outgrow Postgres for the ledger" answer, not the "start here" answer. At 500 assets and a few hundred movements per day, Postgres handles it without thinking. Worth knowing the option exists.

---

## 6. Phasing the Build Against the PRD

| Phase | What gets built | Tech work |
|---|---|---|
| **Phase 0 (now)** | Styled HTML mockup | Done. Use it to drive Delmar buy-in. |
| **Phase 1** | Equipment Registry + Job Planning Board + Mobile Check-In/Out | Spin up Supabase project; build `asset_movements` + `fn_post_movement`; Next.js shell with planning board (server component + realtime); PWA scanner page; offline queue. |
| **Phase 2** | Consignment reporting + forecasting + machine shop | Add equity accrual aggregation views; PDF statement generator; capital-planning utilization report; sub-rental procurement workflow (UF17). |
| **Phase 3** | KPA integration + analytics + advanced scheduling | KPA API integration (or scrape if no API); cert-packet auto-generation tied to loadout movements; Basis ERP sync if/when scoped. |

The ledger-first architecture means Phases 2 and 3 add **queries and views**, not schema rework -- because the underlying transactions are already captured.

---

## 7. Open Questions for Delmar / Validation

1. **District network reliability** -- how often do yards actually lose connectivity? Drives how aggressive offline-queue UX needs to be.
2. **Existing barcoding** -- are assets already labeled with anything machine-readable (QR, barcode, asset tag with serial)? If not, sticker rollout is a Phase 1 prerequisite.
3. **KPA API access** -- is there one, or is it screen-scrape only? Affects Phase 3 scope.
4. **Basis ERP** -- read-only sync, two-way, or out of scope? Drives whether we touch financial booking at all.
5. **Single-tenant install** -- is this Spartan-only, or do we want to architect for selling to similar small/mid oilfield ops (per Delmar's referral comment)? Affects multi-tenancy decisions in the schema.
