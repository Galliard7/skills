---
name: chronicle
description: Auto-capture your project's story as you build — decisions, pivots, dead ends, milestones. Export as onboarding docs or content hooks. Usage: /chronicle [init|<note>|export]
---

# /chronicle — Project Chronicle

Capture the story of your project as it progresses. Not just what changed (that's git) — but *why*, what you tried, what failed, what surprised you, and what you'd tell someone joining tomorrow.

## Commands

Parse `$ARGUMENTS` to determine the subcommand:

- `/chronicle init` — activate chronicle for this project
- `/chronicle <free text>` — append a manual entry
- `/chronicle export --onboard` — generate an onboarding briefing
- `/chronicle export --content` — extract content-worthy hooks for posts/threads
- `/chronicle` (no args) — synthesize & record current session
- `/chronicle show` — display the last 3 entries

---

## /chronicle init

Create `CHRONICLE.md` in the project root:

```markdown
# Chronicle

Project story — decisions, pivots, milestones, and dead ends.
Auto-captured by the chronicle skill. Reverse-chronological.

<!-- chronicle:config
  Auto-capture: When this file exists, CC should append a chronicle entry
  before ending any session where meaningful work was done. Skip trivial
  sessions (just reading, quick questions, no project changes).
  Manual entries: /chronicle <note>
  Exports: /chronicle export --onboard | --content
-->

---
```

If a `.claude/CLAUDE.md` exists in the project, check whether it already mentions `CHRONICLE.md`. If not, append this single line:

```markdown
This project maintains a `CHRONICLE.md` — read it for project history and context.
```

Do not add chronicle instructions, format specs, or anything else to CLAUDE.md — just the pointer.

Confirm to the user: "Chronicle activated. CC will auto-capture entries at session end when it detects this file. Use `/chronicle <note>` for manual entries anytime."

---

## /chronicle <free text> (manual entry)

Append an entry to `CHRONICLE.md` immediately. The user's free text is the note.

1. Read the existing `CHRONICLE.md` (if missing, tell user to run `/chronicle init` first).
2. Determine the entry type from the content:
   - Decision → `#decision`
   - Pivot or failed approach → `#pivot`
   - Milestone or progress → `#milestone`
   - Blocker or surprise → `#blocker`
   - General note → `#note`
3. Append the entry right after the header `---` (before existing entries, so newest is first).

### Entry format (hybrid: narrative + structured footer)

```markdown
## YYYY-MM-DD — {Short Title}

`#type-tag`

{1-3 paragraph narrative: what happened, why, what was tried, what was learned. Write in first person. Be specific — names, numbers, concrete details. Include the dead ends, not just the wins.}

---
**Decisions:** {key choices made, or "—"}
**Progress:** {what moved forward, or "—"}
**Blocked:** {open blockers, or "—"}
**Content-worthy?:** {Yes — 'potential post angle' | No}
```

---

## /chronicle (no args) — Session Capture

Synthesize and record the current session's chronicle-worthy material. This is the primary way entries get created.

1. Read `CHRONICLE.md` (if missing, tell user to run `/chronicle init` first).
2. Reflect on everything that happened in this session:
   - **Decisions made** — architecture choices, tool picks, tradeoff resolutions
   - **Changes implemented** — what was built, modified, or shipped
   - **Design work** — brainstorms, specs drafted, patterns established
   - **Strategy pivots** — approaches tried and abandoned, direction changes
   - **Insights & surprises** — things learned, unexpected behaviors, gotchas discovered
3. Synthesize into a single narrative entry covering the session (not one entry per event).
4. If nothing chronicle-worthy happened (trivial session, just reading or quick questions), say so and skip.
5. Append the entry to `CHRONICLE.md` in the hybrid format (newest first, after the header `---`).
6. Confirm what was captured with a one-line summary.

To view recent entries instead, use `/chronicle show`.

---

## /chronicle show

Read `CHRONICLE.md` and display the 3 most recent entries.

---

## /chronicle export --onboard

Generate an onboarding briefing that gives a human or agent everything they need to start contributing. Combines the chronicle narrative with a scan of the actual project.

### Step 1 — Read the chronicle

Read `CHRONICLE.md` in full.

### Step 2 — Scan the project

Explore the project structure to build a map of key assets. Look for:

- **Entry points** — main scripts, index files, app entry points, CLI commands
- **Configuration** — config files, env templates, CI/CD pipelines, Dockerfiles, Makefiles
- **Documentation** — README, ARCHITECTURE.md, CONTEXT.md, ADRs, specs, design docs
- **Scripts** — build scripts, deploy scripts, migration scripts, test runners, utility scripts
- **Data / schemas** — database schemas, API schemas, fixture files, seed data
- **Tests** — test directories, test config, notable test files
- **Key source directories** — where the core logic lives, how the codebase is organized

Use glob patterns and directory listing to discover these. Don't read every file — just identify what exists and its purpose from names, paths, and brief inspection where needed.

### Step 3 — Synthesize the briefing

Write `CHRONICLE-ONBOARD.md` in the project root with these sections:

```markdown
# {Project Name} — Onboarding

## What this project is
{1-2 paragraphs derived from the arc of chronicle entries}

## Project map
{Table or organized list of key files and directories with one-line descriptions}
| Path | What it is |
|---|---|
| `src/main.py` | Application entry point |
| `scripts/deploy.sh` | Production deploy script |
| ... | ... |

## Key decisions
{Extract from #decision entries — what was chosen and why}

## Current state
{What's working, what's in progress, what's next}

## Known gotchas
{From #blocker and #pivot entries — things that will bite you if you don't know about them}

## What was tried and didn't work
{From #pivot entries — failed approaches and why, so the next person doesn't repeat them}

## Getting started
{Practical steps: how to set up, build, run, and test — derived from config files, scripts, and README}
```

### Step 4 — Deliver

Tell the user the file path. This file is regenerated fresh each time (not append-only). If the project has changed significantly since the last export, the new version reflects current reality.

---

## /chronicle export --content

Scan the chronicle for content-worthy material:

1. Read `CHRONICLE.md` in full.
2. Find all entries where **Content-worthy?** is "Yes" or where the narrative contains interesting lessons, surprising findings, or contrarian decisions.
3. Group by theme (e.g., "debugging war stories", "architecture decisions", "tool discoveries").
4. For each theme, generate 2-3 sentence X post hooks — punchy, opinionated, with a concrete takeaway. These are starting points, not finished posts.
5. Write to `CHRONICLE-CONTENT.md` in the project root.
6. Tell the user the file path and how many hooks were generated.

---

## Auto-capture behavior (session end)

When `CHRONICLE.md` exists in the project root, CC should append an entry before ending a session where meaningful work was done. The process:

1. Reflect on the session: what was the user trying to accomplish? What happened?
2. Identify chronicle-worthy events:
   - **Decisions:** any choice between alternatives where you picked one for specific reasons
   - **Pivots:** any approach that was tried and abandoned, and why
   - **Milestones:** any feature, fix, or capability that now works end-to-end
   - **Blockers:** anything unexpected that changed the plan
3. Write one entry covering the session (not one per event — weave them into a narrative).
4. If nothing chronicle-worthy happened (trivial session), skip the entry entirely.
5. Append to `CHRONICLE.md` in the hybrid format.

---

## Guidelines

- **Narrative over bullet points.** The chronicle tells a story. Future-you (or a new team member) should be able to read it and understand not just what was built, but the journey.
- **Include the dead ends.** "We tried X and it didn't work because Y" is often more valuable than "we did Z."
- **Be specific.** Names, version numbers, error messages, benchmarks. Vague entries are useless entries.
- **Don't over-log.** One entry per meaningful session. Quick Q&A or file reads don't need entries.
- **Content-worthy is a low bar.** If you'd tell a friend about it over coffee, mark it content-worthy.
