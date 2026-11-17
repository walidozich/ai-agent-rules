---
name: project-bootstrap
description: Use when starting a new project, or when spec.md or plan.md is missing, empty, or incomplete. Defines the bootstrap sequence (discussion, spec.md, grill, plan.md) that must complete before any phase begins, including what the grill must resolve and the required Project Facts block.
---

# Project Bootstrap (Opening Game)

Phases do not begin until this sequence is complete. If asked to start work when `spec.md` or `plan.md` is missing or incomplete, stop and say so.

1. **Discussion.** Open-ended exploration producing raw requirements.
2. **`spec.md`.** Written up as the approved design, opening with the Project Facts block below.
3. **Grill.** Stress-test `spec.md` before any code exists.
4. **`plan.md`.** The grilled spec decomposed into numbered phases. This is the living tracker: phases checked off with status notes.

## Required Project Facts block

`spec.md` MUST open with `## Project Facts` covering:

- **Stack:** frameworks, languages, database, ORM, package manager, runtime versions
- **Architecture:** monolith / microservices / clean architecture, and key module boundaries
- **Language & Layout:** UI language, text direction (LTR/RTL), fonts
- **Platform:** target platforms, OS focus, build tools, Docker or native
- **Performance targets:** see below
- **Mode:** `solo` or `collaborative`. Defaults to solo if absent.
- **Constraints:** anything non-obvious limiting design choices

If `spec.md` exists without this block, stop and ask. Never infer the stack from the file tree.

## What the grill must resolve

- Ambiguous requirements
- Data model shape
- Auth model
- Third-party dependencies
- **Performance targets:** expected data volume per table at one year, expected concurrent users, which endpoints are hot, acceptable p95 latency, and whether anything needs caching or background processing

Anything unresolved is recorded in `spec.md` as an explicit open question, never skipped silently. Where the user has no answer on performance, record "unbounded, revisit at Final Phase".

## Decisions owed at bootstrap

Both are Mandatory Stops. State a recommendation with reasoning, then wait.

- **Docker or native?** If dockerized, set up `Dockerfile` and `docker-compose.yml` in early setup phases. If not, skip container tooling entirely and document native setup and run steps.
- **Solo or collaborative?** Recorded as `Mode:` in Project Facts. Collaborative changes the gitignore set and the merge flow (load the `phase-collaboration` skill).

## First phase

Phase 1 is always a **tracer bullet**: a thin slice through every layer (database to API to UI) proving the stack works end to end. It deliberately breaks the one-layer instinct, and only once.

## Setup files created here

- `.gitignore` with the required entries
- `.env` and `.env.example`, created proactively
- `SETUP.md`
- Lint and format config, if the stack has none
