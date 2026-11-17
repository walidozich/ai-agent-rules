---
name: phase-final
description: Use when plan.md has no unchecked phases left, or when the user says the project is finished, ready to ship, ready to hand over, or ready for a client. Defines the mandatory Final Phase (hardening and handover) covering full regression, whole-surface security, performance, scaffolding removal, and the fresh-clone test.
---

# Final Phase (Hardening & Handover)

When `plan.md` is exhausted the project is not done. This phase is mandatory, runs on branch `phase/final-hardening`, and is never skipped. Phases that pass individually break each other, and nothing else in the workflow ever checks the whole.

It is exempt from Phase Sizing, being a full sweep.

## Checks

**Full regression.** The entire test suite, not just the last phase's tests. Real output.

**End-to-end journeys.** Walk the primary user flows manually in the real app, not via unit tests. Evidence required.

**Full security pass, not proportional.** Every endpoint's authorization, every input, whole-repo secret scan, dependency audit. The per-phase passes were scoped by trigger; this is the only look at the entire surface.

**Performance pass.** N+1 queries, missing indexes on foreign keys and filtered columns, unbounded list queries, pagination caps, payload sizes, heavy synchronous work on request paths. Checked against the targets recorded in `spec.md`.

**Scaffolding removal.** Remove fake or mocked auth, seed backdoors, bypass flags, and debug switches. The Authentication Strategy rule deliberately plants an auth bypass in the codebase, so removing it must be an explicit, verified step or it ships.

**Env audit.** Every `.env.example` var still used, every used var documented, zero `REPLACE_ME` remaining.

**Docs reconciliation.** `README.md`, `SETUP.md`, OpenAPI spec, `spec.md` Project Facts, and every Mermaid diagram checked against what was actually built. Drift is guaranteed after N phases.

**Fresh-clone test.** Clone to a clean directory, follow `SETUP.md` literally, confirm it runs. The only honest test of the documentation.

## Handling findings

**Findings become new numbered phases in `plan.md`, never inline fixes.** Otherwise the finish line degrades into a long unreviewed free-for-all, which is where discipline usually collapses.

The Final Phase re-runs after those phases clear.

## Later feature work

Update `spec.md` first, re-grill, append new phases to `plan.md`, and the Final Phase runs again at the end.
