---
name: session-handoff
description: >
  Summarizes the current conversation into a structured handoff document and writes it to
  `~/.claude/handoffs/PROJECT-TOPIC-YYYY-MM-DD-HHMM.md`. Use this skill
  whenever the user says "handoff", "save session", "write a handoff", "summarize for next
  session", "context dump", "I need to stop", or anything implying they want to preserve
  the current session state for later continuation. Also trigger proactively when a long
  session wraps up a major feature or decision, or when the user says "we're done for today".
  Also use when the user says "list handoffs", "ls handoffs", "show handoffs", or "open a handoff".
---

# Session Handoff

## Detect Mode

**If the user invoked this skill with `list` or `ls` (e.g. "list handoffs", "ls handoffs", "show handoffs", "open a handoff"):**
→ Jump to **List Mode** below. Skip all other steps.

**Otherwise:** proceed with Write Mode (Steps 1–4).

---

## Write Mode

The user wants to preserve the current session's context so they (or a future agent) can
continue seamlessly without re-explaining everything. Your job is to synthesize the
conversation into a durable, skimmable document — not a transcript, but a curated brief.

### What makes a great handoff

A handoff is useful when someone picks it up cold. Ask yourself: could a competent developer
resume this work using only this document? If they'd still need to ask "but what file?" or
"which approach did we pick?" — add that. If it's background any developer would know,
cut it.

Prioritize **hard-to-rediscover facts**: file paths, config values, the reason an approach
was rejected, exact error messages, the specific function that was the root cause. Skip
general knowledge about frameworks, languages, or libraries.

**Stay on the main thread.** Sessions often drift into quick tangential changes — a server
tweak, an env var fix, a config adjustment unrelated to the core work. Omit these unless
they directly affect the main topic. The handoff should reflect what the session was
*about*, not everything that happened in it.

### Step 1: Derive slugs

**Project slug:** Run `basename $(pwd)` and lowercase the result (e.g. `localhostly`).

**Project path:** Run `pwd` to get the full absolute path.

**Topic slug:** Infer a 2-4 word topic from the conversation (e.g. `checkout-flow-bug`,
`ga4-tracking`, `payment-gateway-refactor`). Use lowercase-hyphenated form.

### Step 2: Build the handoff document

Use this template exactly — populate every section, omit a section only if it is
genuinely empty (e.g. no blockers):

```markdown
---
session: YYYY-MM-DD HH:MM
topic: Human-readable topic title
project: project-slug
path: /full/local/path/to/project
---

> **Resume prompt** — paste this into a new session to continue without re-explaining:
>
> ```
> Continuing: [topic title]. Goal: [one sentence]. Status: [one sentence on where things stand].
> Next step: [most immediate action]. Full notes: ~/.claude/handoffs/[filename]
> ```

## Summary
[2–3 sentences: what we set out to do, what happened, where it stands now.]

## Work Done
- [Completed item — be specific, include outcomes not just actions]
- …

## In Progress
- [Item currently mid-flight, with its current state]
- …

## Key Decisions
| Decision | Rationale |
|---|---|
| [What was decided] | [Why — the constraint, tradeoff, or reason that drove it] |

## Files Touched
- `path/to/file.ext` — [what changed and why]
- …

## Open Questions
- [ ] [Unresolved question that needs an answer before work can continue]
- …

## Blockers
- [Anything blocking forward progress, with enough detail to unblock]

## Next Steps
1. [Most immediate action — specific enough to start without asking anything]
2. [Second priority]
3. …

## What to Avoid
- [Failed approach, rejected idea, or explicitly out-of-scope direction — with the reason]
- …
```

### Step 3: Write the file

1. Use the user home directory: `~/.claude/handoffs/` (not the project directory).
2. Create `~/.claude/handoffs/` if it doesn't exist.
3. Write the file as `~/.claude/handoffs/PROJECT-TOPIC-YYYY-MM-DD-HHMM.md`, where:
   - `PROJECT` is the project slug from Step 1
   - `TOPIC` is the topic slug from Step 1
   - `YYYY-MM-DD-HHMM` is the current local date and time (use `date '+%Y-%m-%d-%H%M'`)
4. Report the path to the user.

### Step 4: Confirm to the user

After writing the file, tell the user:
- The file path (as a clickable link)
- The resume prompt (so they can copy it immediately without opening the file)

Keep your confirmation short — one sentence on the file path, then the resume prompt block.

### Tone and length calibration

| Session complexity | Summary length | Total doc size |
|---|---|---|
| Short / focused | 1–2 sentences | ~200 tokens |
| Medium | 2–3 sentences | ~400 tokens |
| Long / multi-topic | 3–4 sentences | ~700 tokens |

Don't pad. A tight handoff that fits on one screen is more useful than an exhaustive one
that nobody reads in full.

---

## List Mode

**Goal:** Show the user their saved handoffs and let them pick one to get its resume prompt.

### Step 1: Gather files

Run: `ls -t ~/.claude/handoffs/*.md 2>/dev/null`

If no files exist, tell the user the directory is empty and exit.

### Step 2: Display numbered list

Parse each filename (`PROJECT-TOPIC-YYYY-MM-DD-HHMM.md`) and display like:

```
Saved handoffs:

 1. localhostly — session-handoff-skill  (2026-04-24 15:30)
 2. localhostly — offload-archive-modes  (2026-04-23 11:05)
 3. myapp       — auth-refactor          (2026-04-20 09:42)

Enter a number to load its resume prompt, or press Enter to cancel.
```

Sort newest-first (ls -t already does this). Limit display to 20 most recent.

### Step 3: Ask the user to pick

Use `AskUserQuestion` to prompt: `"Enter a number (1–N) to load its resume prompt, or press Enter to cancel:"`

### Step 4: Show the resume prompt

Read the selected file. Extract and display the resume prompt block (the blockquote immediately
after the frontmatter). Also show the file path as a clickable link.

If the user presses Enter or enters an invalid number, exit cleanly with no error.
