# How to use the phase prompts

Files here:
| File | Use |
|---|---|
| `phase-1-foundation.md` | Paste into Claude Code first, in a repo that contains this planning kit |
| `phase-2-pos-receipts.md` | After Phase 1 is verified |
| `phase-3-cash-inventory.md` | After Phase 2 is verified |
| `phase-4-staff-reports.md` | After Phase 3 is verified |
| `phase-5-safety-delivery.md` | After Phase 4 is verified |
| `RESUME.md` | Any time a session ends mid-phase or you start a new session |

## Setup (once)
1. Create an empty folder for the project and open it in VS Code (or a terminal).
2. Copy the contents of this kit into the folder root (`CLAUDE.md`, `PROJECT_PLAN.md`, `PROGRESS.md`, `TASKS.md`, `DECISIONS.md`, `CHANGELOG.md`, and `docs/prompts/`).
3. `git init` and commit the planning files (Phase 1 does this too if you skip it).
4. Install prerequisites on your Windows PC: Node.js LTS, Rust (stable, MSVC toolchain), Visual Studio Build Tools (C++), WebView2 runtime. Claude Code can run on Windows directly or in WSL, but build and test the Windows installer natively on Windows.
5. Start Claude Code in the folder (`claude`). It reads `CLAUDE.md` automatically.

## Running a phase
- Start a fresh session, then paste the full text of the phase prompt. A phase usually needs several sessions: when one ends, start a new session and paste `RESUME.md`.
- Don't move to the next phase until `PROGRESS.md` shows the previous phase's acceptance criteria `VERIFIED` or documented.
- Keep each session focused on one phase. Prefer a fresh session over a huge context.
- You can check status any time by opening `PROGRESS.md` and `TASKS.md`.

## What only you can verify
Installer build/install/upgrade/uninstall, offline install, physical thermal printing, USB restore across two PCs. Claude must leave exact checklists in `docs/VERIFY_ON_WINDOWS.md`; run them and tell Claude the results so statuses can change to `VERIFIED`.
