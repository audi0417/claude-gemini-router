# Claude Gemini Router

A Claude Code plugin that intelligently routes tasks to Google's Gemini CLI based on task characteristics, saving Claude tokens for large-context operations.

## Features

- **Smart task scoring** — automatically evaluates whether a task is better suited for Gemini (large file analysis, summarization) or Claude (code editing, iterative workflows)
- **Preference learning** — remembers your choices via an experience log, auto-routing familiar task types without asking
- **Dissatisfaction detection** — watches for negative feedback and learns to avoid routing similar tasks to Gemini in the future

## Prerequisites

- [Claude Code](https://claude.ai/code) installed and authenticated
- [Gemini CLI](https://github.com/google-gemini/gemini-cli) installed (`npm install -g @google/gemini-cli`)

## Installation

### As a Plugin (recommended)

```bash
# Install from GitHub
claude /plugin install https://github.com/audi0417/claude-gemini-router
```

### As a Standalone Skill

Copy the `skills/gemini/` directory to your Claude config:

```bash
cp -r skills/gemini ~/.claude/skills/gemini
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

### Disqualifiers (always use Claude)

- File writing or editing
- Iterative tool-use loops
- Short, simple tasks

## License

[MIT](LICENSE)
