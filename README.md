# memory-kit

A memory system for Claude Code that doesn't lie about state.

Two skills (`/save-context` and `/resume`) plus a 4-layer memory architecture that survives across sessions, machines, and weeks. Self-installing: Claude Code reads `INSTALL.md`, asks you 5 questions, and sets it up tailored to how you work.

---

## Why this exists

Claude Code is great inside a single session. Across sessions, it forgets. You either:

- Re-explain everything every time you open a fresh window, or
- Trust a half-stale CLAUDE.md that says yesterday's truth, or
- Drown in a folder of session logs nobody re-reads.

This kit fixes that with three small ideas:

1. **Resume highlights block.** Every session ends with a strict 6-section block at the top of the log: critical state change, trust-but-verify checks, open questions with owners, locked decisions, files touched, pointers to volatile state. The next session reads only this block. Fast, accurate, no archive-scrubbing.

2. **Trust-but-verify on resume.** When you `/resume`, the agent doesn't just read the snapshot. It executes the verification checks the previous session left for it. If the world has changed since the snapshot, the agent flags the drift instead of lying about it.

3. **Layered memory with strict roles.** Identity (CLAUDE.md), Critical Context (MASTER.md per project), Working Memory (priority.md), Episodic (sessions/NN-*.md). Each layer has one job. Volatile data never leaks into stable layers.

Read [PHILOSOPHY.md](./PHILOSOPHY.md) for the longer version.

---

## Install

In any workspace where you want this:

```
1. Clone this repo (or download as zip) into the workspace.
2. Open Claude Code.
3. Say: "Read INSTALL.md and install memory-kit for me."
```

Claude reads `INSTALL.md`, detects whether you already have a `CLAUDE.md` or skills directory, asks you 5-6 short questions about how you work, and installs:

- `.claude/skills/save-context/SKILL.md`
- `.claude/skills/resume/SKILL.md`
- `CLAUDE.md` (seeded, extended, or left alone, your call)
- `MASTER.md`, `priority.md`, `todos.md` (seeded if not present)
- `sessions/` directory

Nothing gets overwritten without your explicit confirmation. If you already have skills installed, the kit just adds to `.claude/skills/` without touching the rest.

---

## Use

### After a working session

Run `/save-context`. The skill:

1. Decides if this is a single-session topic or a multi-session project (creates the folder if you're at session 2)
2. Writes a session log with a Resume highlights block at the top
3. Migrates resolved questions to MASTER.md, prunes stale current-state from MASTER.md
4. Updates `todos.md`
5. (Optional) suggests 0-2 content angles if you publish from your work

### Starting a new session

Run `/resume`. The skill:

1. Lists your sessions (flat files + project folders)
2. Reads MASTER.md first, then only the Resume highlights block of the newest log
3. Executes the trust-but-verify checks the previous session left
4. Reports current state (verified) + open questions + drift, if any
5. Asks what you want to do next

---

## What's NOT in this kit

This kit is the memory layer. Nothing more. It's not:

- A lead pipeline / CRM
- A content production workflow
- An ad campaign manager
- A team coordination tool

Those are separate concerns. If you want them, build them on top of this layer (or wait for companion kits like `lead-pipeline-kit` and `content-kit` when they're ready).

---

## File layout (this repo)

```
memory-kit/
├── README.md                    # this file
├── INSTALL.md                   # the agent's install prompt
├── PHILOSOPHY.md                # the why
├── LICENSE                      # MIT
├── skills/
│   ├── save-context/SKILL.md    # the save skill (template form)
│   └── resume/SKILL.md          # the resume skill (template form)
├── templates/
│   ├── CLAUDE.md.template
│   ├── MASTER.md.template
│   └── priority.md.template
└── examples/
    └── README.md                # link to a live example workspace
```

---

## Origin

The Resume highlights block + trust-but-verify pattern emerged from real cross-session pain on the [Iverksetter](https://iverksetter.no) hub. If your team finds a sharper version, send a PR.

---

## License

MIT. Use it, fork it, ship it.
