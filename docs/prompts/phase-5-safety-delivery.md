# PHASE 5 PROMPT — Backup/Restore, Security Review, Integration Tests, Windows Packaging, Documentation, Final Audit

You are a senior reliability/security engineer, release engineer, QA lead, and technical writer finishing the offline restaurant management system. `CLAUDE.md` rules apply to everything below.

## Mission
Make the system safe for real restaurant data: verified offline backup and restore (including USB transfer between computers), upgrade-safe migrations, a security and permission review, complete automated test coverage of critical flows, Windows installer configuration, finished documentation, and an honest final acceptance audit.

## Prerequisite check
Phases 1–4 `VERIFIED` (or each gap documented with impact). Run the entire test suite first. Any red test is fixed before new work.

## 0. Mandatory start protocol (do this BEFORE any code change)
1. Read `CLAUDE.md`, then `PROGRESS.md`, `TASKS.md`, `PROJECT_PLAN.md`, `DECISIONS.md`, and any `docs/` file relevant to this phase.
2. Inspect the actual repository: structure, `git status`, `git log -5`, installed toolchain. The repository is authoritative over notes; if they disagree, investigate and correct the notes first.
3. Run the existing build, typecheck, lint, and tests to learn the real baseline. Do not trust old "VERIFIED" marks blindly if the baseline is red.
4. Find the first task of THIS phase that is not `VERIFIED` and whose dependencies are satisfied. Resume there. Never restart the phase, recreate working files, or repeat completed work unless verification proves it broken.
5. If the previous phase has unverified acceptance criteria, either verify them now or record each as a documented blocker/deferral with impact before continuing.

## Dependencies
P1-07 migration framework and backup hook, all schemas from Phases 1–4, `restaurant_id` from P1-09, permissions `backup.create` / `backup.restore`.

## Tasks (IDs match `TASKS.md`)
**P5-01 Backup service.** Create a consistent snapshot using SQLite's online backup API or `VACUUM INTO` (never a raw copy of a live DB). Package as the documented format (D-012): archive containing the snapshot and `manifest.json` (format version, restaurant_id, restaurant name, created_at, app version, schema version, SHA-256, record counts as a sanity check). Immediately verify: reopen the snapshot read-only, `PRAGMA integrity_check` and `foreign_key_check`, checksum match. A backup is reported successful only if verification passed; otherwise delete the bad file and show an Arabic error. Handle low disk space, destination unplugged mid-write, and permission errors. Runs without blocking the UI; progress shown. Audit entry. Write the format spec to `docs/BACKUP_AND_RESTORE.md`.

**P5-02 Backup UI and preferences.** Backup screen: create manual backup, choose destination folder/USB (via Tauri dialog with scoped access), last successful backup time, latest attempt result, restaurant identity and DB version, list of recent backups, retention preference with recommendations (e.g. keep several rotating backups and one off-site USB), reminder when the last backup is older than a configurable threshold (shown to Owner/Manager at login and on the home page). Scheduled local backup (to a local folder) only if it can be done safely offline without UI blocking; otherwise `DEFERRED` with reasoning. Never claim data is backed up unless verification passed.

**P5-03 Restore.** Full validation chain: (1) file readable and archive structure valid, (2) manifest parsed, (3) checksum matches, (4) `integrity_check` passes on the snapshot, (5) restaurant_id compared with the current DB, (6) schema version compatibility (older → migrate a COPY; newer → refuse), (7) show restaurant name/ID, backup date, record counts, and exactly what current data will be replaced, (8) explicit Arabic confirmation (type-to-confirm, extra warning when restaurant_id differs and an explicit "I understand this is a different restaurant" path, or refuse entirely — pick and document; default refuse for mismatches unless the Owner confirms), (9) create a recovery backup of the current DB first, (10) swap safely (restore into a temp file, validate, then atomically replace; close/reopen connections properly), (11) validate the restored DB and complete initialization, (12) on ANY failure keep the current DB intact and report. Permission `backup.restore` (Owner only by default). No automatic merging, ever. Audit entry (written into the restored DB and noted in a restore log). Tests for each failure branch.

**P5-04 USB cross-computer restore.** Document and test the flow of creating a backup on PC A, moving it by USB, and restoring on PC B with a fresh install: first-run on PC B, then restore, with identity handling for a brand-new empty install (allow restore into a fresh uninitialized DB without false mismatch). Simulate with two app-data directories in automated tests; the physical two-PC USB test is the owner's checklist in `docs/VERIFY_ON_WINDOWS.md`. Mark `IMPLEMENTED, NOT VERIFIED` for the physical part.

**P5-05 Migration upgrade tests.** Keep schema fixtures for every released schema version (small DB files or SQL scripts with representative data from each phase). Test upgrade from each old version to current preserving data; failed migration → rollback and recovery backup used per the documented recovery path; newer-schema refusal; pre-migration automatic backup integrated with the P5-01 service. Document how to add a migration safely in `docs/DATABASE.md`.

**P5-06 Security and permission review.** Systematically list every Tauri command with its required permission and audit behavior (table in `docs/SECURITY.md`); test each with an unauthorized role; confirm parameterized SQL everywhere (search for string-built SQL); confirm CSP, capabilities, no shell/fs exposure, no network calls, no secrets in logs or audit; review lock screen/session behavior; review that backups contain business data and recommend protecting USB drives (optional backup password/encryption is a documented option or `DEFERRED` — do not invent weak crypto). Document the limits of local security (admin access, disk encryption, Windows account protection). Fix findings and record them.

**P5-07 Integration and failure scenarios.** Ensure automated coverage of the original testing list: sale completion, rollback on failure, duplicate prevention, stock deduction/reversal, purchase and stock update, expense, shift close, payroll finalization, migrations, backup/restore, restaurant mismatch, historical preservation after settings changes; failure cases: double-click completion, invalid quantities/prices, insufficient payment, DB write failure, printer unavailable, printer failure after sale, invalid backup, low disk space where testable, restore failure, restart after committed sale (simulate crash after commit), migration failure, unauthorized access. Produce a coverage matrix in `docs/TESTING.md` mapping each scenario to a test name.

**P5-08 UI/workflow tests.** Primary Arabic RTL flows: first-run setup, login, manual sale, catalog sale, completion, receipt preview and print workflow (mock printer), reprint, open/close shift, add purchase, record expense, create advance, run report, create and restore backup.

**P5-09 RTL verification pass.** Review every screen for RTL correctness (logical properties, mirrored icons, table alignment, pagination direction, dialogs, charts, number/date presentation, focus order, keyboard). Fix defects. Record screenshots or notes; visual confirmation inside WebView2 on Windows is the owner's checklist.

**P5-10 Performance pass.** Seed a large dataset (e.g. 100k sales, 300k sale items, 50k stock movements), measure key queries and screens (POS open, sale completion, history search, dashboard, reports, backup time), add or fix indexes, confirm pagination and non-blocking long tasks, record numbers in `docs/TESTING.md`. No speculative caching.

**P5-11 Windows packaging configuration.** Configure `tauri.conf.json`: product name (Arabic-friendly display with ASCII-safe identifier), version, icons, desktop shortcut, installer target (prefer MSI per owner request — note WiX builds only on Windows; NSIS as documented alternative), per-user vs per-machine install decision documented, WebView2 install mode suitable for offline PCs (offline installer or fixed runtime; record installer size trade-off), upgrade-in-place keeping data, uninstall keeping data by default (any "delete data" option must be explicit and default OFF), data stays in the app-data directory. Write the data-retention policy for uninstall and upgrade in `docs/BUILD_AND_RELEASE.md`.

**P5-12 Build installer.** If the environment is Windows with the required toolchain: run the release build, record the exact command, artifact path, size, and checksum. If not: do NOT claim success; write exact Windows build commands and a full install/upgrade/uninstall/offline checklist in `docs/VERIFY_ON_WINDOWS.md` (install on a PC with no internet, shortcut and icon, first-run setup, create sale, print test, restart, upgrade with data present, uninstall and confirm data kept, restore backup). Status `IMPLEMENTED, NOT VERIFIED` until the owner reports results.

**P5-13 Final documentation.** Complete `README.md` (overview, features, architecture, prerequisites, dev setup, run, test, build installer, DB location, migrations, backup/restore, printer configuration, known limitations, troubleshooting) and `docs/ARCHITECTURE.md`, `DATABASE.md`, `BACKUP_AND_RESTORE.md`, `RECEIPT_PRINTING.md`, `BUILD_AND_RELEASE.md`, `TESTING.md`, `SECURITY.md`, `ACCOUNTING_RULES.md`, `REPORT_DEFINITIONS.md`, `VERIFY_ON_WINDOWS.md`. Include how-tos: add a module, add a field via migration, add a report, modify the receipt template, safely inspect/export SQLite read-only, upgrade without losing data. Also a short Arabic user guide for cashiers/owners (`docs/USER_GUIDE_AR.md`) covering sale, print, shift close, backup. Docs must match the code; unbuilt or unverified features are labeled as such.

**P5-14 Final acceptance audit.** Walk the original acceptance list (below) item by item against code, tests, and verification logs. For each: `VERIFIED` (evidence), `IMPLEMENTED, NOT VERIFIED` (what remains), or gap. Produce `docs/FINAL_AUDIT.md` and an honest final report stating: what is complete, what was tested, what was not, whether the Windows installer was actually built and installed, whether physical thermal printing was verified, and what remains before real restaurant use. Do not declare the project complete while any critical item is unverified.

## Final acceptance checklist (audit each)
Launches as a Windows desktop app · works offline after install · first-run for an independent restaurant · Arabic RTL UI · sale without catalog · sale stored with unique receipt number · print and reprint without duplicate sale · per-restaurant receipt customization · thermal printers configurable (hardware-verified or flagged) · shifts and reconciliation · inventory/purchases/suppliers/stock consistency · expenses recorded and reported · advances and payroll consistent · historical reports reflect transactions and reversals · backup/restore offline · USB restore on another computer · invalid backups and identity mismatches handled safely · migrations preserve data · sensitive operations permission-protected · automated tests cover critical logic · documentation covers development, maintenance, backup, restore, release · full source maintainable · no critical workflow relies on fake data, cloud, or placeholders · progress recoverable from the repo · every phase complete or with a documented blocker.

## Acceptance criteria (measurable)
1. Backup→verify→restore round trip on fixture DBs restores byte-for-byte equal logical content (row counts and a content hash of key tables match).
2. Corrupted archive, wrong checksum, truncated file, newer schema, and different `restaurant_id` each fail safely with the current DB untouched (one automated test per case).
3. Restore always creates a recovery backup first; simulated failure mid-restore leaves the original DB intact and usable.
4. Upgrade tests pass from every historical schema fixture with data preserved.
5. Command-permission table in `docs/SECURITY.md` has 100% coverage of registered commands and each has an unauthorized-role test.
6. The coverage matrix maps every original failure scenario to a passing test or an explicit documented reason it can't be automated.
7. Performance numbers recorded; no main workflow regressed beyond documented thresholds.
8. `npm run verify` and `cargo test`/`clippy` pass; real output recorded in `PROGRESS.md`.
9. Installer built and tested on Windows ONLY if actually done; otherwise status and owner checklist are explicit and the final report says so plainly.
10. `docs/FINAL_AUDIT.md` exists and is consistent with `TASKS.md` and the repository.

## Out of scope
Cloud sync, multi-branch consolidation, internet auto-update, payment-provider integration, e-invoice integration.

## Verification limits
Installer build/install/upgrade/uninstall, offline-PC behavior, USB between two PCs, and physical thermal printing can only be verified on the owner's Windows hardware.

## Session end protocol (mandatory, every session)
Stop starting large tasks when context is running low. Checkpoint the current small task so nothing is half-edited. Run the most relevant verification. Then update:
- `PROGRESS.md`: current phase, last completed task, in-progress task, exact files changed, commands actually executed, actual results, known bugs/limitations, migration/data-safety concerns, **the next concrete task ID**, and the resume command.
- `TASKS.md` statuses (only `VERIFIED` when the verification was really run), `CHANGELOG.md`, `DECISIONS.md`, and affected docs.
- Commit a logical Git checkpoint (no secrets, no real databases, no private business data).
Finish with a concise handover: what was implemented, what was verified (with the commands and results), what remains, blockers/risks, and the exact next task. Do not claim this phase is complete unless every acceptance criterion above has been checked against the real code and real test output.
