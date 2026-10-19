# PROJECT_PLAN.md

Stable roadmap. Status lives in `PROGRESS.md` / `TASKS.md`. Rules live in `CLAUDE.md`.

## 1. Goal
Offline-first Windows desktop app (Tauri 2 + React + TypeScript + SQLite) to run a foul & taameya restaurant: POS, thermal receipts, cash shifts, inventory, purchases, expenses, payroll, reports, backup/restore. Arabic RTL. One independent database per restaurant. Owner owns all source.

## 2. Architecture summary
```
React UI (Arabic RTL)  --invoke-->  Tauri commands (Rust)  -->  Domain services  -->  Repositories  -->  SQLite (app-data dir)
   preview-only money helpers          permission check            authoritative            parameterized SQL      WAL/FK ON
                                       input validation            totals, tx, audit
Receipt renderer -> raster bitmap -> Windows printer service (spooler)      Backup service -> SQLite snapshot + manifest + checksum
```
Key choices (see `DECISIONS.md`): Rust owns the DB and business rules (D-002); money in piastres, quantities in thousandths (D-003); UTC timestamps + Cairo business date (D-004); idempotency keys for duplicate prevention (D-005); receipt printing via rendered bitmap by default because thermal-printer Arabic code pages are unreliable (D-007, risk R-001).

## 3. Phases and dependencies
```
P1 Foundation -> P2 POS & Receipts -> P3 Cash & Operations -> P4 Staff & Reporting -> P5 Safety & Delivery
```
| Phase | Depends on | Hands to next phase |
|---|---|---|
| 1 | — | migrations, money/time core, auth, RBAC, audit, design system, app shell |
| 2 | 1 | sales schema with nullable `shift_id` and stock hook points, receipts, printing abstraction |
| 3 | 2 | shifts enforced on sales, stock ledger wired into sales/reversals, expenses/purchases for reports |
| 4 | 3 | metric definitions, reports, CSV export |
| 5 | 4 | verified backup/restore, packaging, final audit |
Cross-phase rule: every phase re-runs ALL earlier tests; any regression is fixed before the phase closes.

## 4. Requirements traceability (original prompt section -> where it is implemented)
| Original § | Topic | Phase / tasks |
|---|---|---|
| 1, 2, 3, 28 | Role, incremental workflow, tracking files, inspection | `CLAUDE.md`, all prompts, P1-01 |
| 4, 26 | Stack, architecture, structure | P1-02..P1-03, P1-06 |
| 5 | Windows installer, offline, upgrade/uninstall policy | P5-11, P5-12, P1-16 |
| 6 | Independent restaurants, identity | P1-09, P5-03 |
| 7 | Arabic RTL, localization, UX, design system | P1-13, P1-14 (+ every UI task), P5-09 |
| 8 | Auth, roles, permissions, audit | P1-10..P1-12, P5-06 |
| 9 | POS, manual lines, catalog, cart, payments, completion, history, reversal | P2-02..P2-10 |
| 10 | Receipt config and thermal printing | P2-11..P2-13, P2-15 |
| 11 | Cash shifts and reconciliation | P3-02, P3-03 |
| 12 | Inventory ledger, units, recipes, optional POS integration, reports | P3-05..P3-08, P3-13 |
| 13 | Suppliers and purchases | P3-09, P3-10 |
| 14 | Expenses and accounting treatment | P3-04, P3-11 |
| 15 | Employees, advances, payroll | P4-02..P4-04 |
| 16 | Dashboard, reports, definitions, CSV export | P4-05..P4-09 |
| 17 | Search and historical records | P2-09, P3-12, P4-10 |
| 18 | Schema, financial integrity, migrations | P1-04..P1-08, each phase's migration task, P5-05 |
| 19 | Backup, restore, USB | P5-01..P5-04 |
| 20 | Source ownership, handover | P1-01, P1-17, P5-13 |
| 21 | Performance, reliability | P5-10 + per-phase pagination/index work |
| 22 | Security and privacy | P1-16, P5-06 |
| 23 | Central settings | P1-15, P2-03, P2-11, P3-04, P3-05, P5-02 |
| 24 | Testing | per-phase test tasks, P5-07, P5-08 |
| 25 | Documentation | per-phase doc tasks, P5-13 |
| 27 | Phase gates | each phase prompt "Acceptance criteria" |
| 29 | Final acceptance | P5-14 |
Any original-prompt requirement not traceable here is a gap: add it to the right phase before starting that phase.

## 5. Verification matrix (what cannot be proven in a Linux/sandbox session)
| Item | Needs | Where recorded |
|---|---|---|
| Tauri dev run, WebView2 rendering, RTL visuals | Windows (or WebView2/WebKit-capable desktop) | `docs/VERIFY_ON_WINDOWS.md` |
| MSI/NSIS installer build, install, upgrade, uninstall, shortcut, icon | Windows machine with MSVC + WiX/NSIS | same |
| Offline install on a machine without WebView2 | Offline Windows machine | same |
| Thermal printing, Arabic shaping on paper, 58/80 mm, copies, cash-drawer-free | Real printer(s) + Windows drivers | same |
| USB backup/restore across two computers | Two Windows PCs + USB stick | same |
Everything else (domain logic, migrations, SQL, Rust unit/integration tests, frontend tests, bitmap receipt rendering to PNG) should be verified for real in any session that has the toolchain. If the toolchain is missing, mark `IMPLEMENTED, NOT VERIFIED`.

## 6. Risk register
| ID | Risk | Mitigation |
|---|---|---|
| R-001 | Thermal printers' ESC/POS code pages don't shape Arabic | Render receipt to bitmap; spike in P2-12; document compatible printers; text-mode fallback only where verified |
| R-002 | WebView2 missing on offline PCs | `webviewInstallMode` offline installer or fixed runtime (P5-11) |
| R-003 | Live DB copied during writes corrupts backup | Use SQLite backup API / `VACUUM INTO`, verify after (P5-01) |
| R-004 | Uninstall/upgrade wipes data | Data in app-data dir; installer option to keep data default-on; verify on Windows (P5-11/12) |
| R-005 | Frontend/Rust money drift | Shared test vectors (P1-04) |
| R-006 | Context loss across sessions | `PROGRESS.md` protocol, small batches, per-phase prompts |
| R-007 | Egypt DST/date-boundary bugs in reports | IANA tz, boundary tests (P1-05, P4-06) |
| R-008 | Scope creep (payment-provider integration, cloud) | Explicitly out of scope |

## 7. Out of scope (initially)
Cloud/sync/multi-branch consolidation, online payments or payment-provider integration, delivery-app integrations, auto-update from the internet (manual installer upgrades only), cash-drawer kick hardware unless added later, Egyptian e-invoice (ETA) integration. Record any request to add these as a new decision first.

## 8. Repository layout (target)
See `CLAUDE.md` §5. Planning files live at repo root; phase prompts in `docs/prompts/`.
