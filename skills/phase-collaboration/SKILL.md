---
name: phase-collaboration
description: Use when spec.md Project Facts declares Mode: collaborative, or when the project has other contributors, pull requests, code review, or CI. Overrides the solo defaults for gitignore, branch pushing, merging, conflicts, and review handling.
---

# Collaboration Mode

Applies when `spec.md` Project Facts declares `Mode: collaborative`. Default is solo. These rules override the solo defaults in CLAUDE.md.

## Shared source of truth

- `spec.md` and `plan.md` become **committed**, not gitignored. A collaborator who cannot read the spec cannot build from it.
- Requirement changes go through a PR like any other file.
- `.claude/` becomes partially committed: `.claude/skills/`, `.claude/agents/`, and `.claude/settings.json` are shared so the team runs identical agent config. Only `.claude/settings.local.json` stays ignored.
- `.env`, `credentials.md`, and `CLAUDE.md` stay ignored always.

Collaborative `.gitignore`: drops `plan.md`, `spec.md`, and `.claude/`; adds `.claude/settings.local.json`.

## Branches are pushed

Push the phase branch as soon as its first commit lands, not at the end. Work nobody can see cannot be reviewed or recovered.

## Merge only through a PR

Never fast-forward locally into `main`.

- The PR body states: what the phase does, how to test it, and the Definition of Done result.
- Commit messages stay one line. Detail lives in the PR description, never in a commit body.
- **Squash-merge** so `main` keeps exactly one commit per phase. Review fixups never reach `main` history.
- At least one approving review before merge. Never self-merge unless the user explicitly allows it.
- Delete the remote branch after merge. Keep the local one.

## Keep the branch current

Rebase onto `main` before opening the PR if `main` has moved. **Never rebase after review has started**, since it destroys the reviewer's context. Merge `main` in instead.

## CI is part of Done

Where CI exists, the phase is not done until CI passes on the PR, not just locally. Never merge with failing or skipped CI.

## main stays green

If `main` breaks, fixing it takes priority over the current phase. Never force-push `main` or any pushed branch.

## Conflict prevention

Before starting a phase, fetch and check whether another open branch touches the same files. If so, flag it and ask before proceeding. Never resolve a conflict in someone else's code by guessing intent: stop and ask.

## Env vars are announced

A new variable in `.env.example` must be called out in the PR description so collaborators update their local `.env`. Silent additions break everyone else's checkout.

## Seeded accounts stay local

`credentials.md` is never committed. The seed script is committed; each collaborator runs it and generates their own.

## Review feedback is verified, not obeyed

Check a reviewer's suggestion technically before implementing it. If it is wrong, say so with reasoning. Never implement a change believed to be incorrect just because a reviewer asked.
