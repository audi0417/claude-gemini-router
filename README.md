# Claude Gemini Router

> **[繁體中文](README_zh-TW.md)** | **[简体中文](README_zh-CN.md)**

A self-evolving Claude Code plugin that intelligently routes tasks to Google's Gemini CLI based on task characteristics, saving Claude tokens for large-context operations.

**The more you use it, the smarter it gets** — the system remembers your preferences and continuously evolves its decision logic.

## Features

- **Smart task scoring** — automatically evaluates whether a task is better suited for Gemini (large file analysis, summarization) or Claude (code editing, iterative workflows)
- **Preference learning** — remembers your choices via an experience log, auto-routing familiar task types without asking
- **Dissatisfaction detection** — watches for negative feedback and learns to avoid routing similar tasks to Gemini in the future
- **Self-evolution** — implements the same Experience-driven Lifelong Learning (ELL) principles as the AutoSkill research

---

## Self-Evolving Learning Loop

```
User Request → Task Scoring → Experience Log Lookup
                                    ↓
                          ┌─ AUTO_APPROVE → Delegate to Gemini directly
                          ├─ AVOID → Claude handles it
                          └─ Unknown → Ask user
                                              ↓
                                    ┌─ Remember & use → Write AUTO_APPROVE
                                    ├─ Just this once → No record
                                    └─ Claude handles → No record
                                              ↓
                          User dissatisfied? → Write AVOID, auto-skip in future
```

### How This Is "Self-Evolving"

This skill implements the same core principles as **AutoSkill** (Yang et al., 2026, arXiv:2603.01145) — an **Experience-driven Lifelong Learning (ELL)** framework:

| AutoSkill Concept | Gemini Router Implementation |
|-------------------|------------------------------|
| **Experience Detection** — monitor interactions for stable preferences | Scoring system detects task characteristics (file size, read/write, complexity) |
| **Pattern Extraction** — extract reusable skills from feedback | When user chooses "remember", task pattern is written as `AUTO_APPROVE` entry |
| **Version Evolution** — skills update with continued feedback | `AVOID` can override `AUTO_APPROVE`, decision logic continuously self-corrects |
| **Skill Application** — apply learned skills to future tasks | Tasks matching `AUTO_APPROVE` are delegated instantly, zero confirmation |

**Key shared principles:**
- **Human oversight** — both systems ask user consent before recording, never act unilaterally
- **Bidirectional learning** — learns from both success (AUTO_APPROVE) and failure (AVOID)
- **Persistent memory** — experience survives across conversations, never "use and forget"
- **Progressive evolution** — decision accuracy improves with each use

---

## Prerequisites

- [Claude Code](https://claude.ai/code) installed and authenticated
- [Gemini CLI](https://github.com/google-gemini/gemini-cli) installed (`npm install -g @google/gemini-cli`)

## Installation

### Step 1: Add the marketplace

In Claude Code, run:

```
/plugin marketplace add audi0417/claude-gemini-router
```

### Step 2: Install the plugin

```
/plugin install gemini-router@claude-gemini-router
```

Or use the interactive plugin manager:

```
/plugin
```

Then navigate to **Discover** tab and select **gemini-router**.

### Alternative: Manual install

Copy the skill files directly:

```bash
git clone https://github.com/audi0417/claude-gemini-router.git
cp -r claude-gemini-router/plugins/gemini-router/skills/gemini ~/.claude/skills/gemini
```

## How It Works

1. **Evaluate** — When you give Claude a task, the skill scores it for Gemini suitability
2. **Decide** — High-scoring tasks prompt you to choose: delegate to Gemini, or keep with Claude
3. **Learn** — Your choices are logged so similar tasks are auto-routed next time
4. **Adapt** — If Gemini's output is unsatisfactory, the skill learns to avoid that task type

### Scoring Criteria

| Signal | Weight |
|--------|--------|
| Large files (>50KB total) | +2 |
| Read-only / analysis task | +2 |
| Matches AUTO_APPROVE pattern | +2 |
| User explicitly requests Gemini | +2 |
| Multi-file cross-reference | +1 |
| Summarization / extraction | +1 |
| Token conservation benefits | +1 |

### Disqualifiers (always use Claude)

- Matches an AVOID pattern in experience log
- File writing or editing
- Iterative tool-use loops
- Short, simple tasks

### Experience Log (experience.md)

All learning outcomes are stored in `experience.md` — transparent and human-readable:

```markdown
### [AUTO_APPROVE] 2026-04-08
**Pattern**: Full Go codebase read-only analysis
**Trigger keywords**: codebase, analysis, review
**Reason**: Large file volume, read-only, Gemini 1M context is a perfect fit

### [AVOID] 2026-04-08
**Pattern**: File writing, editing, or code generation
**What went wrong**: Gemini headless is single-shot, cannot iterate on edits
**Lesson**: Any task requiring Write/Edit tool use stays with Claude
```

---

## Usage Examples

### Auto-delegation (learned pattern)
```
You: Analyze the architecture across these 20 files
Claude: Routing to Gemini (auto-approved: large codebase analysis)
[Gemini Response] ...
```

### First encounter (asks your preference)
```
You: Read this 200KB log file and find the errors
Claude: This task looks like a good fit for Gemini (large context). How should I handle it?
  1. Use Gemini & remember this task type
  2. Use Gemini, just this once
  3. Claude handles it
```

### Dissatisfaction feedback
```
You: Gemini's analysis was off, redo this yourself
Claude: Should I mark this task type as a poor fit for Gemini?
  1. Yes, log it
  2. No, just this once
```

---

## License

[MIT](LICENSE)
