# Leonidas — Spartan Energy Services Asset Management

> Renamed from "Atlas" to "Leonidas" on 2026-04-07 by Delmar (Spartan CEO). GitHub repo slug retained as `spartan-atlas` for now.

- **GitHub:** https://github.com/asheehan923/spartan-atlas
- **Client:** Spartan Energy Services (Delmar, CEO)

## What This Is
Custom asset management replacing Excel for ~500 serialized valve/frac assets across 3 districts (Keithville LA, Midland TX, Pleasanton TX), 30+ simultaneous jobs, 3 ownership classes (owned, consignment, sub-rental).

## Design System
- **Spartan Red:** `#B32317` | **Gold:** `#C5A55A`
- **Backgrounds:** Engine Black `#0F1419`, Gunmetal `#1A1F2E`, Charcoal `#242B3A`
- **Fonts:** PT Serif (headings), PT Sans (body), Montserrat (data/numbers)
- **Style:** Industrial, bold, sharp corners — no rounded UI elements

## Key Domain Concepts
- **Shortage Resolution Waterfall** (must be consistent across ALL flows):
  1. Local owned → 2. Local consignment → 3. Local sub-rental → 4. Cross-district owned → 5. Cross-district consignment → 6. Cross-district sub-rental
- **Sub-assemblies:** Frac stacks = individual serialized components with mixed ownership
- **Consignment:** ~$7M in partner-owned equipment with per-partner terms and equity accrual
- **Smart serial verification:** Only triggered when check-in qty ≠ loadout qty

## Phasing
- **Phase 0 (current):** Styled HTML mockup (`index.html`, no build step)
- **Phase 1:** Equipment Registry + Job Planning Board + Mobile Check-In/Out
- **Phase 2:** Consignment Reporting + Forecasting + Machine Shop
- **Phase 3:** KPA Integration + Analytics + Advanced Scheduling

## Reference
- `PRD-leonidas.md` — Source of truth for all requirements (~1400 lines, 21 user flows)
