# Leonidas - Backlog (Pre-Plan Parking Lot)

> **Purpose:** items identified before a formal project plan exists. Every entry here is a candidate for inclusion when we build `Project-Plan.md`. Tag `[PLAN]` means "must be scheduled in the project plan." Tag `[DECISION]` means "needs a decision before it can be scheduled."
>
> **How to use:** add items as they surface in conversation or PRD work. Don't delete items after they're included in the plan -- strike them through with `~~text~~` and add `→ see Project-Plan.md §X.Y` so we keep the audit trail.

---

## Build / Implementation

- **[PLAN]** Scaffold Supabase SQL migration files per `Supabase-Schema.md` §11 (0001_enums → 0011_seed). **Blocked on:** NDA executed + Delmar's real Excel handed over. (Added 2026-04-13)
- **[PLAN]** Port HTML prototype (`index.html`) to Next.js + Tailwind + shadcn per `Tech-Stack.md` §4.2. Estimate 2-3 days. **Blocked on:** prototype approved by Delmar. (Added 2026-04-13)
- **[PLAN]** Build `fn_post_movement` stored procedure + unit tests against seeded data. Tests must cover: idempotency, stale-from-location rejection, concurrent-writer race (pg_isolation_test), rate snapshotting. (Added 2026-04-13)
- **[PLAN]** Build ledger-to-state reconciliation `pg_cron` job (Supabase-Schema.md §10.3). (Added 2026-04-13)
- **[PLAN]** Mobile PWA: IndexedDB offline queue + idempotency-key wrapper for movement RPCs. (Added 2026-04-13)
- **[PLAN]** QR sticker rollout for all ~500 serialized assets (Phase 1 prerequisite per Tech-Stack §7 open questions). (Added 2026-04-13)

## Prototype / Demo

- **[PLAN]** Finish remaining prototype screens: Consignment Partner Dashboard, Monthly Statement (UF10), Shortage Waterfall (UF3), Capital Planning (UF11), Transfer Order (UF12), Sub-Rental Vendor (UF17). (Added 2026-04-13; task IDs 3-8 in session tracker)
- **[PLAN]** Add UF16-21 to the in-HTML flow renderer. (Added 2026-04-13; task ID 9)
- **[PLAN]** Wire demo-script happy-path click-throughs end-to-end. (Added 2026-04-13; task ID 10)
- **[PLAN]** Push prototype to GitHub Pages on renamed repo `spartan-leonidas`. (Added 2026-04-13; task ID 11)

## Data & Integrations

- **[DECISION]** KPA integration mode: read-through API, one-way sync, or Phase 3 deferral? Affects whether cert packets appear on loadout tickets from day one. (Tech-Stack §7.3, Supabase-Schema §15.5)
- **[DECISION]** Basis ERP sync: read-only, two-way, or out of scope entirely? Drives whether we touch any financial booking. (Tech-Stack §7.4)
- **[PLAN]** Build partner statement PDF generator (React-PDF or Puppeteer) fed by `v_asset_rental_days_monthly`. (Added 2026-04-13)
- **[PLAN]** Import historical utilization from Delmar's Excel as seed `asset_movements` rows so day-one reports show real trends. (Added 2026-04-13)

## Product Decisions Outstanding

- **[DECISION]** Standby equity accrual -- accrue on standby revenue or only active rental? (Supabase-Schema §15.1)
- **[DECISION]** Day-rate precedence -- job-level override vs. equipment-type default vs. partner-specific rate. (Supabase-Schema §15.2)
- **[DECISION]** Single-tenant or multi-tenant architecture. Delmar hinted at referring similar small/mid oilfield ops; if we productize, `tenant_id` must be added everywhere up-front. (Supabase-Schema §14, Tech-Stack §7.5)
- **[DECISION]** Partner portal -- do MAV and 3E get read-only logins to see their own utilization + statements live? (Would massively reduce month-end reporting friction but opens a support surface.)

## Operational / Commercial

- **[PLAN]** Have attorney-drafted NDA signed (Delmar driving). (From 2026-04-07 action items)
- **[PLAN]** Schedule onsite Lafayette visit in late April. (From 2026-03-31 action items)
- **[PLAN]** Interview Spartan's quality lead about KPA pain and cert/loadout data flow. (From 2026-03-31 action items)
- **[DECISION]** Commercial model -- one-time $10K build + monthly subscription? Subscription tier structure? (2026-04-07 transcript)
- **[PLAN]** Write a one-page "non-compete scope" note confirming Leonidas won't be sold to Spartan's direct competitors. Delmar flagged this explicitly. (2026-04-07 transcript)

## Research / Validation

- **[PLAN]** Validate offline-connectivity assumption -- how often do yards actually lose LTE? Drives mobile UX aggressiveness. (Tech-Stack §7.1)
- **[PLAN]** Audit existing asset labeling -- are serials already machine-readable anywhere, or is QR rollout greenfield? (Tech-Stack §7.2)
- **[PLAN]** Stress-test ledger pattern with simulated 30-concurrent-writer load (worst-case: every yard scanning simultaneously). (Added 2026-04-13)
