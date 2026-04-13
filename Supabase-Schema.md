# Leonidas - Supabase Schema Design

> Companion to `PRD-leonidas.md` and `Tech-Stack.md`. This document proposes the concrete Postgres/Supabase schema to support Phase 1 (Equipment Registry + Job Planning Board + Mobile Check-In/Out) while leaving the shape right for Phase 2 (Consignment Reporting + Equity) and Phase 3 (KPA + Advanced Scheduling).
>
> **Design principle:** append-only movement ledger is the source of truth; every other table is either reference data or derived state. See `Tech-Stack.md` §2 for the architectural rationale.

---

## 0. Conventions

- **Primary keys:** `bigint generated always as identity` for human-readable tables (jobs, assets, locations); `uuid` for high-churn append-only tables (movements, events) so mobile clients can generate IDs offline.
- **Timestamps:** every table gets `created_at timestamptz default now()` and `updated_at timestamptz` (via trigger). Ledger tables also carry a business-event timestamp distinct from the insert time.
- **Soft delete:** `deleted_at timestamptz` where deletion must be reversible (partners, customers). Hard-delete only for correctible mistakes on reference data.
- **Money:** `numeric(12,2)` for currency; never `float`.
- **Enums:** Postgres `CREATE TYPE ... AS ENUM`. Listed inline in each table below.
- **RLS:** every table has RLS enabled; policies defined in §9.
- **Naming:** snake_case, plural table names, singular column names, `_id` suffix for FKs.

---

## 1. Reference / Master Data

These tables change rarely and are read constantly.

### 1.1 `districts`
Physical Spartan operating locations (yards + satellites).

```sql
create table districts (
  id              bigint primary key generated always as identity,
  code            text unique not null,           -- 'KEI', 'MID', 'PLT'
  name            text not null,                  -- 'Keithville, LA'
  state           text not null,
  timezone        text not null default 'America/Chicago',
  address         text,
  geo_lat         numeric(9,6),
  geo_lng         numeric(9,6),
  is_active       boolean not null default true,
  created_at      timestamptz not null default now(),
  updated_at      timestamptz not null default now()
);
```

Seed: Keithville LA, Midland TX, Pleasanton TX.

### 1.2 `locations`
Finer-grained than districts. A yard has multiple locations (main yard, machine shop, staging, in-transit). A job site is also a location.

```sql
create type location_kind as enum (
  'yard', 'machine_shop', 'staging', 'in_transit',
  'job_site', 'partner_warehouse', 'scrap', 'adjustment'
);

create table locations (
  id              bigint primary key generated always as identity,
  district_id     bigint references districts(id),       -- null for job sites
  kind            location_kind not null,
  code            text unique not null,                  -- 'KEI-YARD', 'MID-SHOP', 'JOB-1847'
  name            text not null,
  job_id          bigint,                                -- fk added below; job_site locations link to jobs
  is_active       boolean not null default true,
  created_at      timestamptz not null default now(),
  updated_at      timestamptz not null default now()
);
create index on locations(district_id, kind);
```

**Why kind matters:** the movement ledger uses from/to location_id for everything. "In transit" is its own location so we can query assets currently on trucks. "Adjustment" is a sentinel location used for cycle-count corrections so the ledger balances.

### 1.3 `equipment_types`
The catalog. From PRD §1.4 taxonomy.

```sql
create type ownership_class as enum ('owned', 'consignment', 'sub_rental');

create table equipment_types (
  id                  bigint primary key generated always as identity,
  code                text unique not null,               -- 'WNG-4-15-HYD'
  category            text not null,                     -- 'wing_valve', 'master_valve', 'spool', 'tee', 'cross', 'accumulator', 'javelin'
  size                text,                              -- '4"', '5-1/8"', '7-1/16"'
  pressure_rating     text,                              -- '5K', '10K', '15K'
  actuation           text,                              -- 'hydraulic', 'manual'
  brand               text,                              -- 'MAV', '3E', 'Spartan'
  description         text,
  default_day_rate    numeric(10,2),
  is_active           boolean not null default true,
  created_at          timestamptz not null default now(),
  updated_at          timestamptz not null default now()
);
create unique index on equipment_types(category, size, pressure_rating, actuation, brand);
```

### 1.4 `sub_assembly_definitions`
Bill-of-materials for frac stacks and similar. Composite assets made of serialized components.

```sql
create table sub_assembly_definitions (
  id              bigint primary key generated always as identity,
  code            text unique not null,         -- 'FRAC-STACK-7-15K'
  name            text not null,                -- '7-1/16" 15K Frac Stack'
  description     text,
  created_at      timestamptz not null default now(),
  updated_at      timestamptz not null default now()
);

create table sub_assembly_bom (
  sub_assembly_definition_id   bigint references sub_assembly_definitions(id) on delete cascade,
  equipment_type_id            bigint references equipment_types(id),
  qty_required                 int not null check (qty_required > 0),
  position_label               text,           -- 'top master', 'wing', 'crown valve'
  primary key (sub_assembly_definition_id, position_label)
);
```

### 1.5 `customers`
End customers Spartan bills (BPX, Vital, Civitas, etc.).

```sql
create table customers (
  id              bigint primary key generated always as identity,
  code            text unique not null,          -- 'BPX', 'VITAL', 'CIVITAS', 'XO'
  name            text not null,
  billing_terms   text,
  is_active       boolean not null default true,
  deleted_at      timestamptz,
  created_at      timestamptz not null default now(),
  updated_at      timestamptz not null default now()
);
```

### 1.6 `consignment_partners`
MAV, 3E, plus any future partners. Holds the equity/revshare terms referenced in PRD §1.5.1.

```sql
create table consignment_partners (
  id                      bigint primary key generated always as identity,
  code                    text unique not null,             -- 'MAV', '3E'
  name                    text not null,
  serial_prefix           text unique not null,             -- 'MAV-', 'BWV-' (Best Way is sub-rental but same pattern)
  revshare_pct            numeric(5,4) not null,            -- 1.00 means partner gets 100% of rental revenue (Spartan earns the margin on top)
  equity_rate             numeric(5,4) not null,            -- 0.2500 for MAV, 0.3300 for 3E
  standby_day_rate_pct    numeric(5,4) default 0.50,        -- standby = 50% of day rate (configurable)
  contract_start          date,
  contract_end            date,
  contact_name            text,
  contact_email           text,
  contact_phone           text,
  is_active               boolean not null default true,
  deleted_at              timestamptz,
  created_at              timestamptz not null default now(),
  updated_at              timestamptz not null default now()
);
```

### 1.7 `sub_rental_vendors`
Best Way and other ad-hoc rental sources. Distinct from consignment partners because no equity accrues.

```sql
create table sub_rental_vendors (
  id                  bigint primary key generated always as identity,
  code                text unique not null,
  name                text not null,
  typical_margin_pct  numeric(5,4),                -- Delmar: Best Way ~5%
  contact_name        text,
  contact_email       text,
  contact_phone       text,
  is_active           boolean not null default true,
  deleted_at          timestamptz,
  created_at          timestamptz not null default now(),
  updated_at          timestamptz not null default now()
);
```

---

## 2. Assets (the Serialized Things)

### 2.1 `assets`
One row per serialized piece of equipment -- owned, consigned, or sub-rented. The identity and immutable facts live here.

```sql
create table assets (
  id                      bigint primary key generated always as identity,
  serial_number           text unique not null,             -- 'MAV-1042', 'SPT-00314', 'BWV-772'
  asset_number            text,                             -- internal Spartan asset tag if different from serial
  equipment_type_id       bigint not null references equipment_types(id),
  ownership_class         ownership_class not null,
  consignment_partner_id  bigint references consignment_partners(id),       -- required if ownership_class='consignment'
  sub_rental_vendor_id    bigint references sub_rental_vendors(id),         -- required if ownership_class='sub_rental'
  agreed_purchase_price   numeric(12,2),                    -- for consigned assets; the price-down target
  acquisition_cost        numeric(12,2),                    -- for owned
  intake_date             date,
  return_date             date,                             -- set when asset is returned to partner or scrapped
  status                  text not null default 'active',   -- 'active', 'returned', 'scrapped'
  notes                   text,
  created_at              timestamptz not null default now(),
  updated_at              timestamptz not null default now(),
  constraint ck_ownership_link check (
    (ownership_class = 'consignment' and consignment_partner_id is not null)
    or (ownership_class = 'sub_rental' and sub_rental_vendor_id is not null)
    or (ownership_class = 'owned' and consignment_partner_id is null and sub_rental_vendor_id is null)
  )
);
create index on assets(equipment_type_id);
create index on assets(ownership_class);
create index on assets(consignment_partner_id);
```

### 2.2 `asset_state` (derived)
Current position and status of every asset. **Never written by application code.** Maintained by the `fn_post_movement` stored procedure in the same transaction as the movement insert. Queryable as a normal table for fast reads.

```sql
create type asset_status as enum (
  'in_yard', 'staged', 'in_transit', 'on_job', 'in_shop',
  'returned_to_partner', 'scrapped'
);

create table asset_state (
  asset_id                bigint primary key references assets(id) on delete cascade,
  current_location_id     bigint references locations(id),
  current_job_id          bigint,                           -- fk added below
  current_sub_assembly_instance_id  bigint,                 -- fk added below, nullable
  status                  asset_status not null default 'in_yard',
  last_movement_id        uuid,                             -- fk to asset_movements
  last_movement_ts        timestamptz,
  version                 bigint not null default 0,        -- optimistic lock token
  updated_at              timestamptz not null default now()
);
create index on asset_state(current_location_id);
create index on asset_state(current_job_id);
create index on asset_state(status);
```

Rebuilding `asset_state` from the ledger is always possible (and should be a tested procedure) -- this is how we prove the ledger is canonical.

---

## 3. Jobs & Work Orders

### 3.1 `jobs`
A job = a customer work order. Ryan's XO job; BPX Permian Pad 7; Vital Wolfcamp A-9H.

```sql
create type job_status as enum ('draft', 'scheduled', 'active', 'closing', 'closed', 'cancelled');

create table jobs (
  id                  bigint primary key generated always as identity,
  job_number          text unique not null,         -- 'JOB-1847'
  customer_id         bigint not null references customers(id),
  well_name           text,                         -- 'Wolfcamp A-9H Pad'
  basin               text,                         -- 'Permian', 'Haynesville'
  district_id         bigint not null references districts(id),  -- originating district
  job_site_location_id bigint references locations(id),
  scheduled_start     date,
  scheduled_end       date,
  actual_start        timestamptz,
  actual_end          timestamptz,
  status              job_status not null default 'draft',
  day_rate_override   numeric(10,2),                -- if this job uses special pricing
  notes               text,
  created_by          uuid references auth.users(id),
  created_at          timestamptz not null default now(),
  updated_at          timestamptz not null default now()
);
create index on jobs(customer_id);
create index on jobs(district_id, status);
create index on jobs(scheduled_start);
```

Backfill FK now that jobs exists:
```sql
alter table locations add constraint fk_locations_job foreign key (job_id) references jobs(id);
alter table asset_state add constraint fk_asset_state_job foreign key (current_job_id) references jobs(id);
```

### 3.2 `job_equipment_requirements`
What the job *needs* -- by equipment type and qty. Distinct from what's actually assigned (assignments happen at the serial level via movements).

```sql
create table job_equipment_requirements (
  id                  bigint primary key generated always as identity,
  job_id              bigint not null references jobs(id) on delete cascade,
  equipment_type_id   bigint not null references equipment_types(id),
  qty_required        int not null check (qty_required > 0),
  qty_filled          int not null default 0,
  day_rate            numeric(10,2),                -- snapshotted from equipment_types.default_day_rate at creation
  notes               text,
  created_at          timestamptz not null default now(),
  updated_at          timestamptz not null default now(),
  unique (job_id, equipment_type_id)
);
```

`qty_filled` is derived (count of assets on job for that type) -- maintained by `fn_post_movement`, recomputable from the ledger.

---

## 4. Sub-Assembly Instances

A specific frac stack in existence right now.

```sql
create type sub_assembly_status as enum ('built', 'deployed', 'broken_down');

create table sub_assembly_instances (
  id                          bigint primary key generated always as identity,
  definition_id               bigint not null references sub_assembly_definitions(id),
  instance_code               text unique not null,           -- 'STK-003'
  status                      sub_assembly_status not null default 'built',
  built_at                    timestamptz not null default now(),
  built_by                    uuid references auth.users(id),
  broken_down_at              timestamptz,
  current_location_id         bigint references locations(id),
  current_job_id              bigint references jobs(id),
  notes                       text,
  created_at                  timestamptz not null default now(),
  updated_at                  timestamptz not null default now()
);

alter table asset_state add constraint fk_asset_state_sub_asm
  foreign key (current_sub_assembly_instance_id) references sub_assembly_instances(id);
```

Composition (which serials belong to which instance) is not a separate table -- it's derived from `asset_state.current_sub_assembly_instance_id`. If you break down a stack, the movement "unlinks" each asset. Audit trail stays in `asset_movements`.

---

## 5. The Movement Ledger (THE core table)

### 5.1 `asset_movements`
Append-only. One row per physical event that changes where/how an asset exists. **Never UPDATE. Never DELETE. Corrections are new reversing entries.**

```sql
create type movement_type as enum (
  'loadout',            -- yard → job site
  'checkin',            -- job site → yard
  'transfer_out',       -- yard A → in_transit (cross-district)
  'transfer_in',        -- in_transit → yard B
  'intake_owned',       -- appearance: new owned asset purchased
  'intake_consignment', -- appearance: consignment partner delivered
  'return_to_partner',  -- disappearance: consignment returned
  'scrap',              -- disappearance: destroyed/sold
  'sub_build',          -- assigned into a sub-assembly instance
  'sub_break',          -- removed from a sub-assembly instance
  'shop_in',            -- yard → machine shop
  'shop_out',           -- machine shop → yard
  'adjustment'          -- cycle-count correction (always paired with 'adjustment' location)
);

create table asset_movements (
  id                      uuid primary key default gen_random_uuid(),
  ts                      timestamptz not null,                    -- when it physically happened
  logged_at               timestamptz not null default now(),      -- when it was entered
  asset_id                bigint not null references assets(id),
  movement_type           movement_type not null,
  from_location_id        bigint references locations(id),         -- null for appearance events
  to_location_id          bigint references locations(id),         -- null for disappearance events
  job_id                  bigint references jobs(id),
  sub_assembly_instance_id bigint references sub_assembly_instances(id),
  day_rate_snapshot       numeric(10,2),                           -- rate in effect at time of movement (critical for billing)
  equity_rate_snapshot    numeric(5,4),                            -- equity rate in effect (critical for consignment accrual)
  ticket_id               bigint,                                   -- fk to loadout_tickets (added below)
  created_by              uuid not null references auth.users(id),
  device_id               text,                                     -- scanner/phone identifier
  geo_lat                 numeric(9,6),
  geo_lng                 numeric(9,6),
  idempotency_key         text unique,                              -- mobile-generated; safe retry
  reversal_of             uuid references asset_movements(id),      -- non-null if this entry reverses another
  notes                   text
);
create index on asset_movements(asset_id, ts desc);
create index on asset_movements(job_id);
create index on asset_movements(ts);
create index on asset_movements(movement_type, ts desc);
```

### 5.2 The `fn_post_movement` procedure
The **only** writer allowed on `asset_movements` + `asset_state`. Clients call this via Supabase RPC.

```sql
create or replace function fn_post_movement(
  p_asset_id              bigint,
  p_movement_type         movement_type,
  p_ts                    timestamptz,
  p_from_location_id      bigint,
  p_to_location_id        bigint,
  p_job_id                bigint,
  p_sub_assembly_instance_id bigint,
  p_ticket_id             bigint,
  p_geo_lat               numeric,
  p_geo_lng               numeric,
  p_device_id             text,
  p_idempotency_key       text,
  p_notes                 text
) returns uuid
language plpgsql
security definer
as $$
declare
  v_current_state    asset_state%rowtype;
  v_movement_id      uuid;
  v_day_rate         numeric(10,2);
  v_equity_rate      numeric(5,4);
  v_new_status       asset_status;
  v_new_location     bigint;
begin
  -- 1. Idempotency short-circuit
  if p_idempotency_key is not null then
    select id into v_movement_id from asset_movements where idempotency_key = p_idempotency_key;
    if found then return v_movement_id; end if;
  end if;

  -- 2. Lock the asset row (pessimistic)
  select * into v_current_state from asset_state where asset_id = p_asset_id for update;
  if not found then
    raise exception 'Asset % has no state row', p_asset_id;
  end if;

  -- 3. Precondition: from_location must match current location (except for appearance events)
  if p_movement_type not in ('intake_owned', 'intake_consignment', 'adjustment') then
    if v_current_state.current_location_id is distinct from p_from_location_id then
      raise exception 'Asset % is at location %, not % (stale movement)',
        p_asset_id, v_current_state.current_location_id, p_from_location_id;
    end if;
  end if;

  -- 4. Snapshot rates
  select er.default_day_rate, cp.equity_rate
    into v_day_rate, v_equity_rate
    from assets a
    join equipment_types er on er.id = a.equipment_type_id
    left join consignment_partners cp on cp.id = a.consignment_partner_id
    where a.id = p_asset_id;

  -- 5. Insert movement (immutable)
  insert into asset_movements (
    ts, asset_id, movement_type, from_location_id, to_location_id,
    job_id, sub_assembly_instance_id, day_rate_snapshot, equity_rate_snapshot,
    ticket_id, created_by, device_id, geo_lat, geo_lng, idempotency_key, notes
  ) values (
    p_ts, p_asset_id, p_movement_type, p_from_location_id, p_to_location_id,
    p_job_id, p_sub_assembly_instance_id, v_day_rate, v_equity_rate,
    p_ticket_id, auth.uid(), p_device_id, p_geo_lat, p_geo_lng, p_idempotency_key, p_notes
  ) returning id into v_movement_id;

  -- 6. Compute new derived state
  v_new_location := coalesce(p_to_location_id, v_current_state.current_location_id);
  v_new_status := case p_movement_type
    when 'loadout'             then 'on_job'::asset_status
    when 'checkin'             then 'in_yard'::asset_status
    when 'transfer_out'        then 'in_transit'::asset_status
    when 'transfer_in'         then 'in_yard'::asset_status
    when 'shop_in'             then 'in_shop'::asset_status
    when 'shop_out'            then 'in_yard'::asset_status
    when 'return_to_partner'   then 'returned_to_partner'::asset_status
    when 'scrap'               then 'scrapped'::asset_status
    when 'intake_owned'        then 'in_yard'::asset_status
    when 'intake_consignment'  then 'in_yard'::asset_status
    else v_current_state.status
  end;

  -- 7. Update derived state (same transaction)
  update asset_state set
    current_location_id = v_new_location,
    current_job_id = case when p_movement_type = 'loadout' then p_job_id
                          when p_movement_type = 'checkin' then null
                          else current_job_id end,
    current_sub_assembly_instance_id = case
      when p_movement_type = 'sub_build' then p_sub_assembly_instance_id
      when p_movement_type = 'sub_break' then null
      else current_sub_assembly_instance_id end,
    status = v_new_status,
    last_movement_id = v_movement_id,
    last_movement_ts = p_ts,
    version = version + 1,
    updated_at = now()
  where asset_id = p_asset_id;

  -- 8. Update job fill counts if applicable
  if p_job_id is not null then
    -- recompute qty_filled for the relevant requirement row
    update job_equipment_requirements jer set qty_filled = (
      select count(*) from asset_state ast
      join assets a on a.id = ast.asset_id
      where ast.current_job_id = p_job_id
        and a.equipment_type_id = jer.equipment_type_id
    ) where jer.job_id = p_job_id
      and jer.equipment_type_id = (select equipment_type_id from assets where id = p_asset_id);
  end if;

  return v_movement_id;
end;
$$;
```

Grant `execute` only to the `authenticated` role; revoke direct `insert/update` on `asset_movements` and `asset_state` from everyone.

---

## 6. Tickets (Paperwork that Groups Movements)

A loadout ticket = "truck left the yard with these 12 serials for this job." Check-in ticket = the reverse. Each ticket is the header; the movements it created are the detail.

```sql
create type ticket_type as enum ('loadout', 'checkin', 'transfer');
create type ticket_status as enum ('draft', 'posted', 'voided');

create table tickets (
  id                  bigint primary key generated always as identity,
  ticket_number       text unique not null,           -- 'LO-2026-04-1047'
  ticket_type         ticket_type not null,
  status              ticket_status not null default 'draft',
  job_id              bigint references jobs(id),
  from_location_id    bigint references locations(id),
  to_location_id      bigint references locations(id),
  posted_at           timestamptz,
  posted_by           uuid references auth.users(id),
  truck_number        text,
  driver_name         text,
  notes               text,
  created_by          uuid references auth.users(id),
  created_at          timestamptz not null default now(),
  updated_at          timestamptz not null default now()
);

alter table asset_movements add constraint fk_movements_ticket
  foreign key (ticket_id) references tickets(id);
```

A ticket is posted atomically: one transaction calls `fn_post_movement` for each asset on it, plus flips ticket status to `'posted'`. Discrepancies on check-in (asset on ticket not scanned, or vice versa) → UF8's reconciliation flow, which creates `adjustment` movements rather than silently editing.

---

## 7. Views & Derived Queries

These are the money-makers for the UI. All derivable from the ledger + state.

### 7.1 `v_asset_current` -- one-stop current-state view
```sql
create view v_asset_current as
select
  a.id, a.serial_number, a.asset_number,
  et.category, et.size, et.pressure_rating, et.actuation, et.brand,
  a.ownership_class,
  cp.code as partner_code,
  ast.status, ast.current_location_id,
  loc.code as location_code, loc.name as location_name,
  d.code as district_code,
  ast.current_job_id,
  j.job_number, j.well_name,
  ast.current_sub_assembly_instance_id,
  ast.last_movement_ts
from assets a
join equipment_types et on et.id = a.equipment_type_id
left join consignment_partners cp on cp.id = a.consignment_partner_id
join asset_state ast on ast.asset_id = a.id
left join locations loc on loc.id = ast.current_location_id
left join districts d on d.id = loc.district_id
left join jobs j on j.id = ast.current_job_id;
```

### 7.2 `v_district_inventory` -- for the planning board
```sql
create view v_district_inventory as
select
  d.code as district_code,
  et.category, et.size, et.pressure_rating,
  a.ownership_class,
  count(*) filter (where ast.status = 'in_yard')  as in_yard,
  count(*) filter (where ast.status = 'on_job')   as on_job,
  count(*) filter (where ast.status = 'in_transit') as in_transit,
  count(*) filter (where ast.status = 'in_shop') as in_shop,
  count(*) as total
from districts d
join locations loc on loc.district_id = d.id
join asset_state ast on ast.current_location_id = loc.id
join assets a on a.id = ast.asset_id
join equipment_types et on et.id = a.equipment_type_id
group by d.code, et.category, et.size, et.pressure_rating, a.ownership_class;
```

### 7.3 `v_asset_rental_days_monthly` -- the basis of equity accrual and partner statements
```sql
create view v_asset_rental_days_monthly as
with day_series as (
  -- one row per asset per calendar day it was on a job
  select
    m.asset_id,
    date_trunc('day', gs)::date as rental_date,
    m.day_rate_snapshot,
    m.equity_rate_snapshot,
    m.job_id
  from asset_movements m
  join asset_movements m2 on m2.asset_id = m.asset_id
    and m2.movement_type = 'checkin'
    and m2.ts > m.ts
  cross join lateral generate_series(m.ts, m2.ts - interval '1 day', interval '1 day') gs
  where m.movement_type = 'loadout'
)
select
  asset_id,
  date_trunc('month', rental_date)::date as statement_month,
  job_id,
  count(*) as days_on_job,
  max(day_rate_snapshot) as day_rate,
  sum(day_rate_snapshot) as gross_revenue,
  sum(day_rate_snapshot * coalesce(equity_rate_snapshot, 0)) as equity_accrued
from day_series
group by asset_id, statement_month, job_id;
```

This view is what drives: UF10 monthly consignment statement, UF11 capital-planning utilization flags, partner equity dashboards, customer invoicing.

### 7.4 `v_partner_equity_balance`
```sql
create view v_partner_equity_balance as
select
  cp.id as partner_id, cp.code,
  a.id as asset_id, a.serial_number,
  a.agreed_purchase_price,
  coalesce(sum(m.day_rate_snapshot * m.equity_rate_snapshot), 0) as equity_accrued_to_date,
  a.agreed_purchase_price - coalesce(sum(m.day_rate_snapshot * m.equity_rate_snapshot), 0) as purchase_down_remaining
from consignment_partners cp
join assets a on a.consignment_partner_id = cp.id
left join asset_movements m on m.asset_id = a.id
  and m.movement_type = 'loadout'  -- only count rental days not standby for MVP; refine later
group by cp.id, cp.code, a.id, a.serial_number, a.agreed_purchase_price;
```

### 7.5 `v_equipment_type_utilization` -- UF11 capital planning
```sql
create view v_equipment_type_utilization as
select
  et.id, et.category, et.size, et.pressure_rating,
  date_trunc('month', m.ts)::date as util_month,
  count(distinct a.id) as fleet_size,
  count(distinct m.asset_id) filter (where m.movement_type = 'loadout') as unique_deployments,
  sum(case when m.movement_type = 'loadout' then 1 else 0 end)::numeric
    / nullif(count(distinct a.id), 0) as deployment_rate
from equipment_types et
join assets a on a.equipment_type_id = et.id
left join asset_movements m on m.asset_id = a.id
group by et.id, et.category, et.size, et.pressure_rating, util_month;
```

---

## 8. Auth & Users

Supabase provides `auth.users`. Extend with a profile table for role and district scoping.

```sql
create type user_role as enum ('admin', 'ceo', 'president', 'district_manager', 'yard_supervisor', 'field_crew', 'machine_shop', 'partner_viewer');

create table user_profiles (
  user_id         uuid primary key references auth.users(id) on delete cascade,
  role            user_role not null,
  district_id     bigint references districts(id),       -- scope for DMs and yard crews
  partner_id      bigint references consignment_partners(id),  -- scope for partner_viewer (future)
  display_name    text not null,
  phone           text,
  is_active       boolean not null default true,
  created_at      timestamptz not null default now(),
  updated_at      timestamptz not null default now()
);
```

---

## 9. Row Level Security (RLS) Policies

Headline:
- Admin / CEO / President: full read everywhere.
- District Manager: read everything in their district; write movements only for their district.
- Yard Supervisor / Field Crew: read their district; post movements via RPC (RPC is `security definer`, so policy on `asset_movements` only needs to allow read).
- Partner Viewer (future Phase 2+): read only their own partner's assets, movements, and statements.

```sql
alter table assets enable row level security;
alter table asset_state enable row level security;
alter table asset_movements enable row level security;
alter table jobs enable row level security;
alter table tickets enable row level security;
-- ...repeat for all tables

-- Example: movements readable by staff in the same district; partner viewers see only their partner
create policy read_movements_staff on asset_movements for select using (
  exists (
    select 1 from user_profiles up
    join assets a on a.id = asset_movements.asset_id
    join locations l on l.id = coalesce(asset_movements.from_location_id, asset_movements.to_location_id)
    where up.user_id = auth.uid()
      and up.is_active
      and (
        up.role in ('admin', 'ceo', 'president')
        or (up.role in ('district_manager', 'yard_supervisor', 'field_crew') and up.district_id = l.district_id)
        or (up.role = 'partner_viewer' and up.partner_id = a.consignment_partner_id)
      )
  )
);

-- Movements are only written via RPC; revoke direct insert/update from authenticated
revoke insert, update, delete on asset_movements from authenticated;
revoke insert, update, delete on asset_state from authenticated;
grant execute on function fn_post_movement(...) to authenticated;
```

Similar policies for other tables -- pattern is "join to user_profiles, check role + district/partner scope."

---

## 10. Audit & Integrity Helpers

### 10.1 Reversal enforcement
A trigger prevents updating or deleting any row in `asset_movements`. Corrections must be new rows with `reversal_of` set.

```sql
create or replace function fn_ledger_no_mutate() returns trigger as $$
begin
  raise exception 'Ledger tables are append-only; use a reversing entry instead';
end;
$$ language plpgsql;

create trigger trg_asset_movements_no_update before update or delete on asset_movements
  for each row execute function fn_ledger_no_mutate();
```

### 10.2 Cycle-count reconciliation
When a physical count reveals drift (asset X is actually at location Y, system thinks it's at Z), post an `adjustment` movement from the sentinel `adjustment` location. Required `notes` field explaining why. Surfaces in a report so Delmar can see adjustment volume per district over time -- early warning for yard discipline issues.

### 10.3 Ledger-to-state reconciliation job
A nightly `pg_cron` job rebuilds a shadow copy of `asset_state` from scratch from the ledger and diffs it against the live table. Any mismatches page us -- this is how we prove the trigger logic never silently drifts.

---

## 11. Migration Order (for Supabase CLI)

Roughly one migration per section, in order:
1. `0001_enums.sql` -- all `CREATE TYPE` statements
2. `0002_reference_data.sql` -- districts, locations, equipment_types, customers, partners, vendors, sub_assembly definitions
3. `0003_assets.sql` -- assets + asset_state
4. `0004_jobs.sql` -- jobs, job_equipment_requirements
5. `0005_sub_assemblies.sql` -- instances + FK additions
6. `0006_ledger.sql` -- asset_movements + tickets + `fn_post_movement`
7. `0007_views.sql` -- all v_* views
8. `0008_auth.sql` -- user_profiles
9. `0009_rls.sql` -- all policies
10. `0010_integrity.sql` -- append-only trigger, reconciliation job
11. `0011_seed.sql` -- districts, equipment types from PRD §1.4, MAV + 3E partners

---

## 12. Storage Buckets

- `cert-packets/` -- KPA cert PDFs, namespaced by asset serial. Phase 1 upload, Phase 3 auto-generate.
- `partner-statements/` -- monthly PDF statements generated from `v_asset_rental_days_monthly`.
- `asset-photos/` -- optional photos attached to movements (e.g., damage on check-in).

RLS on storage mirrors the table policies (scoped by district or partner).

---

## 13. Indexes & Performance Notes (for Phase 1 scale)

500 assets, ~30 jobs, maybe 100-300 movements/day. None of these need tuning yet, but for awareness:

- `asset_movements(asset_id, ts desc)` -- the hottest index; per-asset history queries
- `asset_movements(job_id)` -- for job close-out and monthly statement
- `asset_movements(ts)` -- for the realtime planning-board feed
- `asset_state(current_location_id)` -- yard view
- `asset_state(status)` -- "everything in transit right now"

At Phase 3 scale (if Spartan grows 5-10x), consider: partitioning `asset_movements` by month, materializing `v_asset_rental_days_monthly` and refreshing nightly, BRIN index on `asset_movements.ts`.

---

## 14. What's Deliberately *Not* in This Schema

- **No financial GL postings.** We track rental revenue and equity accrual but do not post journal entries to a real accounting system. Basis ERP integration (Phase 3) is where that lives.
- **No purchase-order / invoice generation tables.** Monthly consignment statements are generated from views and stored as PDFs; actual partner payment runs through Delmar's existing AP process.
- **No geofencing or RFID.** Deferred per Tech-Stack §5.2.
- **No multi-tenancy.** Single-tenant for Spartan. If we productize later (per Delmar's referral comment), we add `tenant_id` everywhere and a tenant-scoped RLS layer -- doable but a deliberate re-cut.

---

## 15. Open Questions Before Implementation

1. **Standby equity accrual** -- does equity accrue on standby revenue or only active rental? Transcripts suggest both ("if I go on standby, he goes on standby"). Confirm with Delmar; `v_asset_rental_days_monthly` currently only counts active days.
2. **Day-rate override precedence** -- job-level override vs. equipment-type default vs. partner-specific rate. Need Delmar to rank these.
3. **Sub-assembly inventory semantics** -- when a stack is "deployed," do we treat each component's `current_job_id` as set, or only the instance's? (Current design: both, synced via movements.)
4. **Partner serial prefixes** -- is every MAV asset prefixed `MAV-`? If partners ever reassign serials, we need a historical `serial_aliases` table.
5. **KPA data source** -- read-through API, one-way sync into a `cert_records` table, or out of scope for Phase 1? Affects whether certs appear on loadout tickets from day one.
