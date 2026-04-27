# PHILOSOPHY.md (save-context-kit)

> Why this kit exists, and why it's shaped the way it is.

---

## The problem

A coding agent that can read your whole repo is impressive. Until you close the window. Open a new one, and it's a stranger again. You re-explain the architecture, the in-flight decisions, the things you tried last week that didn't work.

The standard fix is a `CLAUDE.md` file. It works for a few weeks. Then it bloats. Then half of it is stale. Then the agent boots up confidently citing decisions that were reversed in a session that nobody logged.

The deeper problem isn't that the agent forgets. It's that the file system pretending to remember has no discipline about what's stable, what's fresh, and what's already moved on.

---

## The four layers

This kit splits memory into four layers with non-overlapping jobs.

### 1. Identity (`CLAUDE.md`)

Who you are. What you work on. How you like to work. Voice, tone, tools, team.

**Volatility:** rare edits. Maybe once a month.

**Failure mode:** putting volatile data here. Lead status. Current week. Active bugs. None of that belongs in identity. If you put it here, the file rots.

### 2. Critical context (`MASTER.md`, per project)

Locked decisions and the reasons behind them. Architecture. Anti-patterns. Permanent learnings. The handful of facts a new session must know to not redo last month's debate.

**Volatility:** edits per planning session. Maybe weekly on active projects.

**Failure mode:** appending session logs into MASTER. It bloats, current-state mixes with permanent rules, and pruning gets harder every week. Strict pruning is the only thing that keeps it useful.

### 3. Working memory (`priority.md`)

This week's bet. Hot items. Next actions. The 3–5 things that actually matter right now.

**Volatility:** weekly. After major events, more often.

**Failure mode:** trying to be a kanban board. It's not. It's a 1-page summary of what's hot. If you can't fit it on one page, you don't have priorities. You have a backlog.

### 4. Episodic (`sessions/{project}/NN-*.md`)

Append-only changelog of what happened. One file per work session.

**Volatility:** never edited after writing. Pure history.

**Failure mode:** treating it as a knowledge base. It's not. It's a changelog. Permanent learnings move to MASTER. Volatile state stays out of the log entirely (it goes to working memory or pointer-only references).

---

## The Resume highlights block

Every session log starts with a strict block:

```markdown
## Resume highlights (read first on /resume)

**Critical state change since last session:**
- ...

**Trust-but-verify on first tool call:**
- ...

**Open questions with owner:**
- ...

**Locked decisions this session (do not relitigate):**
- ...

**Files the next session should know about:**
- ...

**Pointers to volatile state (don't duplicate; read fresh):**
- ...
```

This is the only part of the log a new session reads by default. The rest of the log is archive: searched, not loaded.

The block solves three problems:

1. **Speed.** Loading a full log every resume burns context. Loading 80 lines of structured highlights is fast and rarely truncated.

2. **Discipline.** Six fixed sections force the writer to separate "what's permanent" from "what's volatile" from "what needs verification." If any section is empty, that's a signal. Not a failure.

3. **Owner accountability.** Every open question carries an owner field. Without an owner, the question rots silently. With one, it shows up in the next resume's todo.

---

## Trust-but-verify

A snapshot is a claim about the world at the moment you wrote it. By the time you read it, the world has often moved on. Cron jobs ran. Files changed. External services updated state. The build is now green.

The standard agent failure mode is to read the snapshot and report it as truth. "Your build is failing." Except the build finished an hour ago.

The Resume highlights block has a `Trust-but-verify` section. It lists concrete checks the next session must run before reporting state:

- "Verify the cron fired Tuesday: `ls leads/inbox/2026-04-28-batch.md`"
- "Check Notion CRM via MCP for actual state, not what MASTER says"
- "Read `priority.md` for fresher hot items than this snapshot"

The `/resume` skill executes these checks before summarizing. If state has drifted, it flags the drift explicitly. The agent reports what's true, not what was true.

This isn't sophistication for its own sake. It's the difference between an agent that wastes the first 10 minutes of every session being confidently wrong, and one that boots into reality.

---

## What this kit is NOT

It is not a CRM. It is not a content workflow. It is not a project management tool. It does not generate plans, write outreach, or score leads.

It is a memory layer. A foundation. The thing other workflows can sit on top of.

The temptation when building something useful is to keep adding. We held the line: this kit ships only what every Claude Code user needs, leaves room for what they want, and stays out of opinions about how you should run your business.

If you want lead pipelines, content extraction, or domain-specific scaffolding: fork this and add them, or wait for companion kits that build on this base.

---

## Where this came from

Two practitioners' work shaped the framing.

**Mark Kashef** (ClaudeClaw) articulated the 3-layer memory pattern: identity, critical context, working memory. His implementation goes further (SQLite hive-mind, Gemini extraction, vector embeddings) and his target is the content-creator workflow. We borrowed the layer framing, kept the layers thin, and added an episodic layer specifically for the Claude Code use case.

**Andrej Karpathy's** behavioral principles for skill design (think before coding, simplicity, surgical changes, goal-driven) informed how the SKILL.md files are written. Skills here describe a workflow, not a feature list.

The Resume highlights block + trust-but-verify pattern is original to this kit, hardened across two months of cross-session work on a real agency hub. If you find a sharper version of any of this, the repo accepts PRs.

---

## A note on the agent reading this

If you (the agent) are reading PHILOSOPHY.md as part of installing or using this kit: the load-bearing concepts are layer roles, the Resume highlights block, and trust-but-verify. Everything else is justification. When you write session logs, enforce the block structure. When you resume, do not skip the verify step. The kit's value depends on those two disciplines.
