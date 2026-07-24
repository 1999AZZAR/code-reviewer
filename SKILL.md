---
name: code-reviewer
description:
  Review local code changes, commits, Git ranges, branches, or GitHub pull requests. Use
  when asked for a code review, PR review, security review, regression check, or approval decision.
---

# Code Reviewer

Find actionable defects in changed code. Prioritize correctness, security, regressions, and missing tests over summaries and style preferences.

## Workflow

### 1. Determine Scope

- For a PR number or URL, resolve its repository, base SHA, and head SHA. Inspect metadata, reviews, issue comments, review comments, checks, commits, changed-file list, and the complete base-to-head diff with `gh pr view`, `gh pr checks`, `gh pr diff`, and `gh api` where needed.
- For local work, inspect repository instructions, status, staged and unstaged diffs, and relevant untracked files.
- For a single non-merge commit, compare it with its parent; compare a root commit with Git's empty tree. For a merge commit, identify which parent represents the intended baseline and ask if that cannot be inferred safely.
- For a branch, use its PR base when one exists, then its configured upstream when appropriate. Otherwise ask for the base rather than guessing from local or remote defaults. Compare the chosen base's merge base with the branch tip.
- For an explicit range, preserve the user's endpoints and `..` or `...` semantics rather than replacing them with an inferred range.
- If no target is given, review current staged and unstaged changes. State the chosen scope.

### 2. Build Context

- Read project instructions and the changed files in full, not only diff hunks.
- Trace affected callers, types, schemas, configuration, migrations, and tests where behavior depends on them.
- Understand the intended behavior from the request, issue, PR description, and existing implementation. Do not accept the description as proof that the code works.
- For a PR that is not checked out, read files at the head SHA through GitHub or a detached temporary worktree. Never analyze current-branch file contents as if they were the PR revision.
- Preserve the user's index, worktree, refs, and untracked files. Record status before and after verification; do not stash, clean, reset, restore, or run mutating checks in the user's worktree. Use and remove a detached temporary worktree when execution may write files.
- Verify diff completeness against the changed-file list, including deleted, renamed, binary, generated, and submodule changes. If `gh pr diff` is unavailable or incomplete, fetch the refs and compute the merge-base-to-head diff locally with `git diff <base-sha>...<head-sha>`. Disclose anything that remains unreviewed.

### 3. Analyze and Verify

Check for:

- Incorrect behavior, broken invariants, edge cases, concurrency issues, and compatibility regressions.
- Trust-boundary failures: injection, authorization, secret exposure, unsafe deserialization, path traversal, and insecure defaults.
- Data loss, migration or rollback hazards, resource leaks, and failure-path handling.
- Performance regressions that are plausible for expected inputs.
- Missing tests for changed behavior, failure paths, and previously reported bugs.
- Maintainability problems only when they create a concrete defect risk.

Run the smallest relevant project-native checks against the reviewed revision when safe and feasible. Discover commands from project configuration or documentation; never assume an ecosystem or hard-code `npm run preflight`. Report commands and outcomes. Review remains necessary even when checks pass.

### 4. Report Findings

List findings first, ordered by severity. Each finding must include:

- Severity: `Critical`, `High`, `Medium`, or `Low`.
- A concise defect title.
- Precise file and line reference.
- The triggering scenario and observable impact.
- A concrete fix direction, without rewriting the patch unless requested.

Do not report speculative issues without a realistic failure path. Avoid style-only nitpicks unless they violate an explicit project standard or conceal a defect. Group repeated instances under one root-cause finding.

After findings, list open questions or assumptions, then a brief verification summary. If there are no findings, say so explicitly and note residual testing gaps. End with `Approve` or `Request changes`; any unresolved blocking finding requires `Request changes`.
