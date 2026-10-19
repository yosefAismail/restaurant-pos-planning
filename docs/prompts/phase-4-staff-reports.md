# PHASE 4 PROMPT — Employees, Advances, Payroll, Dashboard, Reports, Exports

You are a senior full-stack/Rust engineer and reporting/payroll-logic engineer continuing the offline restaurant management system. `CLAUDE.md` rules apply to everything below.

## Mission
Add employee management, salary advances, payroll with immutable finalized records, a truthful owner dashboard, precisely defined reports, and offline CSV/Excel-compatible exports with correct Arabic.

## Prerequisite check
Phases 1–3 `VERIFIED` (or documented). Re-run the full test suite first. Reports read from sales, reversals, payments, shifts, expenses, purchases, stock movements — any unfixed upstream bug will corrupt reports.

## 0. Mandatory start protocol (do this BEFORE any code change)
1. Read `CLAUDE.md`, then `PROGRESS.md`, `TASKS.md`, `PROJECT_PLAN.md`, `DECISIONS.md`, and any `docs/` file relevant to this phase.
2. Inspect the actual repository: structure, `git status`, `git log -5`, installed toolchain. The repository is authoritative over notes; if they disagree, investigate and correct the notes first.
3. Run the existing build, typecheck, lint, and tests to learn the real baseline. Do not trust old "VERIFIED" marks blindly if the baseline is red.
4. Find the first task of THIS phase that is not `VERIFIED` and whose dependencies are satisfied. Resume there. Never restart the phase, recreate working files, or repeat completed work unless verification proves it broken.
5. If the previous phase has unverified acceptance criteria, either verify them now or record each as a documented blocker/deferral with impact before continuing.

## Dependencies
P3-04 expenses, P3-10 purchases, P3-02 shifts, P2 sales/reversals, P3-11 accounting rules, P1-05 Cairo time ranges.

## Tasks (IDs match `TASKS.md`)
**P4-01 Migration.** `employees` (name, phone, job title, hire date, pay type monthly|daily, base amount, active, notes), `salary_advances` (employee, amount, date, notes, recorded_by, idempotency key, status, link to payroll record when deducted), `payroll_periods`, `payroll_records` (employee, period, snapshot of base/wage inputs, days worked if daily, bonuses, deductions, advances_deducted, net, status draft|finalized|paid, paid_at, payment method, notes, finalized_by), `payroll_advance_deductions` (UNIQUE advance id so an advance can be deducted once), report-supporting indexes. Update `docs/DATABASE.md`.

**P4-02 Employees.** CRUD, active/inactive (never deleted if referenced), salary arrangement monthly or daily wage with explicit calculation rules documented, search and filters. Changing salary settings affects only future payroll drafts.

**P4-03 Advances.** Record with employee, amount, date, notes, recorded by (audited, idempotent). Outstanding advances list per employee. Deduction tracking: an advance (or a documented partial amount with remaining balance) can be deducted only up to its outstanding amount; UNIQUE constraints prevent double deduction; reversal of a deduction only via audited correction. If paid from the drawer, create a shift cash movement.

**P4-04 Payroll service (Rust, authoritative).** Create period, generate draft records, edit draft bonuses/deductions (reasons required), auto-propose advance deductions (editable, capped), calculate net with integer piastres, finalize (snapshot becomes immutable; later salary edits never change it), mark paid with date/method (and drawer movement if cash from drawer). Corrections to finalized records via audited adjustment records preserving the original. No invented legal rules: attendance/overtime/absence deductions only as explicit, configurable manual inputs. Employee statement and payroll history. Define in `docs/ACCOUNTING_RULES.md` how paid salaries appear (payroll, not duplicated as generic expense).

**P4-05 Report definitions.** Write `docs/REPORT_DEFINITIONS.md` BEFORE coding reports: gross sales, discounts, reversed sales, net sales, cash collected, non-cash payments, expenses, purchases, supplier outstanding, drawer balance, shift difference, payroll cost, advances outstanding, stock valuation, and (only if cost data exists) cost of goods and gross margin with explicit caveats. State date basis (Cairo `business_date`), treatment of reversals by date, and what each report excludes. Revenue ≠ profit.

**P4-06 Reporting query layer.** Rust query functions implementing the definitions, using Cairo-day range → UTC helpers, efficient SQL with indexes, server-side pagination, EXPLAIN-checked for the main queries. No frontend aggregation of full tables.

**P4-07 Dashboard.** Today's sales, number of sales, average sale, sales by payment method, expenses, expected cash, open shifts, low-stock items, outstanding supplier balances (when tracked), payroll obligations (when relevant). Every number is computed from real data; empty states when nothing exists; permission-aware cards; refresh without freezing UI.

**P4-08 Reports.** Daily, weekly, monthly, custom range sales; by cashier; by payment method; by product or manual item description (normalized grouping documented); discounts; reversals/cancellations; cash-shift reconciliation; expenses by category; purchases and supplier balances; inventory and stock movements; waste and adjustments; employee payroll; employee advances; historical financial activity. Filters, sort, pagination, charts using a local-only library; RTL charts; Accountant role is read-only.

**P4-09 Exports.** CSV (UTF-8 with BOM so Excel shows Arabic) and, if practical, XLSX generated locally; clear file naming including report name and date range; file saved via a Tauri save dialog with scoped capability (no arbitrary fs exposure); permission `reports.export`; progress and error feedback; no network. Test Arabic round-trip, commas/quotes/newlines in fields, large exports without freezing UI.

**P4-10 Search and history.** Consistent search/filter/pagination/sort for employees, advances, payroll records, and a unified historical financial activity view (sales, reversals, expenses, purchases, payments, payroll) with filters; deactivated entities remain visible in history.

**P4-11 Tests.** Payroll: monthly, daily, bonuses, deductions, advances capped, advance never deducted twice (concurrency/replay), finalized immutability after salary edit, correction audit. Reports: every metric against fixtures with hand-computed expected values, including reversals, mixed payments, shifts crossing midnight, DST dates, empty ranges, large data. Exports: Arabic and special characters. Permissions matrix for new commands. UI: run a report, export, create advance, finalize payroll.

**P4-12 Docs and gate.** Update `README.md`, `docs/ARCHITECTURE.md`, `docs/DATABASE.md`, `docs/ACCOUNTING_RULES.md`, `docs/REPORT_DEFINITIONS.md`, `docs/TESTING.md`; phase gate review.

## Expected deliverables
Employees, advances, payroll, dashboard, full report set, exports, definitions doc, migrations, tests, docs, tracking updates.

## Acceptance criteria (measurable)
1. Payroll fixtures (≥ 10 cases) match hand-computed net amounts in piastres; no rounding drift.
2. The same advance cannot be deducted twice (replay + concurrent test); outstanding balance always = advance − deductions.
3. Finalized payroll records remain unchanged after employee salary edits (test); corrections are separate audited rows.
4. Each report metric has a definition in `REPORT_DEFINITIONS.md` and ≥ 1 fixture test; sales reconcile: gross − discounts − reversed = net, and payment-method totals = cash collected + non-cash.
5. Date-range tests pass at Cairo midnight and DST transition days.
6. Dashboard shows zero/empty states correctly on a fresh DB and correct figures on seeded fixtures; no hardcoded numbers (grep evidence).
7. Exported CSV opens with correct Arabic in a UTF-8-aware reader (automated BOM/encoding test); Excel visual check is `IMPLEMENTED, NOT VERIFIED` unless done by the owner.
8. Main report queries on a seeded large dataset (e.g. 100k sales) respond within a documented threshold (record numbers) using indexes.
9. Accountant role can view/export but not modify (Rust-layer test).
10. All Phase 1–3 tests still pass.

## Out of scope
Backup/restore, installer, security review (Phase 5), attendance/biometric systems, legal payroll rules not provided by the owner.

## Verification limits
Opening exports in the owner's Excel and viewing charts in WebView2 need the owner's PC.

## Session end protocol (mandatory, every session)
Stop starting large tasks when context is running low. Checkpoint the current small task so nothing is half-edited. Run the most relevant verification. Then update:
- `PROGRESS.md`: current phase, last completed task, in-progress task, exact files changed, commands actually executed, actual results, known bugs/limitations, migration/data-safety concerns, **the next concrete task ID**, and the resume command.
- `TASKS.md` statuses (only `VERIFIED` when the verification was really run), `CHANGELOG.md`, `DECISIONS.md`, and affected docs.
- Commit a logical Git checkpoint (no secrets, no real databases, no private business data).
Finish with a concise handover: what was implemented, what was verified (with the commands and results), what remains, blockers/risks, and the exact next task. Do not claim this phase is complete unless every acceptance criterion above has been checked against the real code and real test output.
