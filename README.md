# codex

This repository is where I keep my Codex configuration files and workflows, including reusable prompts and the top-level `AGENTS.md` instructions.

## What This Repo Contains

- `AGENTS.md`: global operating instructions for Codex in this repo.
- `prompts/`: reusable slash prompts.
  - `reflect.md`
  - `retrospective.md`

## Using `/reflect`

Use `/reflect` at the end of a Codex session to generate a structured reflection JSON with concrete workflow improvements.

- Command: `/reflect`
- Optional argument: `SESSION_ID=<codex-session-id>`
- Output behavior:
  - Saves a JSON file to `~/.codex/reflection/YYYY/MM/DD/HHMMSSmmmZ.json` (for example: `~/.codex/reflection/2026/02/28/201530123Z.json`)
  - Keeps `session_id` in the JSON payload for session identity
  - Returns the same JSON in the response

Typical use:

1. Finish a coding session.
2. Run `/reflect`.
3. Keep the saved JSON files as history for cross-session analysis.

## Using `/retrospective`

Use `/retrospective` to analyze multiple saved reflection JSON files and identify recurring instruction debt and high-leverage fixes. Run it at the end of the week to review patterns across sessions.

- Command: `/retrospective`
- Optional arguments:
  - `DIR=~/.codex/reflection` (or another reflection directory)
  - `DURATION="1 week"` (default if omitted; examples: `7 days`, `48 hours`)
- Output behavior:
  - Recursively scans reflection files under the target directory
  - Returns a concise Markdown report with recurring trends, prioritized recommendations, and evidence paths.

Typical use:

1. Accumulate several `/reflect` outputs over time.
2. At week end, run `/retrospective` (defaults to `DURATION="1 week"`).
3. Add this as a Codex App automation so the weekly review runs consistently.
4. Apply the highest-leverage prompt/instruction changes first.
