---
name: resume
description: Restore session context from a previously saved session. Use when user runs /resume, says "resume", "pick up where we left off", "continue last session", "what were we working on", or starts a new conversation and wants to restore previous state.
---

# Resume Context

Restore session context from a previously saved session. Read MASTER.md first, then the latest session's Resume highlights, then **verify state against reality before reporting it to the user.**

The `/save-context` skill writes a snapshot. By the time you read it, the world may have moved on (cron jobs ran, files changed, external state shifted). This skill enforces trust-but-verify so you don't lie about state.

## Instructions

When the user runs /resume:

### Step 1: List all sessions

List `{{WORKSPACE_ROOT}}/{{SESSIONS_DIR}}/` recursively. Group the output:

- **Flat session files:** top-level `.md` files, sorted by mtime desc
- **Multi-session projects:** subfolders with MASTER.md + a count of logs, sorted by newest log/MASTER mtime

If no sessions exist, tell the user and suggest running /save-context first.

Recommended command:
```bash
find {{WORKSPACE_ROOT}}/{{SESSIONS_DIR}} -maxdepth 2 -name "*.md" -type f -exec ls -lt {} +
```

### Step 2: If the user named a session

If the user runs `/resume {name}` (e.g. `/resume billing-rewrite`, `/resume client-onboarding`):

- If a subfolder matches (`{{SESSIONS_DIR}}/{name}/`), load `{{SESSIONS_DIR}}/{name}/MASTER.md`. Always prefer MASTER over individual logs.
- If a flat file matches (`{{SESSIONS_DIR}}/{name}.md`), load that.
- Partial/fuzzy match is OK. Load the closest.

### Step 3: If no name was given

Show the grouped list (flat files + project folders) with dates. Ask which to resume.

### Step 4: Once selected

**For a project folder (the standard case):**
1. Read `MASTER.md` first. Permanent knowledge (foundation, locked decisions, anti-patterns, file paths).
2. Read the **entire newest** `NN-*.md` log. Highlights block at the top gives you structured signal (state changes, open questions, trust-but-verify); the full log below gives you the prose context for *why* — bash patterns, debugging detail, decisions that didn't compress cleanly into bullets. The latest log is cheap (typically 100–300 lines) and load-bearing.
3. Do NOT read older `NN-*.md` logs by default. Older logs are archive — fetched on demand only if MASTER + the latest log are missing context, or if the user asks for specific historical detail.

**For a flat file:** read the file directly.

### Step 5: Trust-but-verify BEFORE reporting state

The Resume highlights block has a "Trust-but-verify" section. **Before summarizing state for the user:**

- Execute every concrete check it lists (file existence, log tail, fresh state from external services, etc.)
- If state has drifted since save-context: flag it explicitly to the user ("MASTER said X but actual state is Y")
- Check the volatile-pointer sources Resume highlights pointed to (`{{WORKING_MEMORY_FILE}}`, `{{TODOS_FILE}}`, etc.) for fresher data than the snapshot

Don't skip this step. The whole point of this skill is that the agent verifies before reporting.

### Step 6: Summarize for the user

After trust-but-verify, present:

- **Current state**: what the system actually is (after verification, not just what the snapshot said)
- **Open questions**: from Resume highlights, sorted by owner
- **Locked decisions**: short list from MASTER.md relevant to today's work
- **Trust-but-verify result**: any drift from the snapshot? What changed?

### Step 7: Offer next move

Ask if they want to continue where they left off, read a specific older log, or pivot to something else.
