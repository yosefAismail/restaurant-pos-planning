# PROGRESS.md

> Single source of truth for "where are we". Update before ending EVERY session. If this file disagrees with the repository, the repository wins: investigate, then fix this file.

## Current state
- **Current phase:** Phase 1 — Foundation
- **Overall status:** IN PROGRESS (P1-01 done; no application code yet)
- **Last completed task:** `P1-01` — `git init`, `.gitignore`, toolchain install/record
- **In-progress task:** none
- **Next concrete task:** `P1-02` — scaffold Tauri 2 + React + strict TS + Vite + Tailwind; pin versions
- **Resume command:** open the repo root, then paste `docs/prompts/phase-1-foundation.md` (or `docs/prompts/RESUME.md`) and continue at P1-02.

## Environment (fill in during P1-01; do not guess)
| Item | Value |
|---|---|
| OS | Windows 11 21H2, build 22000 (registry `ProductName`: Windows 10 Pro Education; `CurrentBuild` 22000). Machine: win32 10.0.22000 |
| Node / package manager | Node.js LTS **v24.20.0** (`OpenJS.NodeJS.LTS`), npm **11.19.0** (`C:\Program Files\nodejs\`) |
| Rust toolchain / MSVC build tools | rustup **1.29.1**; default host/toolchain **stable-x86_64-pc-windows-msvc**; rustc **1.99.1** (58cf8f9c3 2026-10-13); cargo **1.99.1**; rustfmt + clippy installed. Visual Studio Build Tools **2022 17.14.42** (`C:\Program Files (x86)\Microsoft Visual Studio\2022\BuildTools`); MSVC `cl` **19.44.35229** x64; Windows 11 SDK **10.0.22621**. `rustc` hello-world linked and ran (`msvc-ok`). |
| WebView2 | **Installed and usable.** Runtime exe: `C:\Program Files (x86)\Microsoft\EdgeWebView\Application\148.0.3967.96\msedgewebview2.exe` (148.0.3967.96). winget lists `Microsoft.EdgeWebView2Runtime` as **149.0.4022.52** with **155.0.4283.62** available. First `winget install` reported success; a later `winget upgrade` failed with installer exit **4294967295** (likely elevation). Evergreen 148/149 is enough for Tauri; 155 upgrade not completed. |
| Git available | **Yes** — git **2.53.0.windows.2**. Repo initialized at `C:/Users/POP/Downloads/cashier/restaurant-pos-planning-kit/.git/` on branch `master`. No commits yet. |
| Can build Windows installer here? | Native compile toolchain is present. **Installer packaging not verified.** WiX/NSIS not installed. Do not claim MSI/NSIS until P5. |
| Printer available for testing? | Windows printers present: **HP LaserJet Professional P1102** (USB001), Microsoft Print to PDF, XPS, Fax, WPS PDF. **No thermal 58/80 mm POS printer identified by name.** Receipt print tests remain owner/hardware work. |

## Verification log (append only; real commands and real results)
- 2026-10-19 — `git init` → `Initialized empty Git repository in C:/Users/POP/Downloads/cashier/restaurant-pos-planning-kit/.git/`
- 2026-10-19 — `winget install --id OpenJS.NodeJS.LTS` → upgraded to **24.20.0**, exit 0
- 2026-10-19 — `winget install --id Microsoft.EdgeWebView2Runtime` → winget reported Successfully installed (exit 0); on-disk runtime remained **148.0.3967.96**
- 2026-10-19 — `winget install --id Microsoft.VisualStudio.2022.BuildTools` with VCTools + Win11 SDK 22621 → Successfully installed **17.14.42**, exit 0 (~17 min)
- 2026-10-19 — `winget install --id Rustlang.Rustup` → Successfully installed **1.29.1**, exit 0
- 2026-10-19 — `rustup default stable-x86_64-pc-windows-msvc` → rustc 1.99.1; `rustc main.rs` hello-world printed `msvc-ok`
- 2026-10-19 — `winget upgrade --id Microsoft.EdgeWebView2Runtime` → **Installer failed** exit 4294967295
- 2026-10-19 — `git status` → empty repo, untracked planning files + `.gitignore`; `git log` empty (no commits)

## Known issues / limitations
- WebView2 is present (148/149) but not upgraded to winget’s 155.0.4283.62; upgrade installer failed after UAC/elevation.
- No Git commits yet (owner-requested installs only; commit when asked).
- No application source yet. `npm run verify` / `tauri dev` cannot be run.
- Thermal receipt printer not present (laser USB printer only).

## Files changed in last session
`.gitignore` (created), `PROGRESS.md`, `TASKS.md`, `CHANGELOG.md`. Git repository initialized. No application code.

## Data-safety / migration concerns
None yet. No SQLite database created.

## Items requiring the owner's Windows computer or real printer
See `PROJECT_PLAN.md` §5. Toolchain is now on this Windows machine. Remaining: approve a WebView2 155 upgrade if desired; thermal printer; first `tauri dev` visual check; installer packaging (WiX/NSIS) in Phase 5.
