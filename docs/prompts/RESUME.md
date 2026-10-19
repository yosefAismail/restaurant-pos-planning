# RESUME PROMPT — use for "continue", "resume", "next phase", "كمل", "كمل من آخر نقطة"

Paste this at the start of ANY later Claude Code session (new session, after a context limit, or after a break). It does not replace the phase prompts; it resumes the phase already in progress.

---
Resume the offline restaurant management project. Do NOT restart, re-plan, or re-send the roadmap.

1. Read `CLAUDE.md`, `PROGRESS.md`, `TASKS.md`, `PROJECT_PLAN.md`, `DECISIONS.md`, and the docs relevant to the current phase.
2. Inspect the real repository: `git status`, `git log -10`, directory structure, and compare with what `PROGRESS.md` claims. The repository wins over notes; fix discrepancies in the notes first.
3. Run the existing verification (`npm run verify`, `cargo test`, `cargo clippy` where available) and report the true baseline in two or three lines.
4. Open the phase prompt named in `PROGRESS.md` ("Current phase") under `docs/prompts/` and re-read its task list and acceptance criteria.
5. Continue from the first task that is not `VERIFIED` and whose dependencies are met. Do not redo `VERIFIED` work unless step 3 shows it is broken.
6. Work in small batches: implement → test → fix → update docs → update `PROGRESS.md`/`TASKS.md`/`CHANGELOG.md` → commit.
7. If the current phase's acceptance criteria are all `VERIFIED` (or each gap documented), say so, run the phase gate review, and only then proceed to the next phase prompt.
8. Before stopping, follow the session end protocol in the phase prompt: record the exact next task ID and resume command in `PROGRESS.md`, then give me a concise handover.
Never claim anything is verified unless you actually ran it in this environment, and clearly list anything that needs my Windows PC or printer.
---
