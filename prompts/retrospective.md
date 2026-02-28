---
description: Review saved reflection JSON files and extract recurring instruction debt
argument-hint: [DIR=~/.codex/reflection] [DURATION="1 week"]
---

Act as a principal engineer reviewing saved Codex reflection artifacts for trends, not just one-off session notes.

Your job is to review the reflection JSON files in `~/.codex/reflection/` unless `DIR` is provided, identify recurring workflow failures, and recommend the best changes to:
1. manual prompts
2. stored prompts under `~/.codex/prompts`
3. generic instructions
4. `AGENTS.md`

Review rules:
1. Prefer recurring instruction debt over one-off repo bugs or transient environment noise.
2. Group similar failures across sessions before suggesting a fix.
3. Separate real workflow contract gaps from local environment mismatches.
4. Use the reflection files themselves as evidence. Cite file paths in the response.
5. If a suggestion is only supported by one session, label it as low confidence.
6. If the same issue appears in multiple sessions, treat it as a higher-priority trend.
7. Do not rewrite prompts or instructions unless asked. This prompt is for diagnosis and prioritization.

Execution steps:
1. Read the available reflection JSON files recursively from the target directory (date-sharded layouts like `YYYY/MM/DD/*.json` are expected).
2. Determine the trailing analysis window from `DURATION` (natural language), defaulting to `1 week` if omitted. Examples: `1 week`, `7 days`, `48 hours`.
3. For each file, determine an effective timestamp in this order: valid `session_end_iso`; otherwise valid `session_start_iso`; otherwise the file modified time.
4. Analyze only files whose effective timestamp falls within the `DURATION` window ending at now.
5. Extract repeated failures, repeated root causes, and repeated improvement themes.
6. Map each recommendation to exactly one surface: `manual prompts`, `stored prompts`, `generic instructions`, or `AGENTS.md`.
7. Rank suggestions by expected leverage, not by novelty.
8. Call out any ambiguity in the existing wording that repeatedly caused drift.
9. Call out any missing fallback or stop condition that repeatedly caused wasted work.

Return a concise Markdown report with this shape:

```md
# Retrospective

## Top Trends
- Short bullet per recurring theme, with how many sessions it appeared in.

## Recommendations
### [Short title]
- Surface: manual prompts | stored prompts | generic instructions | AGENTS.md
- Priority: P0 | P1 | P2
- Confidence: high | medium | low
- Pattern: what kept recurring
- Why it matters: concrete impact
- Best fix: the smallest high-leverage change
- Suggested text: exact copy-paste wording when practical
- Evidence: /absolute/path/to/file.json, /absolute/path/to/file.json

## Environment Noise To Avoid Overfitting
- Short bullets for issues that should not dominate prompt or instruction changes unless they recur more often.
```

Be opinionated. Do not return a generic summary of every file. Focus on the smallest set of changes that would have prevented the most wasted iterations.
