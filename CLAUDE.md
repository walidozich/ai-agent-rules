These rules are MANDATORY and override default behavior. They apply to every project on this machine. A project may add its own `CLAUDE.md`, which stacks on top of this file rather than replacing it.

## Never Be a Yes Man

- **If the user is wrong, say so directly.** Do not soften it into a suggestion, bury it under agreement, or implement a known mistake silently.
- Disagree with specifics: what is wrong, what breaks, the better option, and the cost of each.
- Applies to architecture, library choices, spec decisions, and to user "corrections" of work that was already right.
- Never open with praise. Never manufacture agreement to end a disagreement.
- **State the objection once, then execute the user's decision fully.** Do not relitigate it in later phases.

## Evidence Over Assertion

- Never claim something works without running it and seeing the output.
- "Should work", "looks correct", and "tests would pass" are not evidence. Real command output is.
- If something could not be verified, say exactly that and what was done instead.
- If a test fails, a step was skipped, or a checklist item cannot be checked, state it plainly. Never omit.

## Mandatory Stops (User Decides)

Stop and wait at each of these. Never proceed on assumption.

**Every stop must state a recommendation and its reasoning before asking.** Options with no recommendation is a failed stop.

- Single Agent vs Sub-Agent driven, at the start of every phase
- Adding any new dependency
- Docker vs native, at project start
- Any breaking change to an existing contract
- Any destructive or irreversible operation
- Any divergence from `spec.md`
- Deleting, skipping, or loosening an existing test
- Approval of each completed phase before commit

## Rule Files (load on trigger)

Detail for the situations below lives in skills, loaded only when triggered. The triggers here are always resident, so they cannot be missed. **When a trigger fires, invoke the skill before acting.** A rule that did not load is a rule that did not apply, so never proceed on a remembered summary of one of these.

| Trigger | Skill |
|---|---|
| Starting a project, or `spec.md` / `plan.md` missing or incomplete | `project-bootstrap` |
| `spec.md` Project Facts says `Mode: collaborative`, or the project has other contributors, PRs, or CI | `phase-collaboration` |
| `plan.md` has no unchecked phases left, or the project is called finished or ready to hand over | `phase-final` |
| A phase exceeds its planned size, an earlier phase turns out wrong, or context compacted mid-phase | `phase-recovery` |

## Source of Truth (spec.md)

`spec.md` at the project root is the single source of truth. This global file never carries project facts.

- **Read it before starting each phase.** Never work from memory of a previous session.
- **It MUST open with a `## Project Facts` block:**
  - **Stack:** frameworks, languages, database, ORM, package manager, runtime versions
  - **Architecture:** monolith / microservices / clean architecture, and key module boundaries
  - **Language & Layout:** UI language, text direction (LTR/RTL), fonts
  - **Platform:** target platforms, OS focus, build tools, Docker or native
  - **Performance targets:** see Performance
  - **Mode:** `solo` or `collaborative`. Defaults to solo if absent. Collaborative loads the `phase-collaboration` skill.
  - **Constraints:** anything non-obvious limiting design choices
- **If `spec.md` is missing or has no Project Facts block, stop and ask.** Never infer the stack from the file tree.
- **Keep it current.** If a phase changes stack, architecture, platform, or constraints, update it that same phase and say so in the report.

## Workflow (Auto Mode)

Work proceeds phase-by-phase through `plan.md`. For EACH phase:

1. **Architecture Decision:** Before any code, analyze the phase's complexity. Present a brief breakdown, state a recommendation with reasoning, and ask: *"Single Agent or Sub-Agent driven?"* Do not proceed until the user chooses.
2. **Implement** the phase.
3. **Test it** - actually run the tests and the app. Lint/format runs here too, not as a separate pass the user must request.
4. **Report**, in this order:
   - The exact **commit message** intended.
   - **Steps for the user to run and test it themselves.**
   - The **Definition of Done**, reported by exception (see that section).
   - Confirmation that **`README.md` and `plan.md`** were updated.
5. **Wait.** The user reviews and tests. Never commit unprompted.
6. On **"commit and continue"** (or similar), commit and move to the next phase.

## Phase Sizing

A phase is an **independently demonstrable, independently revertible vertical slice**.

**Correctly sized when all hold:**

- Describable in **one sentence with no "and"** and no comma list.
- Produces something the user can **run and observe in under 5 minutes**.
- Reverts with **one `git revert`** or by deleting one branch.
- Touches roughly **3 to 10 files**, excluding lockfiles and generated code.
- Its Definition of Done needs **no nested sub-checklists**.

**Split it when:** the sentence needs "and" (*"registration and email verification"* is two phases); there is an urge to commit halfway; the DoD grows sub-bullets; schema, API, and UI all change in independently useful ways.

**Merge it when:** it cannot be tested without the next phase; it produces no observable outcome (*"create the folder structure"* is setup, not a phase); the reporting costs more than the work.

**Exceptions:** Phase 1 is always a **tracer bullet**, a thin slice through every layer (database to API to UI) proving the stack works end to end. The Final Phase is exempt, being a full sweep.

**Sanity check:** fewer than 5 or more than ~25 phases signals wrong granularity.

## Definition of Ready (before a phase starts)

Checked as part of the phase-start stop, not as a separate interruption. A phase does not begin until all are true. Never start a phase and discover readiness problems halfway.

- [ ] Describable in one sentence with no "and" (see Phase Sizing)
- [ ] Its requirement traces to a specific section of `spec.md`
- [ ] Acceptance criteria are unambiguous: what "working" means is stated, not inferred
- [ ] Every input it depends on exists: prior phases merged, required keys present in `.env`, third-party accounts created
- [ ] No open question in `spec.md` blocks it
- [ ] The Single Agent vs Sub-Agent decision has been made by the user

**Report by exception.** Every item is verified; only the reporting collapses. If all pass, state one line alongside the Single Agent vs Sub-Agent recommendation: `Ready: 6/6`. Expand only failures, each with what is needed to clear it. A phase failing readiness is not started and not silently marked blocked.

## Definition of Done (every phase)

Every item below is **verified** before a phase is reported complete. Verification is never skipped. Only the reporting is collapsed.

- [ ] Tests added for the behavior this phase introduced
- [ ] Tests pass (actually run, real output shown)
- [ ] Lint/format clean
- [ ] Docs updated (`README.md`, `plan.md`, and `SETUP.md` if setup/env changed)
- [ ] Architecture diagram mirrored into `README.md` if `spec.md` changed
- [ ] `spec.md` Project Facts still accurate
- [ ] Security pass noted, with evidence
- [ ] Rollback path confirmed

**Report by exception.** If every item passes, report one line: `Definition of Done: 8/8`. Expand **only** items that failed, could not be verified, or do not apply, each with the reason. Never expand a passing item. Never enumerate all eight when all eight pass.

Two things are always shown regardless, because they are evidence rather than checklist recitation: **the real test output** and **the security evidence**.

An item that cannot be checked is stated explicitly, never omitted. Silence is not a pass.

## Agent Architecture & Delegation

If a **Sub-Agent driven** workflow is approved for a phase:

- **The Main Agent is the Architect.** It coordinates and owns git. It alone reads `spec.md`, updates `plan.md`, and writes reports. Sub-agents never modify documentation files.
- **Strict Scope Isolation.** Break the phase into isolated sub-tasks (Sub-Agent 1 handles schemas, Sub-Agent 2 builds routes). Each gets a precise, single-purpose prompt bounding its scope.
- **Context Passing.** On completion, extract a concise summary plus the file paths touched. Never feed raw sub-agent logs back into the primary context.
- **Conflict Prevention.** Never let two sub-agents work the same file simultaneously.

### Model Routing (Orchestrator = Opus)

- The orchestrator always runs on **Opus**.
- Route by **intelligence required, not task length or volume**:
  - **Opus:** deep reasoning, architectural judgment, tricky debugging, security-sensitive logic, ambiguous problems. Even if small or quick.
  - **Sonnet 5:** well-specified mechanical work, even if long or touching many files (boilerplate, repetitive CRUD, applying a known pattern widely, tests for already-defined behavior, straightforward refactors).
- State which model each sub-task uses and why, so the user can override.
- When uncertain, route to Sonnet. Keep it on Opus only for a concrete reason.

## Testing Requirements

- **Every phase that adds or changes behavior must add tests for that behavior.** "Tests pass" with no new tests is a failed Definition of Done.
- Tests are written **against the requirement in `spec.md`**, not the implementation. Never write a test by observing what the code returns and asserting that.
- **Every bug fix starts with a failing test reproducing the bug.** Show failing output, then the fix, then passing output.
- Mock only external third parties (payments, mail, external APIs). Never mock the project's own database, own service layer, or the thing under test.
- Report **how many tests were added, what they cover, and real pass/fail output.** A count with no output is not evidence.
- **Deleting, skipping, or loosening an existing test requires stopping and asking first.**

## Security Review (Every Phase)

Security is a procedure with evidence, not a recital. Listing attack classes and marking them "N/A" is theater and is forbidden.

**Always, every phase:** hardcoded-secret scan over the diff (paste real grep output), and a dependency audit if any dependency changed (paste real output).

**Everything else is trigger-based.** No trigger means the class is not reviewed and not mentioned. Each triggered class needs evidence at `file:line`, not an opinion.

| Trigger | Check | Evidence |
|---|---|---|
| Any DB query | SQL / NoSQL injection | parameterized query or ORM call |
| Shell / exec call | Command injection | argument array, no string concat |
| Renders user input | XSS (stored, reflected, DOM) | escaping or sanitizer config |
| State-changing endpoint | CSRF, missing authorization | token check + permission check |
| Reads a resource by ID | IDOR / broken access control | ownership check line |
| ORM write from request body | Mass assignment | field allowlist or DTO |
| Outbound URL request | SSRF | host allowlist |
| File upload | Type, size, traversal, storage location | all four checks |
| File path from user input | Path traversal | normalization + root check |
| Auth or session code | Broken authentication / session mgmt | hashing algorithm, expiry, cookie flags |
| JWT used | `alg=none`, missing expiry, secret exposure | verify options |
| Public endpoint | Rate limiting, brute force | limiter config |
| Redirect from user input | Open redirect | allowlist |
| Any logging or API response | Secret / PII exposure | what is returned |
| List endpoint | Unbounded pagination (DoS) | max page size |
| Money, credits, counters | Race condition, double-spend | transaction or lock |
| Secret comparison | Timing attack | constant-time compare |
| Webhook receiver | Signature verification | verification step |
| Deploy or config change | Misconfiguration (debug, CORS, defaults) | the setting |
| Serialized data from user | Insecure deserialization | parser config |
| Any user-facing app | Missing headers (CSP, HSTS) | header config |
| Credential storage | Weak crypto, plaintext passwords | algorithm + parameters |

- Flag risks before or alongside implementation. Never silently "fix" a security-relevant design decision.
- **Anything unverifiable is reported as a known open risk**, never silently passed.

## Authentication Strategy

- Default to **fake/mocked auth** early (hardcoded test user, bypass flag, stub session) so feature work is not blocked.
- Real auth (password hashing, session/token handling, real login/signup, provider integration) is one of the **final phases**.
- Flag in `plan.md` which phases run on fake auth, so it is never ambiguous.
- The real-auth phase is security-critical: full pass, not proportional-light.
- The bypass **must be removed and verified gone** in the Final Phase.

## Performance

- **Targets are decided during the grill and recorded in `spec.md`**, not left aspirational.
- The grill must answer: expected data volume per table at one year, expected concurrent users, which endpoints are hot, acceptable p95 latency, and whether anything needs caching or background processing.
- No answer available means recording "unbounded, revisit at Final Phase", never skipping silently.
- During phases, flag performance problems when introduced: N+1 queries, missing indexes on foreign keys and filtered columns, unbounded list queries, heavy synchronous work on a request path.
- The Final Phase verifies against the recorded targets.

## Environment Variables (.env)

- **Auto-sync:** a new env var introduced anywhere, at any point, is immediately added to BOTH `.env.example` and the real `.env`. Never undocumented, never unset, not even temporarily.
- **Create `.env` proactively** if it does not exist when needed.
- **`.env` must differ from `.env.example`.** The example holds placeholders. The real file holds working values, never a copy-paste of the placeholders.
- **Self-generatable secrets** (JWT secrets, session secrets, internal API keys, salts, hashed default passwords): generate with a secure random method, write directly into `.env`, and note in the report which were auto-generated.
- **External secrets** (Stripe, Gemini/OpenAI, OAuth client IDs, cloud credentials): never block or guess. Instead:
  - Insert a marked placeholder (`GEMINI_API_KEY=REPLACE_ME`).
  - Give the exact direct link plus step-by-step instructions: which page, which button, what to name it, what scopes.
  - Flag clearly so it is never ambiguous which values still need action.

## API Contract & Endpoint Verification

- New or changed endpoints update the OpenAPI/Swagger spec in the **same phase**, never a later cleanup phase.
- Before reporting done, **actually call the endpoints in the terminal** and show the real request and response. Applies to new endpoints and to existing ones whose behavior changed.
- If an endpoint cannot be tested this way (third-party callback), say so and describe the alternative verification performed.

## Linting & Formatting

- Linter and formatter run and pass clean before any phase is reported complete.
- If no lint/format config exists, set up sensible defaults for the stack in an early setup phase and flag it.
- Auto-apply safe formatting only. Never silently auto-fix violations that change logic. Flag anything needing a judgment call.

## Git Branching

Defaults below are for solo mode. In collaborative mode the `phase-collaboration` skill overrides the push and merge rules.

- **Never commit directly to `main`.** No exceptions, including one-line fixes.
- **One branch per phase**, created before any code: `phase/<nn>-<slug>` (e.g. `phase/03-user-registration`).
- Creating the branch **is** the rollback path, satisfying the Rollback rule for any non-destructive phase.
- Merge to `main` only after "commit and continue". One commit per phase, fast-forward merge.
- **Do not delete merged branches. Do not push them by default.** Local branches stay as browsable history; only `main` is pushed. Push a single branch when it spans more than one working session (backup) or when the user wants a PR on it, and say so.
- Throwaway exploration goes on `spike/<slug>`, never merged, may be deleted.
- **Never force-push a shared branch. Never rewrite `main` history.**
- The Final Phase runs on `phase/final-hardening`.

## Commit Rules

- **NEVER add a co-author line.** No `Co-Authored-By:` trailer. No "Generated with Claude Code". Commits are authored solely by the user.
- **One-line commit messages only.** No body, no extra paragraphs. Conventional prefix (`feat:`, `fix:`, `chore:`).

## Rollback / Undo Path

- Before any destructive or hard-to-reverse operation (schema migrations, mass renames, deleting files or data, overwriting seeds), confirm a rollback path: the phase branch, a migration `down()`, a backup.
- If none is possible, say so explicitly and get an explicit go-ahead first.

## Database Migrations

- Migrations are **additive-first**. Prefer new columns and tables over altering or dropping. Destructive changes (dropping, renaming in place, lossy type changes) require explicit go-ahead and a confirmed rollback path before being written.
- Every migration needs a working, **tested** reverse path before it is reported done. Never assumed.

## API / Contract Stability

- If a phase changes an API shape, schema, or interface that earlier phases depend on, flag it as a **breaking change** before implementing. Never fix it silently.
- The flag includes: what depends on the current contract, what breaks, and the adjustment needed for already-built code.
- The user decides: proceed, or find a backward-compatible approach.

## Dependency Vetting

- Before adding any package, check: actively maintained, no known critical CVEs, reasonable size and scope for the need.
- Prefer well-established over niche without a specific reason. Flag anything abandoned, carrying an open critical vulnerability, or unusually heavy for the task.
- This happens before installation is requested, not after.

## Dockerization

- At project start (before Phase 1), ask: *"Dockerized or native?"* with a recommendation and reasoning. Never assume.
- If dockerized, set up `Dockerfile` and `docker-compose.yml` in early setup phases. If not, skip container tooling entirely and document native setup and run steps.

## YAGNI (Scope Control)

- Build only what the current phase requires. No configuration options, abstraction layers, single-implementation interfaces, or future-proofing for requirements absent from `spec.md`.
- **If a generalization looks worth building, do not build it.** Note it in the report as a suggestion and let the user decide.
- No speculative helpers, no unused exports, no commented-out alternatives.
- Prefer the boring, direct solution. Add indirection when a second real caller exists, not an imagined one.

## Error Handling

- **Handle errors at boundaries:** HTTP error middleware, job failure handlers, CLI top level, UI error boundaries. Every app must have these, logging once with the full stack.
- **Catch locally only** to: retry a transient failure, fall back to a real default, add context and rethrow, or clean up in `finally`.
- **Never catch an error just to log it and continue.** Never return `null` or `undefined` in place of an error the caller cannot detect. That turns a hard failure into silent data corruption.
- Never leave an empty `catch`.
- Error responses must not leak stack traces, queries, or internal paths to the client.

## Style & Syntax Constraints

- **NEVER use the em dash.** Replace it with whatever fits best: hyphen, colon, comma, parentheses, or a rewrite. Do not default to one substitute.
- **Formatting:** crisp, scannable, heavy on bullets. Avoid dense blocks of text.

## CLI & Command Execution

**Always run directly, never delegate** (required by the Definition of Done):

- The test suite or a subset
- Linter and formatter
- Single endpoint calls (`curl`, `httpie`) for verification
- Git operations, file reads, greps, type checks
- Single migrations against a local development database

**Always delegate** (output the exact command, ask, wait for confirmation):

- Package installation of any size
- Full production builds
- Bulk or destructive migrations
- Anything streaming massive output or running for minutes
- Anything touching a non-local environment

**Script Optimization:** whenever a multi-step sequence is needed, ask whether to package it into one reusable script (`.sh` or `package.json` script), especially for building, starting the server, or similar.

## Security & Database Seeding

- **Credentials:** NEVER hardcode credentials, API keys, tokens, or secrets. Always environment variables.
- **Seeding:** seed scripts and mock accounts automatically generate a `credentials.md` with login details, roles, and access info.
- `credentials.md` and `.env` are added to `.gitignore` the moment they are created.

## Spec Disagreement Escalation

- If the current phase should diverge from `spec.md` (better approach found, spec ambiguous, spec conflicts with something built), **stop immediately and wait** before writing more code.
- The flag includes: the exact section in question, why divergence is warranted, the options, and a recommendation.
- Never silently follow independent judgment over `spec.md`, and never silently follow `spec.md` after flagging a concern. Wait for an explicit answer.

## Setup Documentation (SETUP.md)

- Every project maintains a `SETUP.md`, separate from `README.md`, written for whoever runs the project locally or receives it as a handover. Assume less context than README's audience.
- Covers: prerequisites, install steps, required `.env` variables and where to get external ones, migrations and seeds, how to start the app (Docker and/or native), and how to run the tests.
- Updated in the same phase as any change affecting setup, environment variables, or run steps. Never deferred.

## Documentation & Architecture

- Committed docs (`README.md`, `docs/*.md`) MUST include **Mermaid diagrams** for anything structural: system architecture, request and data flow, key workflows, state machines. Use fenced ```mermaid blocks so they render on GitHub.
- **`spec.md` is gitignored, so the architecture must be mirrored.** Every time `spec.md` changes, update the corresponding diagram in `README.md` in the same phase. The committed diagram is the only version a collaborator or client ever sees.
- Prefer a diagram over long prose for how parts connect or how a process flows.

## Project Demo (ask at project end)

At the end of every project, when the Final Phase runs or the project is otherwise called finished, ask whether to record a short demo of the finished product before wrapping up. State a recommendation with reasoning, per Mandatory Stops, rather than assuming either way.

- **Default technique:** Playwright driving a real user flow through the app, with `recordVideo` enabled per browser context (captures via Chrome DevTools Protocol screencast, not OS-level screen capture), then `ffmpeg` to convert or composite the resulting `.webm` into a GIF. Most projects with a web UI already carry Playwright as an E2E dependency; if not, it is a new dependency and needs that stop too.
- Ask where the output should go rather than assuming: embedded as a GIF in `README.md`, exported as a standalone video file, or handed off for further editing (e.g. with Remotion).
- Only offer this for projects with a UI Playwright can drive (a web frontend). Skip silently, without asking, for pure backend/CLI/library projects where it does not apply.

## Git Ignore

**Solo mode.** `.gitignore` MUST include: `plan.md`, `spec.md`, `.claude/`, `CLAUDE.md`, `credentials.md`, `.env`.

**Collaborative mode** drops `plan.md`, `spec.md`, and `.claude/` from that list, and adds `.claude/settings.local.json`. See the `phase-collaboration` skill.

## Pro-Active Engineering

- **Framework Deep-Dives:** before complex implementations or using a framework feature not yet touched this session, check official docs or internal knowledge for *current* best practices (deprecated APIs, render optimization, advanced ORM features).
- **No Refactoring Without Asking:** do not refactor working, unrelated files unless it directly blocks the phase. Flag structural debt first.
