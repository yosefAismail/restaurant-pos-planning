# PHASE 1 PROMPT — Foundation

You are a senior software architect, Rust/Tauri engineer, React/TypeScript engineer, database engineer, QA engineer, and Arabic RTL UX designer, continuing an offline-first Windows restaurant management system. Read `CLAUDE.md` first: it holds the rules that apply to everything below.

## Mission
Build the stable foundation: Tauri 2 + React + strict TypeScript + SQLite, money/time core, versioned migrations, first-run restaurant setup, local authentication, RBAC, audit log, Arabic RTL design system, app shell, and security baseline. Do NOT start POS, inventory, or any later business module.

## 0. Mandatory start protocol (do this BEFORE any code change)
1. Read `CLAUDE.md`, then `PROGRESS.md`, `TASKS.md`, `PROJECT_PLAN.md`, `DECISIONS.md`, and any `docs/` file relevant to this phase.
2. Inspect the actual repository: structure, `git status`, `git log -5`, installed toolchain. The repository is authoritative over notes; if they disagree, investigate and correct the notes first.
3. Run the existing build, typecheck, lint, and tests to learn the real baseline. Do not trust old "VERIFIED" marks blindly if the baseline is red.
4. Find the first task of THIS phase that is not `VERIFIED` and whose dependencies are satisfied. Resume there. Never restart the phase, recreate working files, or repeat completed work unless verification proves it broken.
5. If the previous phase has unverified acceptance criteria, either verify them now or record each as a documented blocker/deferral with impact before continuing.

## Dependencies
None (first phase). Required tooling: Node.js LTS + package manager, Rust stable toolchain, Tauri 2 prerequisites (on Windows: MSVC Build Tools, WebView2). Record what is actually available in `PROGRESS.md`. If Rust or Tauri cannot run in this environment, implement everything that can be built and unit-tested (TS code, SQL migrations, docs), mark Tauri-dependent tasks `IMPLEMENTED, NOT VERIFIED`, and say exactly what the owner must run on Windows.

## Tasks (IDs match `TASKS.md`)
**P1-01 Inspect and initialize.** Inspect the directory. If the repo is empty, `git init`. Keep the existing planning files. Create `.gitignore` (node_modules, target, dist, build output, `*.db`, `*.sqlite*`, backups, `.env*` except `.env.example`, logs). Record OS, Node, Rust, MSVC, WebView2, Git, and printer availability in `PROGRESS.md`. Do not overwrite existing user files.

**P1-02 Scaffold.** Create the Tauri 2 + Vite + React + TypeScript (`strict: true`, `noUncheckedIndexedAccess` recommended) + Tailwind project using the target structure in `CLAUDE.md` §5. Add React Router (or equivalent) and TanStack Query only if needed. Pin exact versions; record them and the reason for each non-obvious dependency in `DECISIONS.md` and `docs/BUILD_AND_RELEASE.md` (draft). No CDN, remote font, or runtime network call anywhere.

**P1-03 Tooling and test harness.** ESLint, Prettier (or Biome), Vitest + React Testing Library with an RTL-aware test helper, `cargo test`, `cargo clippy`, and npm scripts (`dev`, `build`, `typecheck`, `lint`, `test`, `test:rust`, `tauri:dev`, `tauri:build`). Add a CI-style single command `npm run verify` that runs typecheck, lint, frontend tests, and Rust tests where available.

**P1-04 Money and quantity core.** Implement in Rust (authoritative) and TypeScript (preview only): `Money` as integer piastres, `Quantity` as integer thousandths, add/subtract/multiply, percent, round-half-up, line total `round_half_up(qty_milli * unit_price / 1000)`, parsing/formatting of Arabic-Indic and Western digits, ج.م formatting. Overflow-safe (use i64 and checked ops). Create shared JSON test vectors (`tests/vectors/money.json`) consumed by BOTH Rust and TS tests. Cover rounding edges (0.5 piastre, 3 × 33.33, negative inputs rejected where invalid).

**P1-05 Time utilities.** Store timestamps in UTC; compute `business_date` in Africa/Cairo with the IANA tz database (Egypt observes DST). Provide Cairo-day range → UTC bounds helper used later by reports. Optional configurable business-day cut-off (default midnight). Tests: DST transition days, midnight boundaries, year boundaries. Record the final representation in `DECISIONS.md` (D-004).

**P1-06 SQLite integration.** Database file in the OS app-data directory under a stable app identifier (never in the repo or install folder). Rust owns the connection(s): `foreign_keys=ON`, `busy_timeout`, WAL or documented alternative, `synchronous` chosen and documented, graceful handling of busy/locked errors, error types mapped to safe Arabic-ready error codes (no raw SQL errors shown to users). Run a short spike confirming D-002 (Rust-owned DB with typed commands, backup API availability) and mark D-002 `ACCEPTED` or write a superseding decision. Support a test mode with an in-memory or temp DB.

**P1-07 Migration framework.** Versioned, ordered, repeatable migrations stored in `src-tauri/migrations/`; a `schema_migrations` table (version, name, checksum, applied_at, app_version); each migration runs in a transaction; failure rolls back and leaves the DB at the previous version with a clear error; checksum mismatch of an already-applied migration is detected and refused; a pre-migration backup hook (real implementation arrives in Phase 5, but the hook and a simple file-copy-when-closed fallback must exist and be documented); refuse to open a DB with a schema NEWER than the app understands. Tests: apply from empty, idempotent re-run, failure rollback, newer-schema refusal, checksum tamper detection.

**P1-08 Core schema (migration 0001+).** Tables at minimum: `restaurant` (single row: `restaurant_id` UUID, name, branch, address, phones, optional tax/commercial numbers, created_at, currency, timezone), `users`, `roles`, `permissions`, `role_permissions`, `user_roles` (or a single role per user, decide and document), `audit_log`, `app_settings` (key/value with typed accessors), and an `idempotency_keys`-capable pattern for later phases. Use INTEGER ids or UUIDs consistently (document), `created_at/updated_at`, CHECK constraints, UNIQUE constraints, indexes, `ON DELETE RESTRICT` for history-relevant links. Start `docs/DATABASE.md` with an entity-relationship overview (Mermaid or ASCII) of what exists.

**P1-09 First-run setup.** When the DB has no restaurant row, show an Arabic wizard: restaurant name, optional branch/address/phones/tax number/commercial number, then administrator account (name, username, password with strength rules, confirmation). Generate `restaurant_id`. No hardcoded production credentials. A dev-only seed, if any, must be behind a compile-time dev flag and verified absent from release builds. Wizard must be unskippable and idempotent (reload mid-way is safe).

**P1-10 Authentication.** Argon2id hashing (per-user salt via the library, parameters documented), login/logout, session held in Rust (not browser storage), password change (requires current password), account activation/deactivation (cannot deactivate the last active owner), login throttling/backoff on repeated failures, lock screen with quick re-authentication, optional configurable inactivity lock (D-014). Never log secrets. Tests: hash verify, wrong password, throttling, deactivated user, last-owner protection.

**P1-11 Roles and permissions.** Roles: Owner/Admin, Manager, Cashier, Inventory/Purchasing, Accountant (read-only). Define a permission catalog now covering ALL future modules (e.g. `sales.create`, `sales.price_override`, `sales.discount`, `sales.reverse`, `shifts.manage_own`, `shifts.manage_all`, `inventory.manage`, `inventory.adjust`, `purchases.manage`, `expenses.manage`, `payroll.manage`, `reports.view`, `reports.export`, `users.manage`, `backup.create`, `backup.restore`, `settings.manage`, `audit.view`), with sensible default role mappings and an admin UI to adjust them (Owner permissions can't be reduced to lock everyone out). Enforce in Rust via a central `require_permission(session, perm)` used by EVERY command; the frontend only hides UI. Tests: each role against representative commands, unauthorized call rejected even if the UI is bypassed.

**P1-12 Audit log.** `audit_service.record(actor, action, entity, entity_id, summary/before-after (no secrets), reason)` called inside the same transaction as the audited change. Viewer screen with filters (date, user, action) and pagination, permission-gated. Tests: audit row is rolled back with its transaction; no password material ever appears.

**P1-13 Arabic RTL design system.** Locally bundled Arabic-capable font (verify license permits bundling and record it). Design tokens in Tailwind. Components: Button (variants, loading, disabled-while-pending), Input, NumberInput (Arabic/Western digit tolerant), Select, Checkbox/Switch, Dialog, ConfirmDialog, Table (RTL, sortable, paginated), Badge, Alert, Card, PageLayout, EmptyState, Toast, Spinner. Use logical CSS properties (`ms-`, `me-`, `ps-`, `pe-`, `start`, `end`), mirror directional icons, correct focus order and keyboard behavior. Large readable text and touch-friendly targets. Central `src/i18n/ar.ts` for strings (no scattered literals). Component tests under RTL.

**P1-14 App shell.** RTL sidebar navigation (Arabic labels for all planned modules; modules not built yet are hidden or clearly marked "قريبًا" and disabled, never faked), top bar with restaurant name and current user, permission-aware navigation, lock screen, error boundary with Arabic message, 404 page, and a real home page that shows only truthful data available now (restaurant name, user, role, app version). No fake dashboard numbers.

**P1-15 Settings foundation.** Restaurant profile editor, users list/create/edit/deactivate with role assignment, permissions editor, password reset by admin, application information (version, schema version, DB location). Every change audited.

**P1-16 Tauri security baseline.** Restrictive CSP (no remote origins), minimal capability files, no shell plugin, no unrestricted fs plugin, only the commands the app needs. Verify no network fetches in the built frontend (grep bundle for http(s) URLs, check fonts are local). Document in `docs/SECURITY.md` (initial).

**P1-17 Tests, docs, gate.** Complete foundational unit/integration/UI tests (first-run, login, permission denial, migration, money, time). Write `README.md` (English: overview, prerequisites, setup, run, test, build, DB location, migrations), `docs/ARCHITECTURE.md`, `docs/DATABASE.md`, `docs/TESTING.md`. Mark statuses honestly.

## Expected deliverables
Working Tauri+React shell (or, if the toolchain is absent, buildable frontend + tested Rust sources + exact owner instructions); migration framework with ≥1 applied migration; first-run wizard; login; RBAC; audit; design system with Arabic font; shared money vectors; `README.md`, `docs/ARCHITECTURE.md`, `docs/DATABASE.md`, `docs/TESTING.md`, `docs/SECURITY.md` (initial), updated tracking files; Git checkpoints.

## Acceptance criteria (measurable; all must be checked against real output)
1. `npm run verify` passes (or each failing/unavailable part is documented with reason). Paste the real command and result summary into `PROGRESS.md`.
2. Fresh DB → all migrations apply in order; re-running is a no-op; a deliberately failing migration leaves the DB unchanged (automated test).
3. First-run wizard creates exactly one restaurant row with a UUID and one Owner user; reloading mid-wizard doesn't duplicate; no default credentials exist in the repo (grep evidence recorded).
4. Passwords stored only as Argon2id hashes; wrong password and throttling tests pass; deactivated users cannot log in; last Owner cannot be deactivated or demoted.
5. For every registered Tauri command there is a documented required permission, and a test proves an unauthorized role is rejected at the Rust layer.
6. Audit entries are written transactionally and contain no secrets (test).
7. Money vectors pass in BOTH Rust and TypeScript with identical results; rounding rules documented.
8. Time tests pass for Cairo DST and midnight boundaries.
9. All UI strings come from the Arabic i18n file; layout is RTL with logical properties; no remote asset in the built frontend (evidence recorded).
10. The app launches in a desktop environment (`tauri dev`) — or this is explicitly `IMPLEMENTED, NOT VERIFIED` with the owner's exact steps in `docs/VERIFY_ON_WINDOWS.md`.
11. `PROGRESS.md`/`TASKS.md` match the repository.

## Out of scope for this phase
Sales, products, receipts, printing, shifts, inventory, purchases, expenses, payroll, reports, backup/restore UI.

## Verification limits
Visual RTL correctness inside WebView2 and Tauri dev-run on Windows may need the owner's machine. Record exactly what was and wasn't run.

## Session end protocol (mandatory, every session)
Stop starting large tasks when context is running low. Checkpoint the current small task so nothing is half-edited. Run the most relevant verification. Then update:
- `PROGRESS.md`: current phase, last completed task, in-progress task, exact files changed, commands actually executed, actual results, known bugs/limitations, migration/data-safety concerns, **the next concrete task ID**, and the resume command.
- `TASKS.md` statuses (only `VERIFIED` when the verification was really run), `CHANGELOG.md`, `DECISIONS.md`, and affected docs.
- Commit a logical Git checkpoint (no secrets, no real databases, no private business data).
Finish with a concise handover: what was implemented, what was verified (with the commands and results), what remains, blockers/risks, and the exact next task. Do not claim this phase is complete unless every acceptance criterion above has been checked against the real code and real test output.
