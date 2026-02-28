# General

- I make manual edits between turns. Before editing an existing file, re-read it or otherwise confirm it has not changed since your last read. Treat unexpected user edits as authoritative and do not clobber them.
- In Go, skip argument checks such as `nil` context checks only when the function is internal, the precondition is strongly documented, and all callers are controlled such that violating the precondition is a programmer bug.

# For GitHub Pull Requests

- When asked to prepare or open a GitHub PR, first decide whether the work can be split into two or more independently reviewable slices. If it can, prefer a stacked PR series. If it cannot, use a single PR.
- Use a stack only when each PR represents one distinct concern and remains independently reviewable. Do not mix unrelated refactors, cleanup, and behavior changes in the same PR unless they are required for that slice to work.
- If splitting further would make a PR non-buildable, non-testable, or no longer independently reviewable, stop at the last clean boundary and explain the constraint instead of forcing a deeper stack.
- Use `git stack` to manage the branch stack. Run `git stack sync --pull` before opening or updating the stack so each branch is rebased onto the correct parent. Run `git stack sync --push` when updating an existing stack and all branches are ready to push.
- Use `gh pr create` to open PRs from the oldest slice to the newest slice.
- For the root PR in a stack, set the PR base to the protected branch or the repository's configured merge base.
- For each child PR in a stack, set the PR base to the direct parent branch from the previous PR in the stack, not the default branch.
- After creating each PR, capture its PR number and URL.
- Generic prompts under `~/.codex/prompts` must stay policy-only. Do not hardcode repo-specific generators or validation commands in them.
- Concrete review-readiness commands belong in the nearest applicable repo-local `AGENTS.md` for the current working subtree.
- The nearest applicable `AGENTS.md` is the deepest `AGENTS.md` whose directory still contains the files being changed.
- Repo-local `AGENTS.md` files that expect Codex to prepare or update PRs should define a `## Review Readiness` section with literal runnable commands under `Required generators:` and `Required validation:`. `Required generators:` may be `None.`
- If the nearest applicable `AGENTS.md` does not define a clear `## Review Readiness` section, stop and report the missing contract instead of inferring commands from build files, repo conventions, or shell history.
- Before any commit intended for review, any push, or any PR create or update, read the nearest applicable `AGENTS.md`, run the required generators for the touched code, inspect `git status`, and run the required validation for the affected area.
- If generators produce changes, review those changes and include them before treating the branch as ready for review.
- If required generators have not been run, generated output is still unreviewed, or focused validation is failing or unclear, stop and report the gap instead of committing, pushing, or opening or updating the PR.
- Every PR body must contain exactly these sections, in this order: `## Why`, `## Before`, `## After`, `## Evidence`.
- Every non-root PR must add exactly one preamble line before those sections: `Stacked on: [#<number>](<url>)`.
- In `Why`, describe why the change is being made and what reviewer context matters for this PR.
- In `Before`, describe the reviewer baseline for the current PR only: the pre-change behavior, bug, missing abstraction, or limitation being addressed.
- In `After`, describe only the change introduced by the current PR. Do not describe future stack layers there.
- In `Evidence`, prove the current PR using the best available artifact in this order of preference: screenshot, short terminal output, then tests.
- Include a screenshot only when there is already a renderable image URL or a committed image asset that GitHub can reference.
- If a screenshot is not practical, include concise terminal output in a fenced code block.
- If terminal output is not practical either, include the exact test command plus the relevant test files or named test cases that cover the change.
- Keep the evidence focused on the diff in the current PR. Do not dump long logs; include only the lines that demonstrate the change.
- When updating an existing stack, preserve the PR ordering and refresh the `Stacked on:` line if the parent PR changes.
- These PR rules only change how PRs are split, opened, and described. All repository-specific push instructions still apply.

Use this exact PR body shape for non-root PRs:

```md
Stacked on: [#123](https://github.com/OWNER/REPO/pull/123)

## Why
Describe why this change is being made and what reviewer context matters for this PR only.

## Before
Describe the pre-change state for this PR only.

## After
Describe the change in this PR only.

## Evidence
Preferred: screenshot.
Otherwise: short terminal output in a fenced block.
Otherwise: exact test command plus the specific tests that cover the change.
```

For a root PR, omit the `Stacked on:` line and keep the same four sections in the same order.

Use this shape for repo-local review-readiness commands:

```md
## Review Readiness

- Required generators:
  - `bazel run //:gazelle`
  - Or `None.`

- Required validation:
  - `bazel build //path/to:target`
  - `go test ./path/...`

- Notes:
  - Optional repo-specific constraints or reminders.
```

