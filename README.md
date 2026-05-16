# Skills

Custom skills for [Claude Code](https://docs.anthropic.com/en/docs/claude-code).

## What are skills?

Skills are markdown instruction files that teach Claude Code new behaviors via slash commands. Drop a `SKILL.md` into `~/.claude/commands/` and it becomes a `/command` you can invoke in any session.

## Skills in this repo

| Skill | Command | What it does |
|---|---|---|
| [chronicle](./chronicle/) | `/chronicle` | Auto-capture your project's story — decisions, pivots, dead ends, milestones. Export onboarding briefings for agents and humans. |
| [diagram](./diagram/) | `/diagram` | Generate standalone Mermaid diagram HTML files that open in the browser. Auto-selects the best diagram type. |
| [grill-me](./grill-me/) | `/grill-me` | Interview you relentlessly about a plan or design until every branch of the decision tree is resolved. Interactive multiple-choice questions. |
| [grill-with-docs](./grill-with-docs/) | `/grill-with-docs` | Same grilling, but also challenges your plan against the existing domain model, sharpens terminology, and updates CONTEXT.md / ADRs inline. |
| [slides](./slides/) | `/slides` | Create polished, self-contained HTML slide decks. Supports brand-aware styling by extracting identity from a company URL. |
| [statusline](./statusline/) | `/statusline` | Set the Claude Code status bar project name and color. |

## Installation

### Install a single skill

Copy the `SKILL.md` file into your Claude Code commands directory:

```bash
# Example: install the chronicle skill
cp chronicle/SKILL.md ~/.claude/commands/chronicle.md
```

### Install all skills

```bash
for dir in */; do
  if [ -f "$dir/SKILL.md" ]; then
    cp "$dir/SKILL.md" "$HOME/.claude/commands/${dir%/}.md"
  fi
done
```

The skill becomes available as a slash command immediately — no restart needed.

## Credits

- `grill-me` and `grill-with-docs` are modified from [Matt Pocock's skills](https://github.com/mattpocock/skills), with interactive multiple-choice questioning via AskUserQuestion added on top.
- `chronicle`, `diagram`, `slides`, and `statusline` are original.
