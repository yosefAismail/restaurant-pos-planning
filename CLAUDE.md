# CLAUDE.md — Offline Restaurant Management System (Windows, Arabic)

This file is read at the start of every Claude Code session. It holds the STABLE rules. Current status lives in `PROGRESS.md`, never here.

## 1. What we are building
A complete, production-ready, offline-first Windows desktop application for a traditional Egyptian foul and taameya restaurant: POS, receipts, cash shifts, inventory, purchases, suppliers, expenses, employees, advances, payroll, dashboard, reports, backup/restore. Used by owners and cashiers with limited computer skills. The owner is a frontend engineer (React/TypeScript) who must fully own, understand, and extend the source.
Not a prototype. No mock screens, no fabricated data, no fake success messages, no placeholder primary workflows.

## 2. MANDATORY session protocol
### Start of every session (before ANY code change)
1. Read `PROGRESS.md`, `TASKS.md`, `PROJECT_PLAN.md`, `DECISIONS.md`, and the relevant `docs/` files.
2. Inspect the real repository and `git status` / `git log -5`.
3. Verify the last recorded checkpoint matches the code. **The repository is authoritative over any note.** If they disagree, investigate and fix the notes first.
4. Run the build/typecheck/tests that exist (when practical) to learn the true baseline.
5. Resume at the first unfinished task whose dependencies are satisfied. Never restart from scratch, never recreate or overwrite working files, never repeat completed work unless verification shows it is broken.
Triggers such as "continue", "resume", "next phase", "كمل", "كمل من آخر نقطة" mean exactly this protocol, then implementation. Never answer them by re-posting the plan.

### Development loop (per small batch)
1. Pick a small coherent set of tasks; mark them `IN PROGRESS` in `TASKS.md` and `PROGRESS.md`.
2. Implement. 3. Run tests, typecheck, lint, build. 4. Fix regressions before moving on.
5. Review data integrity, permissions, error handling, compatibility with earlier phases.
6. Update docs to match the code. 7. Update `PROGRESS.md`, `TASKS.md`, `CHANGELOG.md` with the exact outcome.
8. Commit a logical Git checkpoint (never secrets, real databases, or private business data).
Keep batches small enough to review. Keep the app buildable at every checkpoint. Do not start a new major module while a critical regression, integrity bug, failing migration, or broken build is open.

### End of session / approaching context limit
Stop starting large tasks. Finish or safely checkpoint the current small task. No partial or destructive edits left behind. Run the most relevant verification. Then update `PROGRESS.md` with: current phase, last completed task, in-progress task, files changed, commands actually run, actual results, known bugs, migration/data-safety concerns, the **next concrete task ID**, and the resume command if any. Update `TASKS.md`, `DECISIONS.md`, `CHANGELOG.md`. Give the owner a concise handover.

### Status values (use only these)
`NOT STARTED` · `IN PROGRESS` · `BLOCKED` · `IMPLEMENTED, NOT VERIFIED` · `VERIFIED` · `DEFERRED`
`VERIFIED` only after acceptance criteria were checked and the verification was actually executed. Cannot be tested here (Windows, installer, physical printer) → `IMPLEMENTED, NOT VERIFIED` plus exactly what remains to check.

## 3. Honesty rules
- Never claim a test passes, an installer works, or a printer prints unless it was actually run in a suitable environment. Say what was and was not run.
- The sandbox may be Linux or lack Rust/Windows. Record the environment in `PROGRESS.md`. Windows-only items go in `docs/VERIFY_ON_WINDOWS.md` as an exact checklist for the owner.
- Never claim payment-provider integration. Non-cash methods only record method and amount.
- Do not silently skip a phase, downgrade a required feature to a placeholder, or drop an item to save effort. If something must be deferred: mark `DEFERRED`, explain why, and record impact.

## 4. Technology stack (pin and document exact versions in `DECISIONS.md` / `docs/BUILD_AND_RELEASE.md`)
- Desktop: **Tauri 2** + Rust. Frontend: **React, TypeScript (strict), Vite, Tailwind CSS**, React Hook Form + Zod, a local-only charting library, Vitest + React Testing Library.
- Database: **SQLite** in the OS application-data directory (not in the repo or install folder). Versioned migrations, `PRAGMA foreign_keys=ON`, transactions for related writes.
- Password hashing: Argon2id (Rust). No cloud backend, no telemetry, no online auth/licensing, no CDN/remote fonts, no runtime internet dependency.
- Add dependencies only when needed; prefer maintained ones; document why.

## 5. Architecture
Separate: UI pages/components · feature modules · domain services (business rules) · repositories (SQL) · schemas/types · Tauri commands/OS integration · receipt rendering & printing · auth/authorization · backup/restore · reporting/financial calculations.
- Target layout: `src/{app,components,features/<module>,services,repositories,schemas,types,lib,styles,i18n}`, `src-tauri/{src,capabilities,migrations,icons,tauri.conf.json}`, `tests/`, `docs/`. Create only folders that hold real code.
- **The Rust side is the authoritative layer** for: authentication, permission checks, financial calculations, sale/purchase/stock/payroll transactions, receipt numbering, backup/restore. The frontend sends intent (items, quantities, payments, idempotency key) and never authoritative totals. Frontend money helpers exist only for preview/display and must pass the same shared test vectors as Rust. (Decision D-002; any change needs a new DECISIONS entry.)
- No unrestricted shell, filesystem, or arbitrary SQL exposed to the frontend. Minimal Tauri capabilities. Restrictive CSP.
- No giant files or god services. One module = one clear responsibility.

## 6. Independent restaurants
One install = one restaurant = one database (sales, receipt sequence, products, inventory, purchases, expenses, employees, payroll, shifts, users, receipt settings, audit log). No sync, no shared server. First-run wizard creates the restaurant profile and a stable `restaurant_id` (UUID) stored in the DB and in backup metadata. Restore/import must show restaurant identity, refuse mismatches without explicit, explained confirmation, and never merge records silently.

## 7. Arabic / RTL / Egypt localization
- Entire user-facing UI in Arabic; identifiers, schema, code, and developer docs in English.
- True RTL: layout, tables, sidebar, forms, dialogs, pagination, icons that imply direction, focus order, keyboard behavior, charts. Not just `dir="rtl"`.
- Locally bundled Arabic-capable font. Currency EGP shown as ج.م. Egyptian number/date presentation. Business dates and timestamps in Africa/Cairo (use the IANA tz database; Egypt observes DST).
- UX for non-technical users: large readable text, big touch-friendly targets, Arabic text labels over ambiguous icons, useful empty states, understandable errors, confirmation for destructive actions, loading/success/failure states, double-submit prevention, mouse+keyboard friendly, fast cashier flow, usable at common Windows resolutions.
- Reusable design system: Button, Input, Select, Dialog, Table, Badge, Alert, Card, PageLayout, Toast, Confirm.

## 8. Financial & data integrity rules (non-negotiable)
- Money = **INTEGER piastres** (1 EGP = 100). Never floats for authoritative money. Quantities = **INTEGER thousandths** (milli-units) so kg/liter fractions are exact. Explicit documented rounding: round half up, applied once per line (`line_total = round_half_up(qty_milli * unit_price / 1000)`), then sum; discounts defined and tested; no hidden rounding.
- Frontend totals are never trusted; recompute and validate in Rust.
- Related writes (sale + items + payments + receipt number + stock + shift/cash effects + audit) in ONE transaction. Failure leaves nothing partial.
- Duplicate-submission protection: client-generated idempotency key stored with a UNIQUE constraint, plus disabled-while-pending UI. Retrying a print never creates a sale; a committed sale survives any print failure.
- Receipt numbers unique per restaurant (UNIQUE constraint), allocated inside the sale transaction.
- Completed financial records are never silently edited or deleted. Use reversals/adjustments/corrections that keep the original, record who/when/why, and have defined effects on reports, cash, and stock.
- Historical snapshots: sale lines store name/price/discount as sold; payroll finalized records store their own figures; changing a product price or employee salary never alters history.
- Deactivate, don't delete, anything referenced by history. Foreign keys protect history (`RESTRICT`, not cascade, for financial links).
- Revenue is not profit. Profit/COGS shown only when cost data supports it, and labeled with its basis. Purchases, stock consumption, and expenses must not be double-counted (rules in `docs/ACCOUNTING_RULES.md`).
- Every schema change is a versioned, repeatable, transactional migration with a pre-risky-migration backup and tests from older schema fixtures. Owners never hand-edit the production DB.
- Indexes for search/report paths; pagination server-side; never load whole histories into the frontend.

## 9. Security rules
- Argon2id with per-user salt; no plaintext passwords, no default production credentials (a dev-only seed must be impossible to ship). Login throttling, lock screen/auto-lock, activation/deactivation, password change.
- RBAC enforced in Rust commands, not just hidden buttons. Roles: Owner/Admin, Manager, Cashier, Inventory/Purchasing, Accountant (read-only). Admin can adjust permissions.
- Audit log (who, when, what, before/after summary, reason) for: user changes, sales, reversals, stock adjustments, expense edits, payroll changes, shift adjustments, backup/restore, sensitive settings. Never log passwords or secrets.
- Parameterized SQL only. No restaurant data in browser storage. No data leaves the machine. Log files avoid unnecessary financial/personal detail.
- Document that local security cannot defeat a person with full Windows admin access; recommend Windows account protection and disk encryption.

## 10. Core business behavior to preserve (summary; full detail is in the phase prompts)
- **POS**: manual lines (editable description, qty, unit price, optional discount) with NO catalog required; optional catalog via search or big buttons; price/discount edits permission-gated; payments cash/card/wallet/other-configurable (cash: received/remaining/change; non-cash: record only); split payments only if safe and tested; fast keyboard flow; clear-cart confirmation.
- **Sales history**: search/filter by receipt no., date range, cashier, payment method, total, status; detail with items, payments, receipt, audit history; controlled reversal workflow.
- **Receipts**: per-restaurant configurable (name, logo, branch, address, phones, optional tax/commercial numbers, header/footer/thank-you/promo, 58/80 mm, font size, copies, visibility toggles, preview); Windows printer selection, test print, reprint without a new sale. Do not rely on browser default printing. Arabic shaping/RTL/mixed text/wrapping must be tested; if a printer/driver can't print Arabic, document a fallback.
- **Cash shifts**, **inventory ledger**, **purchases/suppliers**, **expenses**, **employees/advances/payroll**, **dashboard/reports/CSV export**, **backup/restore (USB, offline)**: see phase prompts.

## 11. Coding standards
TypeScript `strict`, no `any` without justification, Zod at boundaries, explicit error types mapped to Arabic user messages, no swallowed errors, consistent naming, small focused modules, comments only where intent isn't obvious. Rust: `clippy` clean, `Result`-based errors, no `unwrap()` on fallible runtime paths. Tests live next to what they test or in `tests/`; financial logic needs unit AND integration tests. Never hardcode demo data into production paths. All user-facing strings Arabic (central i18n file).

## 12. Phase map (run one phase prompt per focused session; resume across sessions as needed)
| Phase | Scope | Prompt |
|---|---|---|
| 1 | Foundation: Tauri, React, TS, SQLite, migrations, auth, RBAC, audit, Arabic RTL design system | `docs/prompts/phase-1-foundation.md` |
| 2 | Catalog, POS, sales, payments, reversals, receipt config, thermal printing | `docs/prompts/phase-2-pos-receipts.md` |
| 3 | Cash shifts, inventory ledger, recipes, suppliers, purchases, expenses | `docs/prompts/phase-3-cash-inventory.md` |
| 4 | Employees, advances, payroll, dashboard, reports, exports | `docs/prompts/phase-4-staff-reports.md` |
| 5 | Backup/restore, security review, integration tests, Windows packaging, docs, final audit | `docs/prompts/phase-5-safety-delivery.md` |
Mid-phase resume prompt: `docs/prompts/RESUME.md`. Do not begin a phase until the previous phase's acceptance criteria are `VERIFIED` or each gap is documented as a blocker/deferral with impact. Requirement traceability: `PROJECT_PLAN.md` §4.

## 13. Required documentation (keep in sync with code; never document unbuilt features as built)
`README.md` (English) and `docs/ARCHITECTURE.md`, `DATABASE.md`, `ACCOUNTING_RULES.md`, `REPORT_DEFINITIONS.md`, `BACKUP_AND_RESTORE.md`, `RECEIPT_PRINTING.md`, `BUILD_AND_RELEASE.md`, `TESTING.md`, `SECURITY.md`, `VERIFY_ON_WINDOWS.md`. Include how to: add a module, add a DB field via migration, add a report, change the receipt template, run tests, build the installer, safely inspect/export SQLite (read-only), upgrade without losing data.
