---
name: phase-recovery
description: Use when a phase grows beyond its planned size mid-implementation, when an earlier completed phase turns out to be wrong or needs abandoning, or when context was compacted mid-phase and the working state must be reconstructed. Covers phase overrun, abandonment and re-planning, and compaction recovery.
---

# Phase Recovery

Three exception paths for phases that do not run straight through. Each one is a stop, never a silent workaround.

---

## Phase Overrun

Sizing is re-checked during implementation, not only at planning.

- **Stop as soon as a phase breaches its size.** Signals: more than roughly 10 files touched, the work splitting into parts each independently useful, the Definition of Done needing sub-checklists, or a second "and" appearing in its description.
- Report what was found, why it is bigger than planned, and how it should split, with a recommendation.
- **Default action is to split.** Commit the coherent portion finished so far as the now-smaller phase, and append the remainder to `plan.md` as new numbered phases.
- **Never silently carry on.** A phase that quietly triples in size destroys reviewability, which is the entire purpose of the phase unit.
- If the overrun reveals that `spec.md` underestimated the work, update `spec.md` in the same stop.

---

## Phase Abandonment & Re-Planning

A phase can be wrong, not merely incomplete.

- **If a phase reveals a completed earlier phase was wrong, stop immediately.** Do not build a workaround on top of it, and do not quietly refactor it as part of the current phase.
- State: which phase is affected, what specifically is wrong, what now depends on it, and the options with a recommendation. Typical options are revert and re-plan, patch forward with a corrective phase, or accept the flaw and document it.
- **The user decides.** Never rewrite completed phases on independent judgment.
- If the current phase is abandoned: keep its branch, record the phase in `plan.md` as abandoned with the reason, and append any replacement as a new numbered phase. **Never renumber or edit the history of existing phases.**
- Corrective work is always its own phase, never folded into an unrelated one.

---

## Context Compaction Recovery

Context will compact mid-phase on long work. Memory of the phase is not durable. `plan.md` and git are.

- **Working state lives in `plan.md`, never only in context.** The active phase carries a live sub-block: scope, files touched so far, which Definition of Done items are satisfied and which are not, and any open question awaiting an answer.
- Update that block **as each item completes**, not at the end of the phase.
- **After any compaction, reconstruct before continuing.** Re-read `spec.md`, the active phase block in `plan.md`, and run `git status` plus `git diff main...HEAD` on the phase branch. State in one line what was recovered and what remains.
- **Never claim a Definition of Done item is satisfied from memory after a compaction.** If it is not recorded in `plan.md` or visible in the diff, re-verify by running the check.
- If recovered state is ambiguous or contradicts `plan.md`, stop and ask rather than guessing.
