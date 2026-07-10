# Handoff: PCE App — McCoy's Store Development Estimating Tool

## Overview
A field estimating tool for McCoy's Building Supply's Store Development department. Estimators build project cost estimates on-site (phone) with photos per line item, then review a CapEx depreciation/tax roll-up and generate a customer/internal approval proposal (desktop or phone). Modeled directly on McCoy's existing "Project Cost Estimate" master form.

## About the Design Files
The files in this bundle (`EstimatorApp.dc.html`, `PCE App.dc.html`, and the standalone bundle) are **design references built in HTML/JS** — working prototypes showing intended look, data model, and interaction, not production code to copy directly. The task is to **recreate this design as a real application** with a proper backend — persistent database, user accounts, multi-device sync, and real photo storage — using whatever stack fits the team (e.g., React Native or a responsive web app + a backend like Postgres/Node, Firebase, or similar). Choose the most appropriate framework; there is no existing app codebase to conform to.

## Fidelity
**High-fidelity.** Colors, typography, spacing, copy, and the full data model (accounting categories, depreciation math, CSI divisions) are final/real, taken from McCoy's brand and their master estimate form. Recreate pixel-close using the target stack's component libraries.

## Data model (critical — this is the core business logic)

### Project
- `id`, `projectNo` (e.g. "PCE-2614"), `name`, `store` (e.g. "Store #26"), `city`, `estimator`, `taxRate` (%, e.g. 8.25), `status` (Draft / In Progress / Bid Sent / Approved), `schedule` (date range text), `duration` (working days), `approved` (bool)
- `areas`: list of location labels used to group line items (e.g. "Front End", "Lumber Yard", "Site", "Mechanical") — user-defined per project
- `items`: list of Line Items (below)

### Line Item
- `desc` (text), `area` (one of project's areas), `div` (CSI division code, 2-digit string, e.g. "09"), `acct` (accounting category key, see below), `qty` (number), `unit` (LS/SF/LF/EA/SY/CY/HR), `cost` (unit cost, currency), `amount` = qty × cost, `hasPhoto` (bool) + photo asset reference (design uses placeholder patterns; production needs real image upload/storage per item)

### Accounting Categories (drives depreciation — must match exactly)
| Key | Code | Name | Life (years) |
|---|---|---|---|
| A1810 | A1810 | Building | 39.5 |
| B1810 | B1810 | Building | 15 |
| C1810 | C1810 | Building | 7 |
| C1830 | C1830 | WSRT | 7 |
| D1830 | D1830 | WSRT | 5 |
| C1950 | C1950 | FFE | 7 |
| D1950 | D1950 | FFE | 5 |
| EXP | EXP | Expenses | 0 (not depreciated) |
| ADD | ADD | Added Item | 0 (not depreciated) |

**Depreciation method: straight-line.** For each category with life > 0: `annualDepreciation = sum(amount for items in category) / life`. Categories with life 0 are expensed, not capitalized.

### CSI Divisions
Standard 2-digit CSI MasterFormat codes (01 General Requirements … 33 Utilities — full list in `EstimatorApp.dc.html`'s `renderVals()`/constructor). Line items roll up into 4 report groups: General (≤01), Construction/Assemblies/Finishes (02–14), M.E.P. Services (21–28), Site (31+).

## Screens / Views
1. **Projects (dashboard)** — card grid of projects: store, city, status pill, cover photo, photo count, total. Header stats: active project count, total estimated value, Yr-1 depreciation across all projects.
2. **Estimate** — line items grouped by Area, each showing accounting-category chip + CSI division + qty/unit/cost + amount. Chips summarizing capitalized basis per accounting category at top. "Add line item" opens a modal: photo capture, description, area, CSI division, accounting category (dropdown, sets depreciation life), qty/unit/cost, live-computed amount.
3. **Photos** — gallery grouped by Area, one tile per line item with a photo; tap to enlarge (lightbox).
4. **Cost Summary** — project info (project #, estimator, tax rate, schedule, duration); Cost Summary ledger (Subtotal → Sales Tax → Total); CapEx Depreciation Summary (stacked bar + per-category basis and annual $/yr); CSI Division Summary (grouped, matches master form layout). Buttons: preview proposal, export PDF.
5. **Proposal / Approval** — branded document: logo, project #, scope of work by area, cost ledger, total, two signature lines (Operations / Finance — currently labeled "Carlos" and "Art" as placeholders, should be dynamic approver names/roles in production), Approve button that flips project status to Approved.

## Interactions & Behavior
- Bottom tab bar (mobile) / left sidebar (desktop) nav between Projects / Estimate / Photos / Cost Summary — same underlying view state.
- Floating "+" action button on Estimate (mobile) opens the add-item modal; desktop uses a header button instead.
- All monetary values recompute live as line items are added — no manual recalculation.
- Approve action is currently local/in-memory only — production needs this to write to a real backend, probably trigger a notification/email to approvers, and lock the estimate from further edits once approved (or version it).
- Photo capture in the prototype is a stand-in for real camera/gallery upload (currently paints a placeholder stripe pattern); production must let a real photo be taken/selected and stored per line item.

## State Management
- Per-project: list of line items, approval status.
- Global: which project is open, which view/tab is active, add-item form draft state, lightbox open/closed.
- **Production must add**: authenticated user/estimator identity, persistent storage per project (not in-memory), photo upload to real storage (e.g. S3/Cloud Storage) with references saved per line item, multi-device sync, and an audit trail for approvals (who approved, when — accounting needs this for capitalization records).

## Design Tokens
- Colors: McCoy's green `#123f22` (primary/dark), `#1a6e3a` / `#1d5a32` (mid greens for actions/nav), McCoy's gold `#f4c413` (accent/CTA), background `#edf0ea` / `#e7e5df`, card borders `#e2e7df`, body text `#16241b`, secondary text `#5b6b5e`/`#7a877c`.
- Status pill colors: In Progress (amber `#fdeec2`/`#9a7b0a`), Bid Sent (green `#dcefe1`/`#1a6e3a`), Draft (gray `#eaeee8`/`#697567`), Approved (green `#d3edda`/`#15803d`).
- Typography: **Archivo** (400–900 weights) for UI text, **Spline Sans Mono** for all numeric/currency/quantity values. Base UI text ~13–15px, headers 18–26px bold/black weight, monospace amounts 14–30px depending on prominence.
- Border radius: cards/panels 14–18px, buttons 10–13px, chips 5–7px, photo tiles 10–12px.
- Shadows: subtle, e.g. `0 1px 2px rgba(18,63,34,.05)` on cards, `0 8px 30px rgba(18,63,34,.08)` on the proposal document.

## Assets
- `assets/pce-icon.png` — PCE App icon (user-provided, AI-generated), used as the app icon/logo mark throughout.
- `assets/mccoys-logo.png` — McCoy's Building Supply official logo (user-provided), used on the Proposal document header.
- Photo placeholders: diagonal-stripe CSS gradients standing in for real site photos — replace entirely with real image upload/display.

## Files
- `EstimatorApp.dc.html` — the core application (all screens, logic, depreciation math, data model). This is the primary reference.
- `PCE App.dc.html` — presentation wrapper showing the app live inside a phone frame and a desktop browser frame side by side.
- `PCE App - standalone.html` — a bundled, offline-capable single-file version of the same design (for viewing only, not a build artifact).
