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
- `/chronicle export --timeline` — render an interactive HTML timeline of the project's journey
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
  Exports: /chronicle export --onboard | --content | --timeline
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

## /chronicle export --timeline

Render a single self-contained, interactive `CHRONICLE-TIMELINE.html` — a vertical,
**time-scaled** timeline of the project's journey for onboarding a human or an agent.
Nodes are spaced by real elapsed time (quiet stretches get a "~N weeks quiet" marker);
click a node and a **side panel** shows a glanceable summary, with a **Read more** reveal
for the full narrative. Filter by tag, search by text, flip oldest/newest. No build step,
no network, opens offline by double-click. Complements `--onboard` (the prose brief); the
timeline is the navigable map.

### Step 1 — Read the chronicle

Read `CHRONICLE.md` in full (if missing, tell the user to run `/chronicle init` first).

### Step 2 — Parse entries into a JSON array

Each entry starts at a `## YYYY-MM-DD — {Title}` header. Build one object per entry:

```
{ "date": "YYYY-MM-DD", "title": "...", "tags": ["milestone","decision"],
  "narrative": "full narrative text (may be multiple paragraphs)",
  "decisions": "...", "progress": "...", "blocked": "...", "contentWorthy": "..." }
```

Parsing rules (**be tolerant** — older/hand-written entries vary):

- **date / title** come from the header line. Accept either `—` or `-` as the separator.
  Skip the file's `# Chronicle` title and the `<!-- chronicle:config -->` preamble — only
  `## <date> — …` blocks are entries.
- **tags**: collect every `` `#tag` `` token on the tag line(s) right under the header. An
  entry may carry **several** (e.g. `` `#milestone` `#decision` `#pivot` ``) — keep them
  **all**, in order, lowercased, restricted to `decision|pivot|milestone|blocker|note`. If
  an entry has **no** tag, default `tags` to `["note"]`.
- **narrative**: everything between the tag line and the entry's footer `---`, with the
  tag-only line(s) removed. Preserve paragraph breaks. If there's no footer `---`, the
  whole body after the tags is the narrative (and the footer fields below are empty).
- **footer fields**: from the `**Decisions:** … / **Progress:** … / **Blocked:** … /
  **Content-worthy?:** …` block. Missing field or a value of `—` → empty string `""`.

Order in the array doesn't matter (the page sorts by date).

### Step 3 — Choose a theme (make it true to the project)

The template ships three vetted themes, switched by the `data-theme` attribute on the
`<html>` tag:

- **`terminal`** — *the default.* Dark slate + amber, monospace, terminal / git-log feel.
  Use it whenever you're unsure, and for CLIs, libraries, dev tools, and infra projects.
- **`blueprint`** — neon cyan/magenta on deep navy, glass panels, blueprint-grid backdrop.
  Use for web apps / SPAs / anything with a slick product UI.
- **`almanac`** — warm serif, numbered ledger, gold rules. Use for writing, docs, research,
  content, or design-led projects.

Pick the best fit from project signals (project type inferred from files/README, whether
there's a website or app, the overall vibe). **Default to `terminal` if unsure.** Apply it
by replacing `data-theme="terminal"` in the `<html>` tag with your choice.

**Optional accent tint** — only if the project has a *clear, confidently-identified* brand
color (from a logo, its live site, or CSS custom properties like `--primary` / `--accent`
in its stylesheets): tint the timeline to match by replacing the `/*BRAND_OVERRIDE*/`
placeholder with `html[data-theme]{--accent:#RRGGBB;--accent2:#RRGGBB}`. Leave it untouched
otherwise. **Never** change the semantic tag colors (decision/pivot/milestone/blocker/note)
— those stay constant so the legend always reads the same.

### Step 4 — Inject and write

Replace `/*ENTRIES_JSON*/ []` with `/*ENTRIES_JSON*/ <your JSON array>`, set `data-theme`
(Step 3), optionally fill `/*BRAND_OVERRIDE*/` (Step 3), and change nothing else. Write the
result to `CHRONICLE-TIMELINE.html` in the project root, and tell the user the path plus a
one-line note on which theme you chose and why.

All behaviour — time-scaling, color-coding, filtering, search, the side-panel summary,
read-more, and the oldest/newest toggle — is handled client-side by the template's JS, so
output is deterministic regardless of model. The node spine-dot color is the entry's
**first** tag; every tag shows as a chip; the tag filter shows an entry if **any** of its
tags is active. Handles 1 entry to 50+ without layout break; an empty chronicle shows a
friendly empty state.

### Timeline template (set `data-theme`; fill `/*ENTRIES_JSON*/` and optional `/*BRAND_OVERRIDE*/`)

```html
<!DOCTYPE html><html lang="en" data-theme="terminal"><head><meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>Chronicle Timeline</title><style>
/* ============ BASE (theme-agnostic structure + neutral skin) ============ */
:root{
 --bg:#0d1117;--panel:#161b22;--panel2:#11151c;--border:#30363d;--ink:#e6edf3;--muted:#8b949e;
 --accent:#e3a008;--accent2:#e3a008;
 --decision:#58a6ff;--pivot:#f85149;--milestone:#3fb950;--blocker:#f0883e;--note:#8b949e;
 --fdisp:'SF Mono',ui-monospace,Menlo,Consolas,monospace;
 --fbody:-apple-system,BlinkMacSystemFont,'Segoe UI',Roboto,Helvetica,Arial,sans-serif;
 --fmono:'SF Mono',ui-monospace,Menlo,Consolas,monospace;
 --fui:'SF Mono',ui-monospace,Menlo,Consolas,monospace;
 --radius:8px;}
*{box-sizing:border-box;margin:0;padding:0}
body{background:var(--bg);color:var(--ink);font-family:var(--fbody)}
.bgfx{position:fixed;inset:0;pointer-events:none;z-index:0}
.wrap{max-width:1120px;margin:0 auto;padding:28px 18px 64px;position:relative;z-index:1}
.topbar{margin-bottom:18px}
h1{font:700 22px var(--fdisp);color:var(--ink)}
h1 .accent{color:var(--accent)}
.meta{color:var(--muted);font:12px var(--fmono);margin-top:7px}
.bar{display:flex;flex-wrap:wrap;gap:8px;align-items:center;margin-bottom:22px}
.chip{font:600 12px/1 var(--fui);padding:7px 11px;border-radius:6px;border:1px solid var(--border);color:var(--muted);background:var(--panel2);cursor:pointer;user-select:none;transition:.13s}
.chip.on{color:var(--ink)}
.chip[data-tag=decision].on{border-color:var(--decision);color:var(--decision)}
.chip[data-tag=pivot].on{border-color:var(--pivot);color:var(--pivot)}
.chip[data-tag=milestone].on{border-color:var(--milestone);color:var(--milestone)}
.chip[data-tag=blocker].on{border-color:var(--blocker);color:var(--blocker)}
.chip[data-tag=note].on{border-color:var(--note);color:var(--ink)}
.chip.ctrl{margin-left:auto}
#q{flex:1;min-width:160px;max-width:320px;background:var(--panel2);border:1px solid var(--border);border-radius:6px;color:var(--ink);padding:8px 12px;font:13px var(--fmono)}
#q:focus{outline:none;border-color:var(--accent)}
.layout{display:flex;gap:26px;align-items:flex-start}
.tlcol{flex:1;min-width:0}
.tl{position:relative;margin-left:10px;border-left:2px solid var(--border);padding-left:26px}
.gap{position:relative;display:flex;align-items:center;color:var(--muted);font:600 10px var(--fmono)}
.gap span{background:var(--bg);padding:2px 8px;border:1px dashed var(--border);border-radius:4px;margin-left:-2px}
.node{position:relative}
.node::before{content:"";position:absolute;left:-33px;top:15px;width:11px;height:11px;border-radius:50%;background:var(--note);box-shadow:0 0 0 4px var(--bg);transition:.13s}
.node[data-tag=decision]::before{background:var(--decision)}.node[data-tag=pivot]::before{background:var(--pivot)}
.node[data-tag=milestone]::before{background:var(--milestone)}.node[data-tag=blocker]::before{background:var(--blocker)}
.node.sel::before{transform:scale(1.25);box-shadow:0 0 0 4px var(--bg),0 0 12px 2px var(--accent)}
.card{background:var(--panel2);border:1px solid var(--border);border-radius:var(--radius);cursor:pointer;transition:.13s;margin-bottom:2px}
.card:hover{border-color:var(--muted)}
.node.sel .card{border-color:var(--accent)}
.head{display:flex;gap:12px;align-items:center;padding:13px 15px}
.head:focus-visible{outline:2px solid var(--accent);outline-offset:-2px}
.date{font:600 12px var(--fmono);color:var(--accent);white-space:nowrap}
.title{font:600 14.5px var(--fbody);flex:1}
.tags{display:flex;gap:6px;flex-wrap:wrap}
.tag{font:600 11px var(--fmono);padding:2px 8px;border-radius:5px;border:1px solid currentColor}
.tag.decision{color:var(--decision)}.tag.pivot{color:var(--pivot)}.tag.milestone{color:var(--milestone)}.tag.blocker{color:var(--blocker)}.tag.note{color:var(--note)}
.empty{color:var(--muted);padding:40px;text-align:center;font:14px var(--fmono)}
.panel{flex:0 0 420px;position:sticky;top:20px;max-height:calc(100vh - 40px);overflow:auto;background:var(--panel);border:1px solid var(--border);border-radius:12px;padding:18px}
.panel[hidden]{display:none}
.p-close{float:right;background:none;border:none;color:var(--muted);font-size:20px;cursor:pointer;line-height:1}
.p-close:hover{color:var(--accent)}
.p-date{font:600 12px var(--fmono);color:var(--accent)}
.p-title{font:800 18px var(--fdisp);margin:6px 0 12px;line-height:1.3}
.p-tags{display:flex;gap:6px;flex-wrap:wrap;margin-bottom:16px}
.p-sumhd{font:700 10px var(--fmono);color:var(--muted);text-transform:uppercase;letter-spacing:.14em;margin-bottom:9px;padding-bottom:7px;border-bottom:1px solid var(--border)}
.p-sum{font:13px/1.7 var(--fmono)}
.p-sum .row{margin:0 0 8px;display:-webkit-box;-webkit-line-clamp:2;-webkit-box-orient:vertical;overflow:hidden}
.p-sum b{color:var(--accent);font-weight:600}
.p-excerpt{color:var(--ink);opacity:.9;line-height:1.6;font:14px var(--fbody)}
.readmore{margin-top:16px;width:100%;background:transparent;border:1px solid var(--accent);color:var(--accent);font:600 13px var(--fmono);padding:10px;border-radius:7px;cursor:pointer}
.readmore:hover{background:rgba(255,255,255,.04)}
.p-full{margin-top:14px;border-top:1px dashed var(--border);padding-top:12px}
.p-full[hidden]{display:none}
.p-full p{color:var(--ink);opacity:.92;margin:11px 0;font:14px/1.7 var(--fbody)}
@media(max-width:760px){.panel{position:fixed;inset:auto 0 0 0;flex-basis:auto;max-height:80vh;border-radius:12px 12px 0 0;z-index:9}}

/* ============ THEME: terminal (default — BUIDL brand) ============ */
html[data-theme=terminal] body{background-image:linear-gradient(rgba(255,255,255,.014) 1px,transparent 1px);background-size:100% 3px}
html[data-theme=terminal] .bgfx{background:radial-gradient(900px 520px at 82% -12%,rgba(227,160,8,.07),transparent 60%)}
html[data-theme=terminal] .topbar{border:1px solid var(--border);border-radius:10px;background:linear-gradient(#161b22,#11151c);padding:30px 18px 16px;position:relative}
html[data-theme=terminal] .topbar::before{content:"\25CF \25CF \25CF";position:absolute;top:13px;left:16px;letter-spacing:3px;font-size:9px;color:#3a4452}
html[data-theme=terminal] h1{font-size:20px;letter-spacing:-.4px}
html[data-theme=terminal] h1::after{content:"\2587";color:var(--accent);animation:blink 1.1s steps(1) infinite;margin-left:3px}
@keyframes blink{50%{opacity:0}}

/* ============ THEME: blueprint (neon / glass) ============ */
html[data-theme=blueprint]{--bg:#05080f;--panel:rgba(12,19,33,.74);--panel2:rgba(12,19,33,.55);--border:rgba(120,170,255,.20);--ink:#dce8ff;--muted:#7e8db0;--accent:#3ce0d6;--accent2:#ff4d9d;--decision:#5aa9ff;--pivot:#ff5d73;--milestone:#3ce0a0;--blocker:#ffb24d;--note:#8aa0c8;--fbody:'SF Mono',ui-monospace,Menlo,Consolas,monospace;--radius:9px}
html[data-theme=blueprint] body{background:#05080f}
html[data-theme=blueprint] .bgfx{background:linear-gradient(rgba(120,170,255,.045) 1px,transparent 1px) 0 0/100% 40px,linear-gradient(90deg,rgba(120,170,255,.045) 1px,transparent 1px) 0 0/40px 100%,radial-gradient(900px 520px at 12% -6%,rgba(60,224,214,.12),transparent 60%),radial-gradient(760px 520px at 96% 8%,rgba(255,77,157,.10),transparent 60%)}
html[data-theme=blueprint] h1{text-transform:uppercase;letter-spacing:2.5px;text-shadow:0 0 22px rgba(60,224,214,.45)}
html[data-theme=blueprint] .chip{backdrop-filter:blur(6px)}
html[data-theme=blueprint] .chip.on{box-shadow:0 0 12px -2px currentColor}
html[data-theme=blueprint] #q{backdrop-filter:blur(6px)}
html[data-theme=blueprint] #q:focus{box-shadow:0 0 14px rgba(60,224,214,.3)}
html[data-theme=blueprint] .tl{border-left:2px solid transparent;border-image:linear-gradient(var(--accent),var(--accent2)) 1;box-shadow:-1px 0 16px -2px rgba(60,224,214,.35)}
html[data-theme=blueprint] .gap span{background:#070b14}
html[data-theme=blueprint] .node::before{background:#070b14;border:2px solid var(--note);box-shadow:0 0 10px var(--note),0 0 0 4px #070b14}
html[data-theme=blueprint] .node[data-tag=decision]::before{background:#070b14;border-color:var(--decision);box-shadow:0 0 12px var(--decision),0 0 0 4px #070b14}
html[data-theme=blueprint] .node[data-tag=pivot]::before{background:#070b14;border-color:var(--pivot);box-shadow:0 0 12px var(--pivot),0 0 0 4px #070b14}
html[data-theme=blueprint] .node[data-tag=milestone]::before{background:#070b14;border-color:var(--milestone);box-shadow:0 0 12px var(--milestone),0 0 0 4px #070b14}
html[data-theme=blueprint] .node[data-tag=blocker]::before{background:#070b14;border-color:var(--blocker);box-shadow:0 0 12px var(--blocker),0 0 0 4px #070b14}
html[data-theme=blueprint] .node.sel::before{background:var(--accent);box-shadow:0 0 18px 3px var(--accent),0 0 0 4px #070b14}
html[data-theme=blueprint] .card{backdrop-filter:blur(8px)}
html[data-theme=blueprint] .card:hover{border-color:rgba(60,224,214,.5);box-shadow:0 0 18px -4px rgba(60,224,214,.4)}
html[data-theme=blueprint] .node.sel .card{border-color:var(--accent);box-shadow:0 0 22px -4px rgba(60,224,214,.55)}
html[data-theme=blueprint] .date{text-shadow:0 0 10px rgba(60,224,214,.4)}
html[data-theme=blueprint] .title{font-family:var(--fmono);letter-spacing:-.2px}
html[data-theme=blueprint] .panel{border-color:rgba(60,224,214,.35);backdrop-filter:blur(14px);box-shadow:0 0 40px -8px rgba(60,224,214,.3)}
html[data-theme=blueprint] .p-sumhd{color:var(--accent2);border-bottom-color:rgba(255,77,157,.3)}
html[data-theme=blueprint] .p-date{text-shadow:0 0 10px rgba(60,224,214,.4)}
html[data-theme=blueprint] .readmore:hover{box-shadow:0 0 18px -2px rgba(60,224,214,.5);background:rgba(60,224,214,.07)}

/* ============ THEME: almanac (editorial serif ledger) ============ */
html[data-theme=almanac]{--bg:#15120c;--panel:#1c1812;--panel2:transparent;--border:#352d20;--ink:#ece3d2;--muted:#9c8e78;--accent:#c79a4b;--accent2:#c79a4b;--decision:#86a7c4;--pivot:#d2724f;--milestone:#94ac6e;--blocker:#d99a4e;--note:#9c8e78;--fdisp:'Iowan Old Style','Palatino Linotype','Hoefler Text',Georgia,serif;--fbody:'Iowan Old Style','Palatino Linotype',Georgia,serif;--fmono:'Avenir Next','Segoe UI',system-ui,sans-serif;--fui:'Avenir Next','Segoe UI',system-ui,sans-serif;--radius:2px}
html[data-theme=almanac] .bgfx{background:radial-gradient(120% 80% at 50% -10%,rgba(199,154,75,.06),transparent 55%),radial-gradient(90% 70% at 50% 120%,rgba(0,0,0,.45),transparent 50%)}
html[data-theme=almanac] .wrap{max-width:1080px;padding-top:52px}
html[data-theme=almanac] .topbar{border-bottom:2px solid var(--accent);padding-bottom:18px}
html[data-theme=almanac] h1{font-size:40px;letter-spacing:-.6px;font-weight:600}
html[data-theme=almanac] h1 .accent{font-style:italic;font-weight:500}
html[data-theme=almanac] .meta{letter-spacing:.13em;text-transform:uppercase;font-weight:600;font-size:11px}
html[data-theme=almanac] .chip{border-radius:2px;background:transparent;letter-spacing:.04em}
html[data-theme=almanac] .chip.ctrl{font-family:var(--fdisp);font-style:italic;font-size:13px}
html[data-theme=almanac] #q{background:transparent;border:none;border-bottom:1px solid var(--border);border-radius:0;font:italic 16px var(--fdisp)}
html[data-theme=almanac] .layout{gap:44px}
html[data-theme=almanac] .tlcol{counter-reset:e}
html[data-theme=almanac] .tl{border-left:1px solid var(--border);padding-left:32px}
html[data-theme=almanac] .gap{font:italic 14px var(--fdisp)}
html[data-theme=almanac] .node{counter-increment:e}
html[data-theme=almanac] .node::before{top:24px;width:9px;height:9px;left:-37px}
html[data-theme=almanac] .node.sel::before{box-shadow:0 0 0 4px var(--bg),0 0 0 7px rgba(199,154,75,.5)}
html[data-theme=almanac] .card{background:transparent;border:none;border-bottom:1px solid var(--border);border-radius:0}
html[data-theme=almanac] .card:hover{background:rgba(199,154,75,.045)}
html[data-theme=almanac] .node.sel .card{background:rgba(199,154,75,.07)}
html[data-theme=almanac] .head{display:grid;grid-template-columns:auto 1fr auto;gap:18px;align-items:baseline;padding:18px 14px}
html[data-theme=almanac] .date{font:600 11px var(--fmono);letter-spacing:.09em}
html[data-theme=almanac] .date::before{content:counter(e,decimal-leading-zero);display:block;font:400 26px var(--fdisp);color:#4a3f2c;margin-bottom:3px}
html[data-theme=almanac] .title{font:500 20px/1.32 var(--fdisp)}
html[data-theme=almanac] .tags{align-self:center;gap:9px}
html[data-theme=almanac] .tag{font:italic 13px var(--fdisp);padding:0 0 1px;border:none;border-bottom:1px solid currentColor;border-radius:0}
html[data-theme=almanac] .panel{flex-basis:400px;border:none;border-left:2px solid var(--accent);border-radius:0;padding:10px 28px 30px}
html[data-theme=almanac] .p-date{font:600 11px var(--fmono);letter-spacing:.12em}
html[data-theme=almanac] .p-title{font:500 27px/1.18 var(--fdisp)}
html[data-theme=almanac] .p-sumhd{font:italic 16px var(--fdisp);color:var(--accent);text-transform:none;letter-spacing:0;border-bottom:none;padding-bottom:0}
html[data-theme=almanac] .p-sum{font:14.5px/1.7 var(--fdisp)}
html[data-theme=almanac] .p-sum b{font-style:italic}
html[data-theme=almanac] .readmore{font:italic 15px var(--fdisp);border-radius:2px}
html[data-theme=almanac] .p-full p{font:15.5px/1.78 var(--fdisp)}
html[data-theme=almanac] .p-full p:first-child::first-letter{float:left;font:500 52px/.82 var(--fdisp);color:var(--accent);padding:4px 8px 0 0}
</style>
<style>/*BRAND_OVERRIDE*/</style>
</head><body>
<div class="bgfx" aria-hidden="true"></div>
<div class="wrap">
<header class="topbar">
<h1>Project Chronicle <span class="accent">— Timeline</span></h1><div class="meta" id="meta"></div>
</header>
<div class="bar">
 <span class="chip on" data-tag="decision">#decision</span><span class="chip on" data-tag="pivot">#pivot</span>
 <span class="chip on" data-tag="milestone">#milestone</span><span class="chip on" data-tag="blocker">#blocker</span>
 <span class="chip on" data-tag="note">#note</span>
 <input id="q" placeholder="search…" aria-label="search entries"><span class="chip ctrl" id="order">oldest first ↑</span>
</div>
<div class="layout">
 <div class="tlcol"><div class="tl" id="tl"></div></div>
 <aside class="panel" id="panel" hidden aria-live="polite"></aside>
</div></div>
<script>
const ENTRIES = /*ENTRIES_JSON*/ [];   /* skill replaces this array */
ENTRIES.forEach((e,i)=>e._i=i);
let asc=true, selId=null;
const active=new Set(['decision','pivot','milestone','blocker','note']);
const esc=s=>(s||'').replace(/[&<>]/g,c=>({'&':'&amp;','<':'&lt;','>':'&gt;'}[c]));
const tagsOf=e=>(e.tags&&e.tags.length?e.tags:['note']);
const chipHTML=ts=>ts.map(t=>`<span class="tag ${t}">#${t}</span>`).join('');
const days=(a,b)=>Math.abs((new Date(a)-new Date(b))/864e5);
const PX_PER_DAY=5, MIN_GAP=16, MAX_GAP=150, LABEL_DAYS=10;
function gapPx(d){return Math.max(MIN_GAP,Math.min(MAX_GAP,d*PX_PER_DAY))}
function gapLabel(d){d=Math.round(d);if(d<LABEL_DAYS)return'';
  if(d>=14)return`~${Math.round(d/7)} weeks quiet`;return`~${d} days`}
function summaryHTML(e){
  const row=(l,v)=>v&&v!=='—'?`<div class="row"><b>${l}:</b> ${esc(v)}</div>`:'';
  const s=row('Decisions',e.decisions)+row('Progress',e.progress)+row('Blocked',e.blocked)+row('Content-worthy?',e.contentWorthy);
  if(s)return `<div class="p-sum">${s}</div>`;
  const ex=(e.narrative||'').split(/\n{2,}/)[0].slice(0,260);
  return `<div class="p-excerpt">${esc(ex)}${(e.narrative||'').length>260?'…':''}</div>`;
}
function fullHTML(e){
  return (e.narrative||'').split(/\n{2,}/).map(p=>`<p>${esc(p)}</p>`).join('')||'<p class="p-excerpt">No narrative recorded.</p>';
}
function openPanel(e){
  selId=e._i;
  const p=document.getElementById('panel');
  p.innerHTML=`<button class="p-close" aria-label="close" onclick="closePanel()">×</button>
   <div class="p-date">${esc(e.date)}</div><div class="p-title">${esc(e.title)}</div>
   <div class="p-tags">${chipHTML(tagsOf(e))}</div>
   <div class="p-sumhd">Summary</div>${summaryHTML(e)}
   <button class="readmore" onclick="this.nextElementSibling.hidden=!this.nextElementSibling.hidden;this.textContent=this.nextElementSibling.hidden?'Read more ↓':'Show less ↑'">Read more ↓</button>
   <div class="p-full" hidden>${fullHTML(e)}</div>`;
  p.hidden=false; p.scrollTop=0;
  document.querySelectorAll('.node').forEach(n=>n.classList.toggle('sel',+n.dataset.i===selId));
}
function closePanel(){selId=null;document.getElementById('panel').hidden=true;
  document.querySelectorAll('.node.sel').forEach(n=>n.classList.remove('sel'));}
function render(){
  const q=document.getElementById('q').value.toLowerCase();
  let list=ENTRIES.filter(e=>tagsOf(e).some(t=>active.has(t)))
    .filter(e=>!q||((e.title||'')+(e.narrative||'')).toLowerCase().includes(q));
  list.sort((a,b)=>(a.date>b.date?1:-1)*(asc?1:-1));
  const tl=document.getElementById('tl');
  const dates=ENTRIES.map(e=>e.date).sort();
  document.getElementById('meta').textContent=
    ENTRIES.length?`${ENTRIES.length} entries · ${dates[0]} → ${dates[dates.length-1]} · showing ${list.length}`:'0 entries';
  if(!list.length){tl.innerHTML=ENTRIES.length?'<div class="empty">No entries match the current filters.</div>':'<div class="empty">No chronicle entries yet — run <code>/chronicle</code> to capture your first.</div>';return}
  tl.innerHTML=list.map((e,idx)=>{const ts=tagsOf(e);const primary=ts[0];
    let g='';
    if(idx>0){const d=days(list[idx-1].date,e.date);const lbl=gapLabel(d);
      g=`<div class="gap" style="height:${gapPx(d)}px">${lbl?`<span>${lbl}</span>`:''}</div>`;}
    return `${g}
  <div class="node${e._i===selId?' sel':''}" data-tag="${primary}" data-i="${e._i}">
   <div class="card" onclick="openById(${e._i})" tabindex="0" role="button" aria-label="${esc(e.date)} ${esc(e.title)}">
    <div class="head"><span class="date">${esc(e.date)}</span><span class="title">${esc(e.title)}</span>
    <span class="tags">${chipHTML(ts)}</span></div></div></div>`}).join('');
}
function openById(i){openPanel(ENTRIES[i])}
document.querySelectorAll('.chip[data-tag]').forEach(c=>c.onclick=()=>{
  const t=c.dataset.tag;c.classList.toggle('on');active.has(t)?active.delete(t):active.add(t);render()});
document.getElementById('q').oninput=render;
document.getElementById('order').onclick=function(){asc=!asc;this.textContent=asc?'oldest first ↑':'newest first ↓';render()};
document.addEventListener('keydown',e=>{
  if((e.key==='Enter'||e.key===' ')&&e.target.classList.contains('card')){e.preventDefault();e.target.click()}
  if(e.key==='Escape')closePanel();});
render();
</script></body></html>
```

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
