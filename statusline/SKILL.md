---
name: statusline
description: Set the Claude Code status bar project name and color
---

# /statusline

Set the Claude Code status bar project name and color.

## Usage

`/statusline <project-name> <color>`

## Parameters

- **First argument**: project name (required)
- **Second argument**: color (optional, defaults to current color)

Valid colors: `cyan`, `green`, `blue`, `yellow`, `red`, `magenta`

## Behavior

1. Parse arguments to extract project name and color
2. Write the updated values to `~/.claude/statusline.conf` in this exact format:
   ```
   PROJECT=<project-name>
   COLOR=<color>
   ```
3. Confirm the change to the user: show the new project name and color

If no arguments are provided, read and display the current `~/.claude/statusline.conf` values.

If only one argument is provided, treat it as the project name and keep the existing color.

If an invalid color is provided, tell the user the valid options and don't change anything.
