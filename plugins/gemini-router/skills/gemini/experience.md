# Gemini Experience Log

Claude reads this file before every potential Gemini delegation.
Entries are written automatically based on user decisions and feedback.

Two entry types:
- `[AUTO_APPROVE]` — always route to Gemini, skip the confirmation prompt
- `[AVOID]` — Gemini underperformed here, handle with Claude instead

---

## Baseline Knowledge (2026-04-08)

### [AVOID] 2026-04-08
**Pattern**: File writing, editing, or code generation
**What went wrong**: Gemini headless is single-shot and cannot iterate on edits
**Lesson**: Any task requiring Write/Edit tool use stays with Claude

### [AVOID] 2026-04-08
**Pattern**: Iterative tool-use loops (e.g., run → inspect → fix → repeat)
**What went wrong**: Gemini -p mode cannot loop or react to intermediate results
**Lesson**: Keep multi-step agentic tasks with Claude

---
<!-- New entries are appended below this line. Do not remove this comment -->
