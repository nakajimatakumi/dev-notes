---
name: git-push
description: Safely push existing commits from this project to its configured Git remote without creating a commit. Use when the user asks to push, publish committed changes, set the upstream for the current branch, or send existing local commits to the connected remote.
---

# Git Push

Push existing commits from this project. Do not stage or commit changes.

## Repository guard

1. Resolve the repository root with `git rev-parse --show-toplevel`.
2. Confirm the root is the project that contains this skill under `.agents/skills/git-push`.
3. Stop if Git is unavailable, the directory is not a worktree, or the resolved root is a different repository. Never accept an alternate repository path for this skill.

## Workflow

1. Inspect `git status --short`, `git branch --show-current`, `git remote -v`, and `git status --branch --short`.
2. Stop on a detached HEAD, unresolved conflicts, or an in-progress merge/rebase that makes the push ambiguous.
3. Determine the current branch and its upstream with `git rev-parse --abbrev-ref --symbolic-full-name @{upstream}` when available.
4. Review the outgoing commits. With an upstream, inspect `git log --oneline @{upstream}..HEAD`. Without one, inspect the branch history and remote branches before proceeding.
5. Use the configured upstream when present. If no upstream exists, use `origin` when it is the single unambiguous configured destination and run `git push -u origin <current-branch>`. Stop and ask when the destination is ambiguous.
6. Run a normal `git push`; never force. Treat authentication, permissions, remote rejection, or hook failure as blockers and report them without changing history.
7. Verify with `git status --branch --short` and report the remote, branch, pushed commit range, and result.

## Safety

- Never stage, commit, amend, rebase, reset, merge, force-push, delete remote refs, push tags, or change Git configuration unless explicitly requested as a separate action.
- Do not claim success unless the push command succeeds and the branch status confirms it.
- Redact credentials and remote URLs containing embedded secrets.

