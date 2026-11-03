---
name: git-commit
description: Generates Conventional Commit messages from selected Git changes. Use when user asks to draft a commit message from current worktree; never stage or commit.
---

# Git Commit Message

## Workflow

1. Run `git status --short`. Read selected diffs with `git diff --cached`, `git diff`, and untracked file contents when in scope. Treat untracked files as unstaged. Use no unrelated files, docs, history, branch names, or issue/PR metadata.
2. Pick scope:
   - No changes: report, stop.
   - Staged only: use cached diff; proceed.
   - Staged + any unstaged/untracked: ask, "Staged only, or all changes (staged + unstaged + untracked)?" Wait.
   - Unstaged only, including only untracked: use all; treat as staged, never stage.
   - Staged-only excludes unstaged edits and untracked files. For partially staged files, use cached diff only; never use working-tree contents.
   - All changes = cached + unstaged diffs + in-scope untracked contents.
3. Inspect all selected changes. Identify primary change and verified details. Use paths only for optional scope, not behavior. Unresolved conflicts: stop; ask user to resolve. Binary, opaque, or insufficient diffs: ask or report limit; never guess.
4. Compose:
   - Title: `type(scope): imperative summary`; scope optional. Choose type from primary change (`feat`, `fix`, `refactor`, `perf`, `test`, `docs`, `build`, `ci`, `chore`, etc.). Keep concise and factual.
   - Body: 2-4 concise bullets on distinct, verified changes. Fewer if warranted; no padding or assumed motives.
   - Footer: only when supported. Verified breaking change: add `!` to title and `BREAKING CHANGE: ...`. Issue footer: ID must appear in selected changes.
5. Output raw message only: title, bullets, optional footer. No preamble, explanation, or code fence.

Never run `git add`, `git commit`, or commands that change files or index.

## Shape example

Example only; derive actual wording from selected changes.

```text
feat(labels): support bulk updates

- Apply one supplied value across selected labels.
- Preserve unrelated label fields.
```
