# INSTALL.md (memory-kit)

> **For Claude Code agents installing this kit on behalf of a user.**
>
> If you (the agent) are reading this, it means the user just dropped this repo into their workspace and asked you to install it. Follow this prompt step by step. Don't skip questions, don't make assumptions, don't dump files until the user has answered the onboarding flow.
>
> If you (the human) are reading this, you don't need to. Just open Claude Code in your workspace and say: **"Read INSTALL.md and install memory-kit for me."**

---

## Mission

You are setting up a memory system for the user's Claude Code workspace. The kit gives them two skills (`/save-context` and `/resume`) and a 4-layer memory architecture (Identity, Critical Context, Working Memory, Episodic). Your job: detect their existing setup, ask 5-6 short customization questions, then install the skills and seed the memory layer files. Don't clobber anything they already have.

---

## Step 1: Detect the existing setup

Before asking anything, run these checks silently and remember the results:

1. **Workspace root.** Use the user's current working directory unless they pass a different path.
2. **Existing identity file.** Look for `CLAUDE.md` in the workspace root.
3. **Existing skills directory.** Look for `.claude/skills/` in the workspace root.
4. **Existing memory files.** Look for any of: `MASTER.md`, `priority.md`, `sessions/`, `todos.md` in the workspace root.
5. **Git repo?** Check if the workspace is a git repo (`git rev-parse --is-inside-work-tree`).

Hold these as `existing.identity`, `existing.skills`, `existing.memory_files`, `existing.is_git`. You'll branch on them later.

---

## Step 2: Greet and confirm scope

Show the user a short message in their language (default English; use Norwegian if their existing files are Norwegian). Tell them:

- What's about to happen: 5-6 questions, then install.
- That nothing will be overwritten without their confirmation.
- That this takes about 2 minutes.

Then move to Step 3.

---

## Step 3: Onboarding question tree

Ask these one at a time. Use what you detected in Step 1 to set smart defaults. **Do not ask all questions in one wall of text.** Wait for each answer before moving on.

### Q1: Workspace root

> "Where is your workspace root? (default: `{cwd}`)"

If they accept the default, use cwd. If they specify a different path, verify it exists.

### Q2: Identity file

Branch on `existing.identity`:

- **If no `CLAUDE.md` exists:** "I don't see a CLAUDE.md yet. Should I seed one for you? It's the top layer of the memory system: who you are, what you do, how you like to work. (yes / no, default yes)"
- **If `CLAUDE.md` exists:** "I found an existing CLAUDE.md. Should I (a) leave it alone, (b) extend it with a memory-system section pointing at the new files, or (c) replace it? (default: b, extend)"

### Q3: What kind of work

> "What kind of work do you do here? Pick one or describe your own:
>  [a] Agency with clients
>  [b] Solo operator (marketing, dev, design, mixed)
>  [c] Product or startup team
>  [d] Research, learning, or knowledge work
>  [e] Other (describe in one line)"

This shapes the domain folders in Q4 and the tone of the seeded `CLAUDE.md`.

### Q4: Domain folders

Suggest defaults based on Q3:

- Agency → `clients/`, `leads/`, `projects/`, `content/`
- Solo operator → `projects/`, `content/`, `ideas/`, `research/`
- Product team → `product/`, `research/`, `experiments/`, `team/`
- Research → `topics/`, `notes/`, `papers/`, `experiments/`

> "Here are some folders that match how you work: `{suggested}`. Press enter to accept, or type your own comma-separated list."

Domain folders are NOT created as part of this install. The kit only creates `sessions/` (the episodic layer). Domain folders are the user's existing or future workspace. The kit just notes them in `CLAUDE.md` so the agent knows where to look.

### Q5: Memory layer file names

Most users accept the defaults. Only ask if you suspect they want different names (e.g. they already have a `ROADMAP.md` they treat as critical context).

> "I'll use these filenames for the memory layers. Okay?
>  - Identity: `CLAUDE.md`
>  - Critical context: `MASTER.md`
>  - Working memory: `priority.md`
>  - Episodic logs: `sessions/`
>
> Press enter to accept, or specify alternatives."

### Q6: Optional content extraction

> "Optional: when you save a session, I can suggest 0-2 content angles from the work. Useful if you publish on LinkedIn, Twitter, or a blog. Enable? (yes / no, default no)"

If yes, the `/save-context` skill will include the content-angle step. If no, skip that step entirely (and remove the section from the installed skill file).

---

## Step 4: Show install plan

Before writing anything, show the user exactly what you're about to do:

```
Install plan:

✓ Skills:
  - .claude/skills/save-context/SKILL.md   (NEW)
  - .claude/skills/resume/SKILL.md          (NEW)

✓ Memory layers:
  - CLAUDE.md                               ({seed | extend | leave alone})
  - MASTER.md                               (NEW: empty stub)
  - priority.md                             (NEW: empty stub)
  - sessions/                               (NEW directory)
  - todos.md                                (NEW: empty stub)

✓ Configuration baked in:
  - Workspace root: {root}
  - Workflow type: {a|b|c|d|e}
  - Domain folders referenced: {list}
  - Content extraction: {on | off}

Proceed? (yes / no)
```

Wait for explicit "yes" before writing files.

---

## Step 5: Write the skill files

Read the source SKILL.md files from this kit at:
- `<KIT_ROOT>/skills/save-context/SKILL.md`
- `<KIT_ROOT>/skills/resume/SKILL.md`

For each, substitute the following template variables before writing the file to the user's workspace:

| Variable | Value |
|---|---|
| `{{WORKSPACE_ROOT}}` | The path from Q1 |
| `{{WORKSPACE_NAME}}` | The basename of `{{WORKSPACE_ROOT}}` (e.g., `acme-hub` if root is `/Users/x/acme-hub`) |
| `{{IDENTITY_FILE}}` | `CLAUDE.md` (or whatever Q5 set) |
| `{{CRITICAL_CONTEXT_FILE}}` | `MASTER.md` (or Q5) |
| `{{WORKING_MEMORY_FILE}}` | `priority.md` (or Q5) |
| `{{TODOS_FILE}}` | `todos.md` (or Q5) |
| `{{SESSIONS_DIR}}` | `sessions/` (or Q5) |
| `{{DOMAIN_FOLDERS}}` | The list from Q4, comma-separated inline (e.g., `clients/, leads/, projects/`) |
| `{{DOMAIN_FOLDERS_LIST}}` | The list from Q4 as a markdown bullet list (one folder per line, prefixed with `- `) |

**Variables left literal (do not substitute at install time):** `{{PROJECT_NAME}}`, `{{REPLACE_THIS}}`, `{{REPLACE_THIS_OR_DELETE}}`. These are filled in by the user (or the `/save-context` skill) when they actually create a project or edit their CLAUDE.md.

If `content_extraction = false`, **delete the entire "Step 6: Content angle extraction" section** from the installed `save-context/SKILL.md` before writing it. Don't leave a stub.

Write the skills to:
- `{{WORKSPACE_ROOT}}/.claude/skills/save-context/SKILL.md`
- `{{WORKSPACE_ROOT}}/.claude/skills/resume/SKILL.md`

If `.claude/skills/` already exists, just add the two new skill folders. Don't touch anything else.

---

## Step 6: Seed the memory layer files

For each layer, branch on whether it already exists:

### CLAUDE.md (Identity)

- **If "seed" (Q2 = yes/seed):** read `<KIT_ROOT>/templates/CLAUDE.md.template`, substitute variables, write to `{{WORKSPACE_ROOT}}/CLAUDE.md`.
- **If "extend":** append a new section to the existing `CLAUDE.md` headed `## Memory system`. The section explains the 4 layers and points at MASTER.md / priority.md / sessions/. Read the template's "Memory system" section as the source of the appended content.
- **If "leave alone":** do nothing.

### MASTER.md (Critical Context)

- **If file exists:** ask "I found an existing MASTER.md. Leave alone, or seed a fresh stub?" Default: leave alone.
- **If not:** read `<KIT_ROOT>/templates/MASTER.md.template`, substitute, write.

### priority.md (Working Memory)

Same pattern as MASTER.md. If exists → ask. If not → seed from template.

### sessions/ (Episodic)

Create the directory if it doesn't exist. Add a `sessions/.gitkeep` if the workspace is a git repo. Don't seed any session logs.

### todos.md (Working Memory companion)

Same pattern. Ask if exists, seed if not.

---

## Step 7. Confirm install

Show the user the result:

```
Installed memory-kit at {root}

Files written:
  ✓ .claude/skills/save-context/SKILL.md
  ✓ .claude/skills/resume/SKILL.md
  ✓ CLAUDE.md ({action})
  ✓ MASTER.md ({action})
  ✓ priority.md ({action})
  ✓ sessions/ ({action})
  ✓ todos.md ({action})

Try it now:
  1. Have a conversation about something real.
  2. Run /save-context when you want to pause or wrap up.
  3. Open a fresh Claude Code session and run /resume.

The /resume skill will read your memory layers AND verify them against
your actual workspace state, so it tells you what's true, not what
the snapshot said yesterday.

Read PHILOSOPHY.md if you want the why behind the 4-layer architecture.
```

---

## Step 8. Optional: offer git commit

If the workspace is a git repo and `existing.is_git == true`, ask:

> "Want me to commit the new files? (yes / no)"

If yes, stage the new files and commit with message:

```
Install memory-kit

- /save-context and /resume skills
- 4-layer memory system (Identity, Critical Context, Working Memory, Episodic)
- See PHILOSOPHY.md for architecture
```

Don't push. Don't force user to commit. They can clean up later.

---

## Failure modes, handle these gracefully

- **User answers "no" to Step 4 install plan:** abort cleanly, write nothing, tell them they can re-run install anytime.
- **A target file exists and user said "leave alone":** skip it silently in Step 6, note it in Step 7 confirm.
- **User's workspace already has `.claude/skills/save-context/`:** show a diff between the existing skill and what you'd install. Ask: "Replace, merge, or skip?" Default: skip.
- **User cancels mid-flow:** abort. Nothing partial. Either fully installed or not at all.

---

## After install

The user can immediately:
- Run `/save-context` at the end of any conversation
- Run `/resume` at the start of a new one
- Read `PHILOSOPHY.md` to understand the system
- Edit `CLAUDE.md` to reflect who they are

If they want to extend the system later (lead pipelines, content workflows, client briefs), those are separate companion kits, not part of this one. Don't bake them in.
