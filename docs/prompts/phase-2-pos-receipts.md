# PHASE 2 PROMPT — POS, Sales, Payments, Receipts, Thermal Printing

You are a senior full-stack/Rust engineer, financial-systems engineer, QA engineer, and Arabic UX designer continuing the offline restaurant management system. `CLAUDE.md` rules apply to everything below.

## Mission
Deliver the most important module: a fast, reliable cashier workflow that completes sales safely, stores them transactionally with unique receipt numbers, supports controlled reversals, and prints (or reprints) configurable thermal receipts without ever duplicating or losing a sale.

## Prerequisite check
Phase 1 acceptance criteria must be `VERIFIED` (or each gap documented). Confirm: migrations work, auth/RBAC/audit work, money core passes shared vectors, design system exists. If not, fix or document before building on top.

## 0. Mandatory start protocol (do this BEFORE any code change)
1. Read `CLAUDE.md`, then `PROGRESS.md`, `TASKS.md`, `PROJECT_PLAN.md`, `DECISIONS.md`, and any `docs/` file relevant to this phase.
2. Inspect the actual repository: structure, `git status`, `git log -5`, installed toolchain. The repository is authoritative over notes; if they disagree, investigate and correct the notes first.
3. Run the existing build, typecheck, lint, and tests to learn the real baseline. Do not trust old "VERIFIED" marks blindly if the baseline is red.
4. Find the first task of THIS phase that is not `VERIFIED` and whose dependencies are satisfied. Resume there. Never restart the phase, recreate working files, or repeat completed work unless verification proves it broken.
5. If the previous phase has unverified acceptance criteria, either verify them now or record each as a documented blocker/deferral with impact before continuing.

## Dependencies
P1-04 money core, P1-07 migrations, P1-10/11 auth and permissions, P1-12 audit, P1-13 design system.

## Tasks (IDs match `TASKS.md`)
**P2-01 Migration.** Tables: `product_categories`, `products` (Arabic name, optional SKU, category, unit label, default price in piastres, optional cost price, `track_inventory` flag reserved for Phase 3, active), `payment_methods` (code, Arabic name, enabled, sort, is_cash flag), `sales` (id, receipt_number UNIQUE, idempotency_key UNIQUE, cashier user id, nullable `cash_shift_id`, status `completed|reversed`, subtotal, discount_total, total, notes, created_at UTC, business_date), `sale_items` (sale id, nullable product id, description snapshot, quantity_milli, unit_price_piastres, discount_piastres, line_total_piastres, sort), `payments` (sale id, method code snapshot, amount, cash received, change given), `receipt_counter`, `sale_reversals` (original sale id UNIQUE, reason, authorized_by, performed_by, created_at, idempotency_key), `receipt_settings`, `printer_settings`. Constraints: quantity_milli > 0, price ≥ 0, totals consistent (CHECK where feasible), FK RESTRICT. Indexes for receipt number, date, cashier, status, payment method. Update `docs/DATABASE.md`.

**P2-02 Categories and optional catalog.** CRUD with search, active/inactive (deactivate, never delete if referenced), large-button "quick items" selection. The catalog is OPTIONAL: POS must work with an empty catalog. Permission-gated. Product price changes never alter past sales (snapshot).

**P2-03 Payment methods.** Seed Cash, Card, Wallet; admin can enable/disable and add "other" methods with Arabic names. At least one enabled method required; cash method flagged for change/cash-shift logic. No claim of provider integration anywhere in UI or docs.

**P2-04 Sales domain service (Rust, authoritative).** `complete_sale` command accepts lines (description, qty_milli, unit price, optional discount, optional product id), optional notes, payments, and an `idempotency_key`. In ONE transaction: check permission (`sales.create`; `sales.price_override` when price differs from catalog default; `sales.discount` for any discount), validate (qty > 0, price ≥ 0, discount ≤ line gross, description non-empty, at least one line), recompute every total with the money core (ignore client totals; reject if a client-supplied expected total differs, returning the authoritative figure), validate payments (sum ≥ total for cash overpay → change; non-cash must not exceed due; split payments sum exactly rules), allocate the next receipt number from the counter, insert sale, items, payments, record cashier and timestamps and business_date, write audit entry, commit. Replaying the same idempotency key returns the original sale. Decide and document discount order/rounding (D-003, D-006) and finalize `docs/ACCOUNTING_RULES.md` (draft sections for sales, discounts, reversals).

**P2-05 Extension points.** Nullable `cash_shift_id` and a documented `StockEffects` hook invoked inside the sale/reversal transaction (no-op now). Do not fabricate shift or stock behavior. Record the contract in `DECISIONS.md` (D-008).

**P2-06 POS UI.** Single fast screen: manual line entry (editable description, quantity, unit price, optional discount), numeric keypad-friendly inputs, quick-item buttons and searchable catalog list, editable cart with remove/edit, order note, clear-cart confirmation, prominent amount due, payment panel (cash: received/remaining/change; card/wallet/other: record only), split payments (implement only with full tests; otherwise mark `DEFERRED` with impact), keyboard shortcuts (documented on-screen: new line, finish, pay, clear, search), Arabic validation messages. Cashier with no catalog can sell "فول" or "طعمية" by typing a name and price.

**P2-07 Duplicate protection.** UI disables submit while pending and generates the idempotency key once per cart; backend UNIQUE key; test double-click, double Enter, and delayed response.

**P2-08 Completion flow.** After commit show a success state with receipt number and total, then offer print / skip / new sale. Failure shows an Arabic error and keeps the cart intact. A print failure never undoes or duplicates the sale.

**P2-09 Sales history.** Server-side paginated list and filters: receipt number, date range (Cairo days), cashier, payment method, total range, status. Detail view: lines, payments, receipt preview, reversal info, audit history, reprint button. Cashiers see their own sales unless permitted otherwise.

**P2-10 Reversal/cancellation.** Permission `sales.reverse` (Manager/Owner by default). Requires a reason; optional second-user authorization setting; one reversal per sale (UNIQUE); original sale preserved; status becomes `reversed` via the reversal record; effects on reports defined (gross sales, reversed sales, net sales) and on cash (a reversal of a cash sale creates a documented drawer effect, wired fully in Phase 3) and on stock (via the hook). Audit entry. Tests for double reversal, unauthorized reversal, rollback.

**P2-11 Receipt settings.** Per-restaurant: name, optional logo (stored locally, size-limited, validated image), branch, address, phones, optional tax and commercial numbers (never forced), header, footer, thank-you, promo text, paper width (58/80 mm), font size, copies, toggles for cashier name, receipt number, date/time, payment method, subtotal/discount/total, amount received/change. Live preview that matches printed output. Persisted in DB; survives restart.

**P2-12 Receipt renderer.** Layout engine that produces a receipt model → bitmap (D-007) for 58 and 80 mm: Arabic shaping and RTL alignment, mixed Arabic/English, Arabic-Indic or Western digits per setting, ج.م, long item descriptions with wrapping, QR/barcode NOT required. Output PNG in tests so layout can be inspected and golden-compared. Spike result and printer-compatibility notes go into `docs/RECEIPT_PRINTING.md`.

**P2-13 Printing service.** `Printer` interface + real Windows implementation + mock. List installed printers, save the selected printer (never change it silently), print test receipt, print N copies, reprint a past sale (reads stored data, creates NO sale), useful Arabic errors (printer offline, not selected, driver error), non-blocking UI, timeouts. If the toolchain cannot reach a Windows printer here, mark `IMPLEMENTED, NOT VERIFIED` and write the exact owner test procedure (58 mm and 80 mm, Arabic line wrap, copies, reprint, unplugged-printer failure) in `docs/VERIFY_ON_WINDOWS.md`.

**P2-14 Tests.** Unit: totals, per-line rounding, discounts, change, payment validation, receipt numbering, permissions. Integration: sale persisted with items/payments; forced failure mid-transaction → nothing persisted and counter not consumed; duplicate idempotency key → one sale; two concurrent completions → distinct unique numbers; reprint → sale count unchanged; mock printer failure after commit → sale intact; historical price immutability; reversal flows. UI: manual sale flow in RTL, double-click guard.

**P2-15 Docs and gate.** `docs/RECEIPT_PRINTING.md`, `docs/ACCOUNTING_RULES.md` (sales part), update `docs/DATABASE.md`, `docs/ARCHITECTURE.md`, `docs/TESTING.md`, `docs/VERIFY_ON_WINDOWS.md`. Phase gate review.

## Expected deliverables
Working POS, sales history, reversal workflow, receipt settings with preview, bitmap renderer, printing service with mock and Windows implementation, migrations, tests, updated docs and tracking files.

## Acceptance criteria (measurable)
1. A manual sale with an empty catalog completes and persists; totals equal an independent hand-calculated fixture set (≥ 15 cases including fractional kg quantities and half-piastre rounding).
2. Tampering test: a frontend-supplied wrong total is rejected or ignored; stored total equals Rust's computation.
3. 100 sequential and 20 concurrent completions yield unique, gap-free (on success) receipt numbers; UNIQUE constraint verified.
4. Same idempotency key sent 5 times → exactly 1 sale.
5. Injected failure at each step of `complete_sale` leaves zero partial rows and an unconsumed receipt number.
6. Reprint 10 times → sale count and receipt number unchanged. Mock printer failure after commit → sale exists, UI shows Arabic error, retry prints without creating a sale.
7. Reversal: one per sale, requires reason and permission, original rows untouched, audit entry present, report definitions documented.
8. Cashier role cannot override price/discount/reverse without permission (Rust-layer tests).
9. Receipt PNG goldens exist for 58 mm and 80 mm covering Arabic wrapping, mixed text, long names, optional fields on/off.
10. Sales history filters return correct rows at Cairo date boundaries (tests) and are paginated server-side.
11. All earlier-phase tests still pass.
12. Physical printing is `VERIFIED` only if actually done on hardware; otherwise `IMPLEMENTED, NOT VERIFIED` with the owner checklist written.

## Out of scope
Cash shift enforcement and stock deduction (Phase 3), purchases, expenses, payroll, dashboard, report exports.

## Verification limits
Thermal output, driver behavior, and real Arabic shaping on paper require the owner's printer and Windows PC.

## Session end protocol (mandatory, every session)
Stop starting large tasks when context is running low. Checkpoint the current small task so nothing is half-edited. Run the most relevant verification. Then update:
- `PROGRESS.md`: current phase, last completed task, in-progress task, exact files changed, commands actually executed, actual results, known bugs/limitations, migration/data-safety concerns, **the next concrete task ID**, and the resume command.
- `TASKS.md` statuses (only `VERIFIED` when the verification was really run), `CHANGELOG.md`, `DECISIONS.md`, and affected docs.
- Commit a logical Git checkpoint (no secrets, no real databases, no private business data).
Finish with a concise handover: what was implemented, what was verified (with the commands and results), what remains, blockers/risks, and the exact next task. Do not claim this phase is complete unless every acceptance criterion above has been checked against the real code and real test output.
