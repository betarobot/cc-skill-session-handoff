# cc-skill-session-handoff

A community skill for [Claude Code](https://claude.ai/code) — not affiliated with or endorsed by Anthropic. This extends Claude Code's skill/plugin system with a session handoff workflow.

## What it does

Synthesizes a conversation into a structured handoff document so work can resume in a new session without re-explaining context.

### Write mode

Triggered by: "handoff", "save session", "I need to stop", "we're done for today"

Writes `~/.claude/handoffs/PROJECT-TOPIC-YYYY-MM-DD-HHMM.md` containing:
- Resume prompt — paste into a new session to continue instantly
- Summary, work done, key decisions, files touched, next steps, blockers

### List mode

Triggered by: "list handoffs", "show handoffs", "open a handoff"

Lists saved handoffs newest-first, prompts you to pick one, shows its resume prompt.

## Installation

```bash
git clone https://github.com/betarobot/cc-skill-session-handoff ~/.claude/skills/session-handoff
```

Claude Code discovers skills automatically from `~/.claude/skills/`.

## Usage

Just talk naturally — Claude will invoke the skill when it detects handoff intent. Or explicitly:

```
/session-handoff
/session-handoff list
```
