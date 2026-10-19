# TASKS.md

Status values: `NOT STARTED` · `IN PROGRESS` · `BLOCKED` · `IMPLEMENTED, NOT VERIFIED` · `VERIFIED` · `DEFERRED`.
Detail and acceptance criteria for each ID are in the matching phase prompt. Update this file as work happens. "Req" = section of the owner's original prompt.

## Phase 1 — Foundation (`docs/prompts/phase-1-foundation.md`)
| ID | Task | Req | Status |
|---|---|---|---|
| P1-01 | Inspect repo, git init, `.gitignore`, record toolchain/environment | 3, 20 | VERIFIED |
| P1-02 | Scaffold Tauri 2 + React + strict TS + Vite + Tailwind; pin versions | 4, 26 | NOT STARTED |
| P1-03 | Lint, format, Vitest + RTL, cargo test, scripts | 24 | NOT STARTED |
| P1-04 | Money & quantity core (piastres, milli-units, rounding) Rust + TS + shared vectors | 18 | NOT STARTED |
| P1-05 | Time utilities (UTC storage, Cairo business date, DST) | 7, 18 | NOT STARTED |
| P1-06 | SQLite integration in app-data dir, PRAGMAs, error mapping, D-002 spike | 4, 18, 21 | NOT STARTED |
| P1-07 | Migration framework (versioned, transactional, failure-safe, pre-migration backup hook) | 18 | NOT STARTED |
| P1-08 | Core schema: restaurant, users, roles/permissions, audit_log, settings; DATABASE.md | 6, 8, 18 | NOT STARTED |
| P1-09 | First-run setup wizard (restaurant profile + restaurant_id + admin) | 6, 8 | NOT STARTED |
| P1-10 | Authentication (Argon2id, login/logout, session, lock, password change, activation, throttling) | 8 | NOT STARTED |
| P1-11 | RBAC (5 roles, configurable permissions, Rust enforcement) | 8 | NOT STARTED |
| P1-12 | Audit log service + viewer | 8 | NOT STARTED |
| P1-13 | Arabic RTL design system, bundled font, core components | 7 | NOT STARTED |
| P1-14 | App shell: RTL sidebar, routing, permission-aware nav, lock screen, real home page | 7 | NOT STARTED |
| P1-15 | Settings foundation: restaurant profile, users & permissions UI, app info | 23 | NOT STARTED |
| P1-16 | Tauri security: CSP, minimal capabilities, offline asset check | 5, 22 | NOT STARTED |
| P1-17 | Foundation tests, docs (README, ARCHITECTURE, DATABASE, TESTING), phase gate | 24, 25, 27 | NOT STARTED |

## Phase 2 — POS, Sales, Receipts (`docs/prompts/phase-2-pos-receipts.md`)
| ID | Task | Req | Status |
|---|---|---|---|
| P2-01 | Migration: categories, products, payment methods, sales, sale_items, payments, receipt counter, reversals, receipt/printer settings | 9, 10, 18 | NOT STARTED |
| P2-02 | Product categories + optional product catalog | 9, 23 | NOT STARTED |
| P2-03 | Payment methods configuration | 9, 23 | NOT STARTED |
| P2-04 | Sales domain service in Rust (validation, authoritative totals, idempotency, receipt number, transaction, audit) | 9, 18 | NOT STARTED |
| P2-05 | Extension points for shifts and stock (nullable shift id, hook interface) | 9, 11, 12 | NOT STARTED |
| P2-06 | POS UI: manual lines, catalog quick-add, cart, notes, shortcuts, payments, change, split payments | 7, 9 | NOT STARTED |
| P2-07 | Double-submit protection (UI + backend) | 9, 21 | NOT STARTED |
| P2-08 | Completion success/failure flow, print offer | 9 | NOT STARTED |
| P2-09 | Sales history: search, filters, pagination, detail, audit view | 9, 17 | NOT STARTED |
| P2-10 | Reversal/cancellation workflow | 9, 8 | NOT STARTED |
| P2-11 | Receipt settings module + live preview | 10, 23 | NOT STARTED |
| P2-12 | Receipt renderer (58/80 mm, Arabic shaping, RTL, wrapping) + bitmap spike | 10 | NOT STARTED |
| P2-13 | Printing service: Windows printers, test print, copies, reprint, error handling, mock printer | 10 | NOT STARTED |
| P2-14 | Phase 2 tests (totals, rounding, payments, duplicates, rollback, numbering, reprint, printer failure, permissions) | 24 | NOT STARTED |
| P2-15 | Docs (RECEIPT_PRINTING, ACCOUNTING_RULES draft, VERIFY_ON_WINDOWS), phase gate | 25, 27 | NOT STARTED |

## Phase 3 — Cash, Inventory, Purchases, Expenses (`docs/prompts/phase-3-cash-inventory.md`)
| ID | Task | Req | Status |
|---|---|---|---|
| P3-01 | Migration: shifts, cash movements, expense categories, expenses, units, conversions, inventory items, stock movements, recipes, suppliers, purchases, supplier payments | 11–14, 18 | NOT STARTED |
| P3-02 | Cash shift module (open/close, drawer in/out, reconciliation, immutability, adjustments, policy) | 11 | NOT STARTED |
| P3-03 | Enforce shifts on sales; per-shift payment breakdown | 11 | NOT STARTED |
| P3-04 | Expenses: categories, CRUD, drawer link, filters, totals, audited correction | 14, 23 | NOT STARTED |
| P3-05 | Units and explicit conversions | 12, 23 | NOT STARTED |
| P3-06 | Inventory items + stock ledger + reconciliation | 12 | NOT STARTED |
| P3-07 | Stock movements (opening, purchase, sale, adjustment, waste, return), low-stock | 12 | NOT STARTED |
| P3-08 | Recipes + optional deduction wired into sale and reversal transactions | 12 | NOT STARTED |
| P3-09 | Suppliers | 13 | NOT STARTED |
| P3-10 | Purchases (transactional, partial payment, balances, idempotent, audited correction) | 13 | NOT STARTED |
| P3-11 | Accounting rules doc (purchases vs expenses vs stock consumption) | 14 | NOT STARTED |
| P3-12 | Search/filter/pagination for products, inventory, purchases, suppliers, expenses, shifts | 17 | NOT STARTED |
| P3-13 | Inventory reporting screens (stock, low stock, movements, valuation, waste, adjustments, reconciliation) | 12 | NOT STARTED |
| P3-14 | Phase 3 tests | 24 | NOT STARTED |
| P3-15 | Docs update, phase gate | 25, 27 | NOT STARTED |

## Phase 4 — Staff, Dashboard, Reports (`docs/prompts/phase-4-staff-reports.md`)
| ID | Task | Req | Status |
|---|---|---|---|
| P4-01 | Migration: employees, advances, payroll periods/records, reporting indexes | 15, 16, 18 | NOT STARTED |
| P4-02 | Employees + salary arrangements | 15 | NOT STARTED |
| P4-03 | Salary advances + no-double-deduction | 15 | NOT STARTED |
| P4-04 | Payroll service, finalization snapshot, payment, statements, history | 15 | NOT STARTED |
| P4-05 | `REPORT_DEFINITIONS.md` (precise metrics) | 16 | NOT STARTED |
| P4-06 | Reporting query layer (Rust), date-range and Cairo boundaries, indexes | 16, 21 | NOT STARTED |
| P4-07 | Dashboard | 16 | NOT STARTED |
| P4-08 | Full report set | 16 | NOT STARTED |
| P4-09 | CSV / Excel-compatible export, Arabic UTF-8, naming, permissions | 16 | NOT STARTED |
| P4-10 | Search/history for employees, payroll, historical financial activity | 17 | NOT STARTED |
| P4-11 | Phase 4 tests | 24 | NOT STARTED |
| P4-12 | Docs update, phase gate | 25, 27 | NOT STARTED |

## Phase 5 — Safety and Delivery (`docs/prompts/phase-5-safety-delivery.md`)
| ID | Task | Req | Status |
|---|---|---|---|
| P5-01 | Backup service (safe snapshot, manifest, checksum, post-verify) | 19 | NOT STARTED |
| P5-02 | Backup UI: manual, destination/USB, last result, recent list, reminder, scheduled local backup | 19, 23 | NOT STARTED |
| P5-03 | Restore with validation chain, identity check, recovery backup, safe swap | 6, 19 | NOT STARTED |
| P5-04 | USB cross-computer restore flow (simulated + owner checklist) | 19 | NOT STARTED |
| P5-05 | Migration upgrade tests from old schema fixtures; pre-migration backup integration | 18 | NOT STARTED |
| P5-06 | Permission and security review, audit coverage, `SECURITY.md` | 8, 22 | NOT STARTED |
| P5-07 | Integration and failure-scenario test completion | 24 | NOT STARTED |
| P5-08 | UI/workflow tests for primary Arabic flows | 24 | NOT STARTED |
| P5-09 | Arabic RTL verification pass | 7 | NOT STARTED |
| P5-10 | Performance pass on large seeded dataset | 21 | NOT STARTED |
| P5-11 | Windows packaging config (installer, shortcut, icon, version, WebView2 offline, data policy) | 5 | NOT STARTED |
| P5-12 | Build installer if possible; else exact Windows steps + `VERIFY_ON_WINDOWS.md` | 5 | NOT STARTED |
| P5-13 | Final documentation set and handover | 20, 25 | NOT STARTED |
| P5-14 | Final acceptance audit against original §29; honest report | 29 | NOT STARTED |
