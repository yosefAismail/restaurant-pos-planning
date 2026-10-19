# PHASE 3 PROMPT — Cash Shifts, Inventory, Purchases, Suppliers, Expenses

You are a senior full-stack/Rust engineer and financial-systems/inventory engineer continuing the offline restaurant management system. `CLAUDE.md` rules apply to everything below.

## Mission
Add cash-shift reconciliation, expenses, a ledger-based inventory with optional recipe deduction, suppliers, and purchases — all transactional, auditable, and consistent with the Phase 2 sales model. Reports are Phase 4; here you build the data, the workflows, and the inventory screens.

## Prerequisite check
Phase 2 criteria `VERIFIED` (or documented). Confirm sales, reversals, receipt numbering, idempotency, and permissions pass before extending the sale transaction.

## 0. Mandatory start protocol (do this BEFORE any code change)
1. Read `CLAUDE.md`, then `PROGRESS.md`, `TASKS.md`, `PROJECT_PLAN.md`, `DECISIONS.md`, and any `docs/` file relevant to this phase.
2. Inspect the actual repository: structure, `git status`, `git log -5`, installed toolchain. The repository is authoritative over notes; if they disagree, investigate and correct the notes first.
3. Run the existing build, typecheck, lint, and tests to learn the real baseline. Do not trust old "VERIFIED" marks blindly if the baseline is red.
4. Find the first task of THIS phase that is not `VERIFIED` and whose dependencies are satisfied. Resume there. Never restart the phase, recreate working files, or repeat completed work unless verification proves it broken.
5. If the previous phase has unverified acceptance criteria, either verify them now or record each as a documented blocker/deferral with impact before continuing.

## Dependencies
P2-04 sale transaction, P2-05 hooks and nullable `cash_shift_id`, P2-10 reversal, P1-11 permissions, P1-12 audit.

## Tasks (IDs match `TASKS.md`)
**P3-01 Migration(s).** `cash_shifts` (cashier, opened_at, opening_cash, closed_at, counted_cash, expected_cash snapshot, difference, notes, status), `cash_movements` (shift, type in/out/expense/reversal/adjustment, amount, reason, user, link to expense/sale/purchase payment), `shift_adjustments`, `expense_categories`, `expenses`, `units`, `unit_conversions`, `inventory_items`, `stock_movements` (append-only), `stock_balances` (cached), `recipes`, `recipe_ingredients`, `suppliers`, `purchases`, `purchase_items`, `supplier_payments`; add enforced FK from `sales.cash_shift_id`; `products.track_mode` (`none|item|recipe`). Idempotency keys on purchases, expenses, shift operations, stock adjustments. Constraints: no negative amounts where invalid, conversion factor > 0, FK RESTRICT. Indexes for date/supplier/item/shift lookups. Update `docs/DATABASE.md` and ERD.

**P3-02 Cash shifts.** Open (opening cash, assigned cashier, start time), cash additions/withdrawals with required reasons, cash expenses paid from drawer (permission-gated), close with counted cash; expected closing = opening + cash sales − cash change handled correctly − cash reversals/refunds + additions − withdrawals − drawer expenses (define exactly in `docs/ACCOUNTING_RULES.md`); surplus/shortage; notes; history list. Distinguish cash from card/wallet/other everywhere. Closed shifts are immutable; later corrections use audited `shift_adjustments`. Default policy D-009 (finalize: one open shift per cashier, simultaneous shifts for different cashiers, `require_open_shift_for_sales` default ON); document it. Cashiers manage only their own shift unless `shifts.manage_all`.

**P3-03 Enforce shifts on sales.** `complete_sale` links the active shift (or rejects with a clear Arabic message when the setting requires one); reversal of a cash sale creates the drawer effect defined in P3-02; per-shift totals by payment method. Update Phase 2 tests; no regressions. Handle a shift closed while a cart is open (clear message, cart preserved).

**P3-04 Expenses.** Configurable categories (seed: rent, electricity, water, gas, salaries, maintenance, ingredients and supplies, transportation, other), create expense (category, description, amount, date/time, payment method, optional supplier/recipient, optional shift link when paid from drawer, notes, reference, recorded by), filters, daily/monthly totals, category breakdown, history. Corrections: void or amend through an audited workflow that preserves the original; drawer effect reversed accordingly. Duplicate prevention via idempotency key. Ensure salary-category expenses and later payroll don't double-count (see P3-11).

**P3-05 Units and conversions.** Seed kg, g, L, mL, piece, pack/box; admin-configurable additional units; conversions only when explicitly defined with a validated factor; never treat units as equivalent implicitly. Quantities remain integer thousandths. Tests for g↔kg and invalid cross-family conversions.

**P3-06 Inventory items and ledger.** Items: Arabic name, category, unit, optional SKU, minimum threshold, optional unit cost, optional supplier, active. Stock = append-only `stock_movements`; cached `stock_balances` updated in the same transaction as every movement; `reconcile_stock()` recomputes from the ledger and reports differences. Finalize D-010. Movements can never be inserted without updating the balance (test with injected failure).

**P3-07 Stock movements.** Types: opening, purchase receipt, sale consumption, manual adjustment (reason required, permission `inventory.adjust`), waste/spoilage (reason), return. Each records item, quantity and unit, type, timestamp, user, related purchase/sale/adjustment, notes. Screens: item list, item detail with movement history, adjustment/waste forms, low-stock list. Decide and document whether negative stock is blocked, warned, or allowed (setting; default warn) and test it.

**P3-08 Recipes and optional deduction.** Recipe mapping product → ingredient quantities only when the owner enters them (never invent amounts). Product track modes: `none` (default; no stock movements at all), `item` (direct stock item), `recipe`. Inside `complete_sale`'s transaction create `sale consumption` movements for tracked products only; reversal creates matching `return` movements; manual lines never touch stock. Configurable policy documented. Tests: untracked sale creates zero movements; recipe sale deducts exact milli-quantities; reversal restores; failure rolls back both.

**P3-09 Suppliers.** Name, phone, address, notes, active; deactivate-not-delete; outstanding balance derived from purchases minus supplier payments.

**P3-10 Purchases.** Create/confirm purchase: supplier, date, reference/invoice number, lines (item, quantity, unit, unit cost), discounts, total, amount paid, remaining due, payment method, notes, recorded by. Confirmation writes purchase, lines, `purchase` stock movements, supplier payment (if any) and a drawer cash movement when paid in cash from the drawer — in ONE transaction, idempotent (double submit adds stock once). Later payments against balances via `supplier_payments`. Corrections: reversal/correction records preserving the original, reversing stock and payments with audit. Optionally update item last-cost (documented, never rewrites history).

**P3-11 Accounting rules.** Complete `docs/ACCOUNTING_RULES.md`: sales, discounts, reversals, cash drawer, purchases (stock + supplier obligation, NOT an expense), non-stock purchases (expense), stock consumption cost (only with cost data), waste, supplier balances, drawer expenses, shift reconciliation. Include worked numeric examples that tests reuse. Finalize D-011.

**P3-12 Search and filters.** Server-side paginated search/sort/filter for products, inventory items, stock movements, purchases, suppliers, expenses, shifts. Deactivated records stay searchable and referenced history never disappears.

**P3-13 Inventory screens/reports.** Current stock, low-stock alerts, movement history, valuation (only where cost exists, labeled), waste report, adjustment report, reconciliation view. CSV export arrives in Phase 4; keep data access reusable.

**P3-14 Tests.** Unit: shift expected-cash formula, unit conversions, stock math, purchase totals/remaining. Integration: shift open→sales→expense→withdrawal→close with surplus and shortage; closed-shift immutability; expense not double-counted; purchase updates stock once under replay; purchase failure rollback; reversal restores stock; waste/adjustment audit; reconcile detects a deliberately corrupted balance; permissions (cashier cannot adjust stock; inventory role cannot reverse sales). UI: open/close shift, add purchase, record expense.

**P3-15 Docs and gate.** Update `README.md`, `docs/ARCHITECTURE.md`, `docs/DATABASE.md`, `docs/ACCOUNTING_RULES.md`, `docs/TESTING.md`; phase gate review.

## Expected deliverables
Shift module, expenses, units, inventory ledger with reconciliation, recipes, suppliers, purchases, sale/reversal integration, migrations, tests, docs, tracking updates.

## Acceptance criteria (measurable)
1. Shift reconciliation fixtures (≥ 8 scenarios: cash-only, mixed methods, reversal, change given, drawer expense, withdrawal, addition, shortage and surplus) match hand-calculated expected cash.
2. Closed shift cannot be modified through any command (test); corrections appear as audited adjustments.
3. An expense paid from the drawer reduces expected cash exactly once; the same expense never appears in two places (test).
4. Every stock movement changes the cached balance in the same transaction; `reconcile_stock()` reports zero differences on valid data and detects a tampered balance.
5. Purchase replayed with the same key adds stock once; injected failure leaves no partial purchase, movement, or payment.
6. Untracked/manual sales create zero stock movements; recipe sales deduct exactly; reversal restores exactly.
7. Unit conversions only via defined factors; invalid conversion rejected (test).
8. Supplier balance equals purchases minus payments on fixtures.
9. Permissions are enforced in Rust for all new commands (matrix test).
10. `docs/ACCOUNTING_RULES.md` complete with worked examples that match tests.
11. All Phase 1–2 tests still pass.

## Out of scope
Payroll, employees, dashboard, report exports (Phase 4), backup/restore (Phase 5).

## Verification limits
Everything here is testable without Windows hardware. UI RTL visuals still need a desktop run on the owner's PC to be fully `VERIFIED`.

## Session end protocol (mandatory, every session)
Stop starting large tasks when context is running low. Checkpoint the current small task so nothing is half-edited. Run the most relevant verification. Then update:
- `PROGRESS.md`: current phase, last completed task, in-progress task, exact files changed, commands actually executed, actual results, known bugs/limitations, migration/data-safety concerns, **the next concrete task ID**, and the resume command.
- `TASKS.md` statuses (only `VERIFIED` when the verification was really run), `CHANGELOG.md`, `DECISIONS.md`, and affected docs.
- Commit a logical Git checkpoint (no secrets, no real databases, no private business data).
Finish with a concise handover: what was implemented, what was verified (with the commands and results), what remains, blockers/risks, and the exact next task. Do not claim this phase is complete unless every acceptance criterion above has been checked against the real code and real test output.
