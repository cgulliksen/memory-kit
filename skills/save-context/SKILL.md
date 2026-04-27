---
name: save-context
description: Save the current session context to a named file before clearing the conversation. Use when user runs /save-context, says "save context", "save session", "save state", "save progress", or wants to preserve work before clearing or resetting. Also use when user says "I need to stop" or "let's pause".
---

# Save Context

Save the current session context so it can be resumed later: accurately, without lying about state.

This skill writes a session log with a strict "Resume highlights" block that the `/resume` skill reads first. Permanent knowledge gets migrated to `{{CRITICAL_CONTEXT_FILE}}`. Volatile data (this week's priorities, current state, hot items) stays in `{{WORKING_MEMORY_FILE}}` and is referenced by pointer, not duplicated.

## Iron rule

**All session-state lives ONLY in `{{WORKSPACE_ROOT}}/{{SESSIONS_DIR}}`.** Never in your domain folders ({{DOMAIN_FOLDERS}}) or other working directories. Working files (transcripts, JSON data, renders, code) can stay in their project directories. Session-state and `{{CRITICAL_CONTEXT_FILE}}` do not.

## Instructions

When the user runs /save-context:

### Step 1: Decide structure (flat or folder)

**Single session (flat):** A one-off topic with no prior sessions. Save as `{{SESSIONS_DIR}}/{slug}.md`.

**Multi-session project (folder):** Either an existing `{{SESSIONS_DIR}}/{project}/` folder OR this is session 2+ on a recurring topic (a specific project, client, build, lead, etc). Use:

```
{{SESSIONS_DIR}}/{project}/
├── MASTER.md
├── 01-{slug}.md
├── 02-{slug}.md
└── ...
```

**Conversion rule:** If a flat `{{SESSIONS_DIR}}/{topic}.md` exists and this is the second session on that topic:
1. Create `{{SESSIONS_DIR}}/{topic}/` folder
2. Move the old flat file to `{{SESSIONS_DIR}}/{topic}/01-{old-slug}.md`
3. Create `{{SESSIONS_DIR}}/{topic}/MASTER.md` (seeded from the old file's content)
4. Write this session as `02-{slug}.md`

### Step 2: Determine slug

- Use the session name if one exists. Convert to filename-safe slug (lowercase, hyphens). "Database Migration" → `database-migration`
- For numbered logs, use the next available: `01-`, `02-`, etc.
- If no session name exists, ask the user what to name this session.

### Step 3: Write the session log

Write the new session log file (flat or numbered).

**STRICT STRUCTURE: header + Resume highlights block first, then full log.**

```markdown
# Session NN: {slug}

**Date:** {YYYY-MM-DD}
**Session name:** {original session name}

---

## Resume highlights (read first on /resume)

**Critical state change since last session:**
- {1-3 sentences on the main change. What's new and load-bearing.}

**Trust-but-verify on first tool call:**
- {Concrete checks the next session must run BEFORE trusting state. E.g. "verify the cron fired Tuesday: ls path/to/expected/file"}
- {Pointers to fresher authoritative sources if state may have drifted}

**Open questions with owner:**
- **{Q1}**: {short context}. Owner: {user / agent / shared}
- **{Q2}**: {short context}. Owner: {...}

**Locked decisions this session (do not relitigate; migrated to MASTER.md):**
- {decision 1}: {1-line reason}
- {decision 2}: {...}

**Files the next session should know about (touched this session):**
- `path/to/file`: what it contains
- `path/to/another`: ...

**Pointers to volatile state (don't duplicate; read fresh from source):**
- This week's priorities: `{{WORKING_MEMORY_FILE}}`. Snapshot at save: {brief}
- Active todos: `{{TODOS_FILE}}`. Snapshot: {brief}
- Other volatile sources: {pointers}

---

## Full session log

### What we worked on
{summary of this session's task/conversation}

### Key decisions made
{decisions or directions agreed upon this session}

### Files modified
{created, edited, deleted this session}

### Open questions
{unresolved from this session. Same as Resume highlights but allowed more detail.}
```

**Rules for the Resume highlights block:**
- Critical state change = max 3 sentences
- Trust-but-verify = concrete checks, not vague reminders
- Open questions ALWAYS have an owner field. Without one, the question rots.
- Locked decisions = "don't relitigate" flag. What we agreed not to revisit.
- Volatile state (this week's priorities, todos, current metrics) = POINTERS to authoritative sources, NOT duplicates
- If a section is empty: write "-" or skip it entirely

### Step 4: Update or create MASTER.md (multi-session folders only)

`{{CRITICAL_CONTEXT_FILE}}` (typically `MASTER.md`) = **PERMANENT KNOWLEDGE.** Lean, stable, pruned aggressively. Not session-state.

**MASTER.md structure (permanent only):**
- **Resume protocol:** how a new session should boot up
- **Foundation:** locked stack/tools/integrations
- **Phase plan:** compact table (status column can update, but not detail)
- **File / folder convention:** locked
- **Locked decisions:** do not relitigate
- **Anti-patterns:** what doesn't work, don't try again
- **Permanent learnings:** patterns that work, reuse them
- **Resolved open questions:** historical table, closed
- **Key files + paths:** where things live
- **Session log:** pointer-only links to NN-logs (one line each)

**STRICT pruning rules:**
- "Current state with date" bullets MOVE to NN-log Resume highlights, NOT in MASTER.md
- Hot items / weekly priorities / current metrics → NEVER in MASTER.md (volatile)
- When an Open question gets answered → move it to "Resolved open questions" table with the resolution
- When a Phase finishes → update the status column in Phase plan, don't append history bullets
- If MASTER.md grows past ~150 lines on a stable project → re-compress
- Permanent learnings and anti-patterns: APPEND, but check whether they generalize an existing entry before adding a new one

**Migration on every save-context:**
1. Check the previous NN-log's Open questions → move resolved ones to MASTER.md "Resolved" + carry unresolved into this session's Resume highlights
2. Check whether this session's Locked decisions are evergreen → add to MASTER.md, drop from "do not relitigate" lists in future logs (they're in MASTER.md from now on)
3. Trim "Current state" details out of MASTER.md if you spot them. They belong in the newest NN-log Resume highlights.

### Step 5: Auto-update todos

Read `{{WORKSPACE_ROOT}}/{{TODOS_FILE}}` and update based on what happened:

1. **Mark completed items:** `[x]` + date
2. **Add new items:** sort into Now / This Sprint / Soon (or whatever sections the file uses)
3. **Move items:** if priorities shifted
4. **Show what changed:**

```
Todos updated:
✓ {item marked done}
+ {new item added}
↑ {item moved}
```

### Step 6: Content angle extraction

Suggest 0–2 content angles. Only if genuinely interesting work worth sharing.

Rules:
- Based on REAL work, not generic topics
- Angle only (1–2 sentences), never a full draft
- Think: "What did the user do that would make a peer think 'can I do that too?'"
- Skip if pure internal/admin

```
Content angles from this session:
→ {angle 1}
→ {angle 2}
```

If nothing: "No content angle from this session."

### Step 7: Confirm

Tell the user:
- Where the session was saved (path)
- If MASTER.md was created/updated
- What todos changed
- Any content angles
- Whether to commit (if it's a git repo)

The user can safely clear or reset.
