---
name: qa-swarm
description: "App-agnostic QA swarm — auto-discovers features (incl. state-driven SPAs), mines existing tests + a persistent QA knowledge base, then launches parallel Sonnet Playwright agents with evidence-gated verdicts. Usage: /qa-swarm [--plan] [--quick] [--agent <name>] [--headed]"
---

# QA Swarm

You orchestrate an app-agnostic QA swarm: explore the codebase, build (or reuse) a feature map, write a **shared test harness**, then launch parallel browser-test agents against a running instance. Results accumulate into a living **QA knowledge base** that guides every future run.

**You are the orchestrator (Opus). Leaf test agents run on Sonnet** — the shared harness does the hard infrastructure work, so agent scripting is mechanical. Evidence-gating backstops any agent that flails.

## Argument Parsing

- `/qa-swarm` → if `qa-reports/QA-KNOWLEDGE.md` exists, run in **returning mode** (seed from KB, validate, run autonomously). Otherwise **first-run mode** (discover → grill → run).
- `/qa-swarm --plan` → force full re-discovery and a fresh user interview, rebuilding KB sections even if a KB exists.
- `/qa-swarm --quick` → run only P1 (critical-path) scenarios.
- `/qa-swarm --agent <name>` → rerun a single named agent from the current plan.
- `/qa-swarm --headed` → run browsers headed (visible) instead of headless.

Flags compose (e.g. `--quick --headed`).

---

## Phase 0: Knowledge Base Check

Look for `qa-reports/QA-KNOWLEDGE.md`.

- **Absent** → first-run mode. Do full discovery (Phase 1) and grill the user (Phase 3).
- **Present** (and no `--plan`) → returning mode:
  1. Load the KB's feature map, selector map, behavior notes, flaky list, and last-run status table.
  2. **Validate**: before planning, spot-check the KB's key selectors against the live app (a fast headless probe). Stale selectors (not found) get re-discovered and the KB section flagged for update.
  3. Run **autonomously** — skip the interview. Only pause to ask the user if validation finds **major drift**: multiple broken selectors, a removed feature area, or an obvious new feature area not in the KB.

---

## Phase 1: Discovery

Goal: a **feature map** — feature areas with the concrete routes/components/selectors/handlers that belong to each.

First classify the app:
- **Routed / backend app** (React Router, Next.js, Flask, Django, Express, etc.) → discover routes, pages, auth, API endpoints, components.
- **State-driven SPA** (one HTML entry, no server routes, client-side state) → **UI-interaction discovery mode**: features are *user actions*, not URLs. Map:
  - Interactive surfaces: buttons, links, modals, dropdowns, editors, forms, toggles, menus.
  - DOM-mutating event handlers (`onclick`, `addEventListener`, framework handlers).
  - Persistence: `localStorage`/`sessionStorage`/`IndexedDB` keys read/written.
  - Async/timing behavior: debounce/throttle timers, animations, lazy loads, fetch calls.
  - Third-party widgets (e.g. Monaco editor, charts) and their ready-state signals.

Always also read: README/docs, config/env/feature flags, and **existing tests**.

### Mine existing tests and the KB
If test files exist (`tests/`, `*.spec.*`, `*.test.*`, `e2e/`, Playwright/Cypress configs):
- Extract **already-verified selectors, URLs, and covered scenarios** → seed the harness selector map and feature map.
- Note what's covered (avoid duplication) and **flag gaps** the existing tests miss — new agents focus there.
If a KB exists, its selector/feature/behavior sections seed discovery directly (subject to Phase 0 validation).

Produce the feature map (feature area → selectors/handlers/behavior notes).

---

## Phase 2: Test Plan Generation

For each feature area, generate scenarios covering: **happy path**, **validation** (required fields, constraints, error messages), **state transitions** (navigation, persistence across reload), **edge cases** (empty states, long/special-char inputs, rapid actions).

Always include a cross-cutting **edge-case-tester** agent:
- XSS injection in text inputs (`<script>alert(1)</script>` and attribute-breakouts)
- Unicode/emoji handling in all text fields
- Rapid double-submit / double-click on buttons and forms
- Page refresh mid-flow (state recovery)
- Browser back/forward behavior
- Console + page-error monitoring (attach listener at session start)
- Empty/null submissions

Assign priority per scenario: **P1** critical path · **P2** important (validation/error handling) · **P3** edge/polish.

### Group into agents
- One agent per feature area (scales with complexity).
- One always-present `edge-case-tester`.
- **Soft cap: ~6 parallel agents + the edge-case agent.** If the plan needs more, either fold related areas together or ask the user to confirm a larger swarm. Never silently exceed the cap.
- Name agents descriptively (`editor-tester`, `sidebar-tester`, …).

---

## Phase 3: User Interview (first run / `--plan` only)

Present the full plan as tables grouped by agent (`# | Scenario | Priority | Description`), with totals. Then ask **one** open question via AskUserQuestion:

> "Here's what I plan to test. What would you add, remove, or change? Any features I missed? Any known-flaky areas to focus on?"

Incorporate feedback. In **returning mode, skip this** unless major drift was detected in Phase 0.

### Target selection (deployed vs local)
Prefer a **detected deployment** (GitHub Pages via `gh api repos/{owner}/{repo}/pages`, Vercel/Netlify config, etc.).
- Compute drift: compare the working tree against what's deployed (`git status`, unpushed commits, dirty files that affect the app).
- If drift exists, **warn explicitly**: "Testing committed/deployed code — your local changes to X are NOT included." Let the user choose deployed vs local.
- If no deployment is detected, serve locally: auto-start a static server (`python3 -m http.server <port>` for static apps, or the project's dev/start script) in the background, health-check it, and tear it down at the end.

Health-check the chosen URL (`curl -sI`) before proceeding.

---

## Phase 4: Environment + Harness

### Harness language — auto-detect
- `package.json` deps/devDeps include `playwright`/`@playwright/test`, or tests are `.js`/`.ts` → **Node/JS harness**.
- A Python env with `playwright` importable → **Python harness**.
- Neither → offer to install the one that matches the repo (`npm i -D playwright && npx playwright install chromium`, or `pip install playwright && playwright install chromium`). Confirm before installing.

Verify the chosen runtime actually works (import/require check) before spawning agents.

### Write the shared harness FIRST
Generate one harness module (`qa-reports/<date>/qa_harness.{js,py}`) that every agent imports. It must provide:
- **Browser lifecycle**: launch (headless unless `--headed`), context, page.
- **Console + page-error capture**: listeners attached at launch; collected into an array the agent reports.
- **Verified selector map**: the selectors discovered/mined in Phase 1, as named constants — agents reference these, not raw strings.
- **Evidence helpers**: `assert_visible`, `assert_text`, `assert_storage(key, matcher)`, etc. Each returns a structured result `{name, status, evidence, screenshot}`.
- **Screenshot helper**: writes to `qa-reports/<date>/screenshots/<agent>-<scenario>.png`. Called on **both PASS and FAIL** (evidence for every verdict).
- **Deterministic waits**: Playwright auto-waiting + explicit helpers for discovered async behavior, e.g. `wait_for_debounce(ms)` for autosave/debounced inputs, `wait_for_widget_ready()` for third-party editors. **No random human-pacing sleeps.**
- **Verdict recording + report writer**: collects per-scenario results and emits the agent's markdown report.

**Evidence gate**: a scenario that records a PASS/FAIL with no asserted evidence is auto-downgraded to **SKIP** by the harness. Agents cannot claim success without an assertion.

**Specific-value rule**: every assertion must encode the scenario's *specific predicted value*, never a weaker proxy that's easier to pass. If the scenario claims "the title ends on problem 4", assert `title.includes('4.')` — NOT `title.length > 0`. If it claims "complexity shows O(n)", assert the exact Big-O — not "some text exists". A weak proxy satisfies the evidence gate while hiding the very bug the scenario was meant to catch. State this rule in every agent prompt.

### Report directory
```bash
QA_DATE=$(date +%Y-%m-%d); QA_DIR="./qa-reports/${QA_DATE}"
mkdir -p "$QA_DIR/screenshots"
```

### Git policy
Ensure `.gitignore` ignores run artifacts but **not** the KB:
- Add `qa-reports/*/` (dated run dirs + screenshots) to `.gitignore` if absent.
- Keep `qa-reports/QA-KNOWLEDGE.md` tracked.
- **Never auto-commit.** Stage/ignore only; tell the user what to commit.

---

## Phase 5: Agent Spawning

Spawn all agents in **a single message** (parallel), each via the Agent tool with `model: sonnet` and `subagent_type: general-purpose`. Respect the ~6 soft cap.

### Agent prompt template
```
You are a Sonnet QA agent testing the "{feature_area}" of {APP_NAME} at {APP_URL}.

Harness: import the shared harness at {HARNESS_PATH}. It provides browser launch,
the verified SELECTORS map, evidence assert_* helpers, screenshot capture, and
deterministic wait helpers ({wait helpers}). DO NOT re-implement these. Use Playwright
via {Bash command to run the script}; MCP browser tools are NOT available.

Write a script at {QA_DIR}/agent-{name}.{ext} that imports the harness and runs these
scenarios, then run it via Bash and report.

TEST SCENARIOS ({N}):
{numbered scenarios with explicit pass/fail criteria tied to real selectors/URLs/expected content}

RULES:
- Every verdict MUST come from an assert_* helper (real DOM/state evidence). A verdict
  without an assertion is auto-SKIPped by the harness — never fake a PASS.
- Each assertion MUST check the scenario's SPECIFIC predicted value (e.g. `title.includes('4.')`,
  exact Big-O, the exact marker string) — never a weaker proxy like "non-empty" or "exists".
  A loose proxy passes the gate while hiding the bug you were sent to find.
- Use SELECTORS.* constants, not raw selector strings.
- Use the wait helpers for async behavior (e.g. wait_for_debounce for autosave); no sleeps.
- Screenshots are captured for every verdict by the harness.

REPORT: write {QA_DIR}/agent-{name}-report.md — summary table (PASS/FAIL/SKIP counts),
per-scenario verdict + evidence + screenshot path, and any console/page errors captured.
Return the report path and the counts.
```

---

## Phase 6: Aggregation + Recommendations

After all agents return, write `{QA_DIR}/qa-report-{YYYY-MM-DD-HHmm}.md`:
- **Summary table**: agent × pass/fail/skip/total, plus grand total.
- **Critical failures**: each FAIL with agent, scenario, evidence, screenshot path.
- **Regressions** (returning mode): scenarios that were PASS in the last KB run and are now FAIL (and fixes: FAIL→PASS).
- **Skipped**: each SKIP + reason (incl. evidence-gate downgrades).
- **Console/page errors**: uncaught exceptions across all agents.
- **Ranked fix recommendations**: HIGH/MED/LOW, each mapping the failure to the **likely culprit file + selector/line** (use discovery knowledge — don't just restate the symptom).

Print a concise terminal summary (the table + critical failures).

---

## Phase 7: Update the Knowledge Base

Write/update `qa-reports/QA-KNOWLEDGE.md` (the distilled, always-current memory — **not** a per-run dump):

```markdown
---
app: {name}
target: {deployed-url or local}
last_run: {ISO timestamp}
last_run_report: qa-reports/{date}/qa-report-{...}.md
---

# QA Knowledge Base — {App}

## App Model
{routed/SPA; serving; auth model}

## Feature Map
{feature area → what it does → key user actions}

## Verified Selectors
{name → selector, last confirmed {date}}

## Behavior Notes
{debounce timings, third-party widget quirks, persistence keys, gotchas}

## Known-Flaky Areas
{area → symptom → mitigation}

## Last-Run Status
| Scenario | Agent | Priority | Status | Since |
{used for regression diffing next run}

## Coverage Gaps / TODO
{areas not yet tested}
```

Update — don't blindly append: refresh selectors confirmed/broken this run, move newly-stable areas out of flaky, record regressions, and rewrite Last-Run Status. Stage the KB (tracked); leave run artifacts gitignored.

---

## Behavioral Notes
- Discovery quality drives everything — spend real effort on the feature map and selector map.
- Use **real selectors** from code/tests/KB, never guesses.
- Scale agents to complexity within the ~6 soft cap; fold or confirm beyond it.
- The edge-case-tester is always present, even for simple apps.
- Evidence for every verdict; no random sleeps; never auto-commit; KB is the memory that makes each run smarter than the last.
