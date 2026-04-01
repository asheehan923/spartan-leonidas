# Atlas — Spartan Energy Services Asset Management System

## Repository
- **GitHub:** https://github.com/asheehan923/spartan-atlas
- **Client:** Spartan Energy Services (Delmar, CEO)
- **Author:** Alex Sheehan, VectisOS

## What This Is
Atlas is a custom asset management and job planning system replacing Spartan's Excel-based workflow for tracking ~500 serialized valve/frac assets across 3 districts (Keithville LA, Midland TX, Pleasanton TX), 30+ simultaneous jobs, and 3 ownership classes (owned, consignment, sub-rental).

## Project Files
- `index.html` — Single-file styled HTML mockup (Phase 0 prototype, no backend)
- `PRD-atlas.md` — Full product requirements document (~1100 lines)
- `Spartan-Energy-Services-V2.webp` — Spartan logo

## Design System
- **Spartan Red:** `#B32317` (primary actions, active states)
- **Gold:** `#C5A55A` (accents, highlights, data callouts)
- **Dark backgrounds:** Engine Black `#0F1419`, Gunmetal `#1A1F2E`, Charcoal `#242B3A`
- **Fonts:** PT Serif (headings), PT Sans (body), Montserrat (data/numbers)
- **Style:** Industrial, bold, sharp corners — no rounded UI elements

## Key Domain Concepts
- **Shortage Resolution Waterfall:** 6-tier priority for sourcing assets:
  1. Local owned → 2. Local consignment → 3. Local sub-rental → 4. Cross-district owned → 5. Cross-district consignment → 6. Cross-district sub-rental
- **Sub-assemblies:** Frac stacks composed of individual serialized components with mixed ownership
- **Consignment:** ~$7M in partner-owned equipment with per-partner terms and equity accrual
- **Smart serial verification:** Only triggered when check-in quantity ≠ loadout quantity

## Architecture (Target)
- **Frontend:** Next.js + Tailwind CSS
- **Backend:** Supabase (Postgres + Auth + Realtime)
- **Mobile:** PWA (field crews have good connectivity)

## Phasing
- **Phase 0 (current):** Styled mockup for stakeholder review
- **Phase 1:** F1 Equipment Registry + F2 Job Planning Board + Mobile Check-In/Out
- **Phase 2:** F4 Consignment Reporting + F6 Forecasting + F8 Machine Shop
- **Phase 3:** KPA Integration + Analytics + Advanced Scheduling

## 15 User Flows (UF1–UF15)
All documented in PRD-atlas.md with current-state and future-state descriptions.
Flows visible in the mockup under "User Flows" in the sidebar.

## Working With This Codebase
- The mockup is a single HTML file — open `index.html` directly in a browser
- All CSS and JS are inline (no build step, no dependencies)
- PRD is the source of truth for requirements — update it before changing the mockup
- The waterfall priority order must be consistent across ALL flows and screens

## Source Project
This repo is published from `VectisOS/Projects/Spartan/` in the main workspace.
PRD source: `C:\Users\ashee\Workspace\VectisOS\Projects\Spartan\PRD-atlas.md`
