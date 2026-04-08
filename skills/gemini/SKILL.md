---
name: gemini
description: Smart Gemini CLI delegation skill. Activate this skill at the START of processing any user request — before reading files or starting work — to evaluate whether the task should be routed to Gemini. Always check experience.md first for auto-approve patterns. If unsure, use AskUserQuestion to let the user decide. Do NOT activate for casual conversation or tiny tasks.
version: 2.0.0
allowed-tools: [Bash, Read, Write, AskUserQuestion]
---

# Gemini Smart Delegation Skill

Route tasks to Google's Gemini CLI (1M token context window) intelligently. The key behavior:
**always evaluate FIRST, then decide — never start a big task without checking**.

---

## Step 1: Read the Experience Log

Before doing anything else, read the experience log:

```bash
cat ~/.claude/skills/gemini/experience.md
```

The log contains:
- `[AUTO_APPROVE]` entries — task types to always send to Gemini, no prompt needed
- `[AVOID]` entries — task types where Gemini underperformed, handle with Claude instead

---

## Step 2: Score the Task

### Strong Gemini signals (+2 each)
- Files involved are likely >50KB total (codebase scan, large logs, big documents)
- Task is purely read/analysis — no file editing needed
- Task matches an `[AUTO_APPROVE]` pattern in experience.md
- User explicitly says "use gemini", "ask gemini", "let gemini handle"

### Mild Gemini signals (+1 each)
- Multiple files need to be read and cross-referenced
- Task involves summarization or extraction from long content
- Token conservation would meaningfully benefit the conversation

### Disqualifiers (skip Gemini entirely)
- Task matches an `[AVOID]` pattern in experience.md → handle with Claude, mention why
- Task requires writing or editing files
- Task requires iterative tool-use loops
- Task is short and simple

---

## Step 3: Decision Branch

### Score >= 2 AND matches an [AUTO_APPROVE] pattern:
→ **Go straight to Gemini** — skip the question
→ Briefly note: "Routing to Gemini (auto-approved: [pattern name])"

### Score >= 2 but NO prior auto-approve for this pattern:
→ **Use AskUserQuestion** with this exact structure:

```
Question: "This task looks like a good fit for Gemini (large context / token savings). How should I handle it?"
Header: "Route to Gemini?"
Options:
  1. label: "Use Gemini & remember this task type"
     description: "Delegate to Gemini now, and auto-route similar tasks in the future without asking"
  2. label: "Use Gemini, just this once"
     description: "Delegate to Gemini now, but ask again next time"
  3. label: "Claude handles it"
     description: "Skip Gemini for this task"
```

### Score < 2:
→ Handle with Claude normally, no question needed

---

## Step 4: After the User Chooses

### Option 1 — "Use Gemini & remember this task type":
1. Append an `[AUTO_APPROVE]` entry to experience.md (format below)
2. Run Gemini
3. Present output labeled `[Gemini Response]`

### Option 2 — "Use Gemini, just this once":
1. Run Gemini
2. Present output labeled `[Gemini Response]`
3. Do NOT write to experience.md

### Option 3 — "Claude handles it":
1. Handle with Claude normally
2. Do NOT write to experience.md

---

## Step 5: Running Gemini

### Standard headless call
```bash
gemini -p "COMPLETE SELF-CONTAINED PROMPT" --output-format text --yolo 2>&1
```

### Pipe large file content
```bash
cat /path/to/large/file | gemini -p "YOUR INSTRUCTION" --output-format text --yolo 2>&1
```

### Multi-file analysis
```bash
gemini -p "$(printf 'Analyze these files:\n\n=== file1 ===\n'; cat file1; printf '\n\n=== file2 ===\n'; cat file2; printf '\n\nTask: YOUR INSTRUCTION')" --output-format text --yolo 2>&1
```

**Important**: Gemini headless is **single-shot** — craft a complete, self-contained prompt with all needed context included.

---

## Step 6: Post-Result Dissatisfaction Detection

After presenting Gemini output, watch the user's next message for dissatisfaction signals:

**Signals**: "wrong", "not good", "that's off", "redo this", user corrects the output, user asks Claude to redo the same task

**When detected**:
1. Use AskUserQuestion:
   ```
   Question: "Should I mark this task type as a poor fit for Gemini and handle it with Claude going forward?"
   Options:
     - "Yes, log it" — description: "Record this as an [AVOID] pattern"
     - "No, just this once" — description: "Don't log anything"
   ```
2. If user picks "Yes, log it" → append an `[AVOID]` entry to experience.md
3. If this task type had an existing `[AUTO_APPROVE]` entry, update it to `[AVOID]`
4. Confirm: "Noted. I'll handle this type of task myself from now on."

---

## Experience Log Entry Formats

### AUTO_APPROVE entry
```markdown
---
### [AUTO_APPROVE] YYYY-MM-DD
**Pattern**: one-line description of the task type (e.g., "Full Go codebase read-only analysis")
**Trigger keywords**: keywords that identify this pattern
**Reason**: why Gemini is well-suited for this
```

### AVOID entry
```markdown
---
### [AVOID] YYYY-MM-DD
**Pattern**: one-line description of the task type
**What went wrong**: brief failure description from user feedback
**Lesson**: what to do instead next time
```

---

## Decision Table

| Condition | Action |
|-----------|--------|
| Matches [AUTO_APPROVE] | Go straight to Gemini, no question |
| Matches [AVOID] | Use Claude, explain why |
| Large/complex, no prior record | AskUserQuestion (3 options) |
| Small/simple task | Claude handles, no question |
| User picks "remember this type" | Write AUTO_APPROVE + run Gemini |
| User flags poor result | AskUserQuestion → write AVOID |
