---
description: Produce a machine-readable end-of-session reflection (JSON) focused on reusable workflow improvements
argument-hint: [SESSION_ID=<codex-session-id>]
---

You are Codex. REVIEW the entire just-completed session (all user messages, your messages, files produced, edits, and manual steering).

Your goal is not only to summarize what happened, but to extract the smallest set of high-leverage changes that would have prevented wasted iterations next time.
Prioritize reusable instruction debt over one-off repo bugs. Whenever possible, identify whether a fix belongs in:
1. a manual prompt from the user
2. a stored prompt under `~/.codex/prompts`
3. generic instructions
4. `AGENTS.md`

Your primary job has two outputs:
1. Create and save one compact UTF-8 JSON file at an absolute path under `~/.codex/reflection/`.
2. Return that same JSON object as your final response, with no markdown and no extra text.

Identifier and path rules:
1. If `SESSION_ID` is provided as an argument, use it for the `session_id` field.
2. If `SESSION_ID` is not provided and the current Codex session exposes `CODEX_THREAD_ID`, use `CODEX_THREAD_ID` for the `session_id` field.
3. If neither `SESSION_ID` nor `CODEX_THREAD_ID` is available, generate a fallback identifier as `session-unknown-<UTC timestamp in YYYYMMDDTHHMMSSZ format>`.
4. When using the fallback identifier, set `session_id` to that fallback value, not `null`.
5. The output directory path must be UTC date-sharded as `~/.codex/reflection/YYYY/MM/DD/`.
6. The output filename must be UTC time-stamped as `HHMMSSmmmZ.json` (millisecond precision), independent of `session_id`.
7. If that filename already exists, append `-2`, `-3`, and so on before `.json` until unique.
8. The output file path must be `~/.codex/reflection/YYYY/MM/DD/HHMMSSmmmZ.json` (plus optional collision suffix when needed).

Persistence rules:
1. Before sending the final response, use an available write-capable tool to create `~/.codex/reflection/YYYY/MM/DD/` if needed and write the JSON file.
2. After writing the file, read it back and verify that it contains the same JSON value as the final response.
3. Do not emit a shell command for someone else to run.
4. Do not ask the user to save or copy anything.
5. If no write-capable tool is available, or if write verification fails, still return the JSON object only and record the problem in `root_causes` and `metadata.persistence_status`.
6. Set `metadata.persistence_path` to the actual sharded absolute file path used for persistence.

The JSON must conform to the schema below and must be compact (no pretty-printing required):

```json
{
  "session_id": "<string>",
  "session_start_iso": "<ISO-8601|null>",
  "session_end_iso": "<ISO-8601|null>",
  "summary_one_liner": "<string, <=20 words>",
  "what_worked": ["<string>", "..."],                  // 1–10 items
  "what_failed_or_was_painful": ["<string>", "..."],   // 1–10 items
  "root_causes": ["<string>", "..."],                  // 1–6 items
  "top_instruction_debts": [                           // 0–3 items; omit one-off noise
    {
      "surface": "<manual_prompt|stored_prompt|generic_instructions|AGENTS>",
      "problem": "<short recurring or high-leverage issue>",
      "why_it_matters": "<short concrete impact>",
      "best_fix": "<smallest credible change>",
      "confidence": "<high|medium|low>"
    }
  ],
  "concrete_improvements": [                           // 1–8 objects (include at least one edit of the user's original prompt when applicable)
    {
      "type": "<prompt|tool|step|test|guardrail|example|config>",
      "surface": "<manual_prompt|stored_prompt|generic_instructions|AGENTS>",
      "classification": "<instruction_gap|environment_mismatch|repo_specific|one_off>",
      "description": "<short summary>",
      "target_files": ["<absolute path>", "..."],      // 0–3 items when known
      "how_to_apply": "<copy-paste text or numbered steps>",
      "expected_value": "<what this prevents or speeds up>",
      "priority": "<P0|P1|P2>"
    }
  ],
  "example_prompts_to_use_next_time": ["<string>", "..."], // 0–3 items
  "stop_doing": ["<string>", "..."],                      // 0–6 items
  "quick_checklist_for_user": ["<string>", "..."],        // 2–6 items
  "actions_for_automation": [                             // optional
    {
      "action": "<string>",
      "triggerable_by": "<string>"
    }
  ],
  "metadata": {                                          // optional
    "files_created": ["<absolute path>", "..."],
    "languages": ["<lang>", "..."],
    "estimated_iterations_saved_next_time": <integer|null>,
    "persistence_path": "<absolute path|null>",
    "persistence_status": "<saved|write_tool_unavailable|verification_failed|null>",
    "missing_context": ["<short note>", "..."]          // optional
  }
}
```

Content constraints (ENFORCE strictly):
1. Final response must be JSON ONLY (no text before/after). Invalid output = fail.
2. Strings should be short and actionable (<=60 words unless otherwise constrained).
3. For "concrete_improvements" include at least one literal prompt-edit (exact text to paste) when applicable.
4. If you cannot determine timestamps, set the ISO fields to null.
5. If files were produced in the session, list them under metadata.files_created using absolute paths only.
6. If session was ambiguous, list ambiguous tokens/phrases under root_causes.
7. Limit arrays to the ranges specified above.
8. Set `metadata.persistence_path` to the intended absolute output path.
9. Set `metadata.persistence_status` to `saved`, `write_tool_unavailable`, or `verification_failed` as appropriate.
10. Prefer improvements that change prompts, instructions, or guardrails over improvements that only restate repo-specific implementation details.
11. Do not let a one-off environment problem dominate `top_instruction_debts` unless the real fix is a generic fallback rule.
12. Each `concrete_improvements` item must map to exactly one `surface`.
13. Use `classification=environment_mismatch` for local tooling or credential problems when the best fix is environment-specific rather than prompt-specific.
14. If important context is missing, record it in `metadata.missing_context` instead of hiding it behind nulls alone.

Execution requirements:
1. Build the JSON object first.
2. Attempt to save that JSON to `metadata.persistence_path` before the final response.
3. Read the saved file back and verify that it represents the same JSON value as the final response.
4. If persistence fails, still return valid JSON and record the exact failure cause in `root_causes`.
5. Final response must contain the JSON object only.
