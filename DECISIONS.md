# DECISIONS.md

Record every architectural or business-rule decision with its reason and consequences. Status: `PROPOSED` (confirm in the named task) · `ACCEPTED` · `SUPERSEDED by D-xxx`. Never edit an accepted decision silently; add a new one that supersedes it.

## D-001 — Tauri 2 desktop shell, React + TypeScript + Vite + Tailwind
Status: ACCEPTED (owner requirement). Reason: owner is a React/TS engineer; Tauri gives a small offline installer. Pin exact versions in P1-02 and record them here.

## D-002 — Rust owns the database and all authoritative business logic
Status: PROPOSED — confirm in P1-06 with a small spike. 
Decision: SQLite is opened by Rust (`rusqlite` with bundled SQLite, or equivalent maintained crate). The frontend calls typed Tauri commands (e.g. `complete_sale`, `create_purchase`) and never sends arbitrary SQL.
Reason: the requirements demand permission enforcement and recalculated financial totals "in the business and database command layers, not only by hiding buttons". A frontend-side SQL plugin would let any UI code bypass rules. Rust also gives access to SQLite's backup API (needed for safe backups) and Argon2id.
Consequence: more Rust than a pure-TS app; frontend stays thin. Frontend money helpers are preview-only and tested against the same fixtures (D-003). If the spike shows a blocking problem, write a superseding decision with the trade-offs.

## D-003 — Money in integer piastres; quantities in integer thousandths
Status: ACCEPTED. Reason: exactness; kg/liter fractions (e.g. 0.250 kg) must be exact.
Rules: `amount_piastres INTEGER`, `quantity_milli INTEGER`. `line_gross = round_half_up(quantity_milli * unit_price_piastres / 1000)`. Discounts are applied per line (fixed amount or percent converted to piastres once, round half up) and/or per sale as a fixed amount; the order and rounding are fixed in P2-04 and documented in `docs/ACCOUNTING_RULES.md`. No floats in authoritative paths. Shared JSON test vectors used by both Rust and TS tests.

## D-004 — Time handling
Status: ACCEPTED. Store `created_at` as UTC (ISO-8601 or epoch ms, chosen in P1-05) and ALSO store `business_date` (Africa/Cairo, `YYYY-MM-DD`) computed at write time with the IANA tz database. Reports group on `business_date`; range boundaries are tested around midnight and DST transitions (Egypt observes DST). Business day cut-off (e.g. restaurant open past midnight) is a configurable setting, default midnight; decided in P1-05.

## D-005 — Duplicate prevention by idempotency keys
Status: ACCEPTED. Every state-changing financial command (sale, purchase, expense, advance, payroll finalize, reversal) takes a client-generated UUID `idempotency_key` with a UNIQUE constraint. Replaying the same key returns the original result and creates nothing new. UI also disables the submit control while pending.

## D-006 — Receipt numbering
Status: PROPOSED — finalize in P2-04. Per-restaurant monotonically increasing integer from a counter row updated inside the sale transaction; displayed with configurable prefix/zero-padding. UNIQUE constraint on the stored number. Gaps may occur only if a transaction rolls back (counter update rolls back with it, so no gaps expected); reversals do not reuse numbers.

## D-007 — Thermal receipt printing strategy
Status: PROPOSED — confirm via spike in P2-12/P2-13.
Decision: render the receipt (Arabic shaping, RTL, wrapping) to a monochrome bitmap at the paper's dot width (58 mm ≈ 384 dots, 80 mm ≈ 576 dots typical) and send it to the selected Windows printer through the spooler (ESC/POS raster or GDI). A text-mode ESC/POS path may be offered only for printers verified to render Arabic. Printing lives behind a `Printer` interface with a mock implementation for tests.
Reason: most thermal printers lack reliable Arabic shaping in text mode; bitmap output is deterministic and previewable. Cost: slower, larger print jobs. Verification on real printers is the owner's (VERIFY_ON_WINDOWS.md).

## D-008 — Sales ↔ shifts ↔ stock sequencing across phases
Status: ACCEPTED. Phase 2 sales tables include nullable `cash_shift_id` and a documented stock-hook interface; Phase 3 adds the shift module and enforces it, and wires stock consumption/return into the existing sale and reversal transactions. Phase 2 must not invent fake shift or stock behavior.

## D-009 — Cash shift policy (default)
Status: PROPOSED — finalize in P3-02. Default: one open shift per cashier; multiple cashiers may have simultaneous shifts; setting `require_open_shift_for_sales` default ON. Closed shifts are immutable; corrections use audited adjustment records.

## D-010 — Inventory model
Status: PROPOSED — finalize in P3-06. Append-only `stock_movements` ledger is the source of truth; a cached balance per item is updated in the same transaction as each movement; a reconciliation function recomputes from the ledger and reports differences. Tracking modes per product: `none` (default), `direct item`, `recipe`. No invented recipes.

## D-011 — Accounting treatment
Status: PROPOSED — finalize in P3-11 (`docs/ACCOUNTING_RULES.md`). Purchases are recorded as stock/supplier obligations and are NOT also reported as expenses; the "ingredients and supplies" expense category is for non-stock purchases only. Stock-consumption cost appears only when cost data exists. Salary payments appear as payroll, not duplicated as generic expenses.

## D-012 — Backup format
Status: PROPOSED — finalize in P5-01. A single archive containing a SQLite snapshot (taken with the SQLite backup API or `VACUUM INTO`, never a raw copy of a live file) plus `manifest.json` (restaurant_id, restaurant name, created_at, app version, schema version, SHA-256 of the DB). Verified immediately after creation by opening the snapshot, `PRAGMA integrity_check`, and checksum.

## D-013 — Windows installer and WebView2
Status: PROPOSED — finalize in P5-11. Prefer MSI as the owner requested, but note WiX-based MSI builds only on Windows; NSIS is the fallback. For PCs without internet, configure the WebView2 install mode to an offline installer or fixed runtime. Data lives in the app-data directory; uninstall and upgrade keep it by default. Must be verified on Windows before any claim.

## D-014 — Session and lock behavior
Status: PROPOSED — finalize in P1-10. Session held in Rust memory (not browser storage); configurable inactivity auto-lock (default off for a busy cashier, or long timeout) with quick PIN/password unlock; failed-login throttling.
