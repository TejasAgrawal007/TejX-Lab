# Project Olympus — reference (challenge authoring)

Companion to `SKILL.md`. Read sections as needed for eligibility checks, feasibility writing, deliverable construction, and review.

## Workspace paths (authoring)

- **`repo/`**: local clone; apply patches and run tests here when validating.
- **`deliverables/`**: ship only `metadata.md`, `Description.md`, `Dockerfile`, `test.patch`, `solution.patch`.
- Workspace root: `handoff.md` (hard deliverable — updated after every turn; not part of the five-file submission).

## Baseline vs new tests

`test.sh` is invoked as **`test.sh --output_path <path> base|new`** and writes one JUnit XML file to `<path>` (see §JUnit XML):

- **`test.sh --output_path <path> base`**: repo at **pinned commit**, **`test.patch` applied**, **`solution.patch` not applied**. Runs the repo's **real existing tests for the area you touched** (the change's blast radius — a genuine regression check, not a token smoke test); must **pass**; writes JUnit XML. **No fail-fast flags** — every test result is needed, not just the first failure. Existing tests that are flaky, require network, or fail for pre-existing issues may be excluded; do **not** exclude valid tests just because the solution breaks them.
- **`test.sh --output_path <path> new`**: run **only** tests added for this challenge; must **fail** without `solution.patch` and **pass** with it.

## Patch application order

Clone at pinned commit → apply **`test.patch`** → `test.sh base` (pass) and `test.sh new` (fail) → apply **`solution.patch`** → both modes pass. Dockerfile build must **not** depend on patches.

## Required deliverable files (summary)

All submission artifacts live under **`deliverables/`**:

### `deliverables/metadata.md`

All rubric-required metadata fields, kept out of `Description.md` entirely:

| Field | Requirement |
| --- | --- |
| **Title** | Clear, concise |
| **GitHub repository link** | Full public URL |
| **Commit hash** | Immutable; same as Dockerfile clone |
| **Issue category** | Exactly one of: **feature request**, **bugfix**, **enhancement** |

### `deliverables/Description.md`

| Field | Requirement |
| --- | --- |
| **Problem description** | **Observable** correctness; **what** not **how**; no source paths, function names, or algorithms. Headingless prose, **≤ 600 words** (700 hard cap); no motivation/backstory; no Constraints or Out-of-Scope sections — see §Problem description format |

Description.md contains **only** this prose body — no title, repo link, commit hash, or category fields. Those live in `metadata.md`.

### `deliverables/Dockerfile`

- Base: the per-language Olympus image matching the repo's language (pick one):
  | Language | Base image |
  | --- | --- |
  | Python | `public.ecr.aws/d3j8x8q7/olympus-base-python:latest` |
  | TypeScript / JavaScript | `public.ecr.aws/d3j8x8q7/olympus-base-typescript:latest` |
  | Go | `public.ecr.aws/d3j8x8q7/olympus-base-go:latest` |
  | Rust | `public.ecr.aws/d3j8x8q7/olympus-base-rust:latest` |
  | Java (JVM) | `public.ecr.aws/d3j8x8q7/olympus-base-jvm:latest` |
  | C++ | `public.ecr.aws/d3j8x8q7/olympus-base-cpp:latest` |
- `WORKDIR /app`, then `COPY . .` to bring in the local clone (already checked out at the pinned commit) — **never** `git clone` / `git checkout` in the Dockerfile
- Dependencies installed in `RUN` layers at **build** time; the container must then run fully **offline** (`docker run --network none`)
- Ends with **`CMD ["/bin/bash"]`** — never `ENTRYPOINT`; no test invocation in any `RUN` layer
- Image builds **without** `test.patch` / `solution.patch`
- Full rule set, rationale, and self-review checklist: see SKILL.md "Hard requirement: Dockerfile construction rules"

### `deliverables/test.patch`

Valid unified diff: **tests + harness only**; includes executable `test.sh` (`100755`) invoked as `test.sh --output_path <path> base|new` (see §Baseline vs new tests); **JUnit XML mandatory**.

### `deliverables/solution.patch`

Implementation only; no Dockerfile; no conflict with `test.patch`; meets structural minimums below.

## Repo requirements (hard gates)

| Requirement | Details |
| --- | --- |
| Public | Public GitHub |
| Commit | Immutable hash (not branch/tag name) |
| Activity | ≥1 commit in last 12 months |
| Stars | 500+ |
| Quality | Production-level |
| Language | TypeScript, JavaScript, Python, Go, Rust, C++, or Java (JVM) |
| License | Allowed list only |

### Allowed licenses

MIT, BSD (all variants), Apache-2.0 (and variants including Apache-with-LLVM-Exception, Apache-with-Runtime-Exception), BSL-1.0, CC-BY (1.0 through 4.0), BLAS, GNU-All-permissive-Copying-License.

### Disqualifiers

- Inactive/abandoned
- **Any existing PR — open, merged, _or closed_ — already implements the idea.** A closed/unmerged PR that did the work still rules it out; not landing does not make it fresh. This is the #1 rejection reason.
- **Maintainers already declined it.** Check the issue tracker **and GitHub Discussions**, not just PRs — design rulings often live in Discussions. A declined feature counts as repo misalignment.
- Nonsensical or off-mission feature (read the README for context before committing to an idea)
- BSL-1.1 or non-permissive license (e.g. SurrealDB BSL-1.1 not allowed)

## Structural solution minimums

> **Authoritative bar = the in-form "Submission criteria" panel.** The platform's exact thresholds (LOC, files touched, difficulty bar) vary by submission type and **change with the product**. Always read the "Submission criteria" panel shown inside the submission form for the assignment you're on; treat the numbers below as the working defaults to design against, not a fixed contract.

- ≥ **600** lines real, purposeful code (no filler). Only **effective solution lines** count — the ones implementing the task; comments, blank lines, generated files, padding/reordering, and test code do **not** count. If agents solve it in noticeably fewer lines than your solution, your real count is probably lower than it looks.
- ≥ **5** files meaningfully modified
- ≥ **120** agent messages (rough estimate for full solve path)
- Olympus-quality challenges normally engage **≥3** distinct **fair difficulty amplifiers** (see below)

## Problem description rules

- Self-contained; unambiguous; behavior-focused; no solution leakage; no file-pointing
- Hard by **substance** (interacting requirements, edge cases, compatibility), not obscurity
- **Headingless prose, ≤ 600 words** (700 hard cap); no motivation/backstory; just what the feature/enhancement does and how it observably behaves; no separate Constraints or Out-of-Scope sections — see §Problem description format

## Test requirements

- During challenge construction or revision, author tests yourself directly — see SKILL.md §Test authoring. Do not delegate to the `behavioral-test-writer` subagent or any other test-writing subagent; the bundled copy at `subagents/behavioral-test-writer/SKILL.md` is retained only as reference documentation of the methodology, not something to spawn. Work from the repo path, pinned commit, the approved `Description.md` behavioral spec, packaging constraints, JUnit requirement, and runtime constraints. By the time tests are written the `solution.patch` already exists (it is drafted first to confirm LOC), but it is used only for post-hoc verification *after* test design and authoring are complete — derive every test from the behavioral spec, **never** from the solution.
- Write the plan as individual tests, not just a bulk suite: each test needs a standalone `TEST-SPEC` block with requirement IDs, target file, test name, public entry point, setup, action, assertions, negative assertions/non-goals, cleanup, expected baseline failure, expected solved result, and narrow verification command. Use these `TEST-SPEC` blocks to write tests one at a time.
- Deterministic; no timing/randomness/races/network in tests (unless harness explicitly allows)
- **Don't over-pin the output.** Do not assert on exact error text, messages, wording, or formatting unless the description states it or it's obvious from the repo's existing patterns. Otherwise an agent that gets the behavior right but words it differently fails unfairly — asserting the behavior holds is enough.
- JUnit XML to exact `--output_path`
- Assert **behavior** / public APIs, not internals
- Distinctive test names (avoid TEST_MISMATCH)
- Success, failure, and edge cases; negative tests where appropriate
- **Requirement↔test coverage is complete and exact (no false positives)** — build the bidirectional traceability matrix in `handoff.md`: every requirement in `Description.md` (decomposed into REQ-IDs, multi-part sentences split) has ≥1 red-on-baseline covering test (forward: no test gap), and every test traces to a quotable requirement (reverse: no over-reach / no description gap). The stated-requirement set and tested-behavior set must be **identical**, not a superset or subset. Run the adversarial self-check (no wrong/partial implementation passes the suite for any requirement) and triage the platform's **Test Fairness** suggestions icon *before* agents run. See SKILL.md "Hard requirement: requirement↔test coverage". This is the top cause of a failed Test Fairness check.

## Solution requirements

- No new runtime deps needing network
- No Dockerfile in solution patch
- No regressions; idiomatic; no AI slop

## Difficulty Amplifiers

1. **Cross-Cutting Features** — many subsystems (CLI, core, formatting, errors, config).
2. **New Abstraction Integration** — new abstraction adopted by existing consumers.
3. **Protocol Extension** — parser, validator, executor, serializer, client stay aligned.
4. **Configuration Propagation** — option through many layers (CLI → runtime effect).
5. **Backward-Compatible Migration** — internal change with old compatibility.
6. **Error Handling Completeness** — failure modes through call chains.
7. **Deep Type-Level Programming** — generics, traits, inference pressure.
8. **Concurrent/Async Correctness** — ordering, cancellation, cleanup.
9. **Graph-Structured Dependencies** — cycles, topo order, incremental recompute.
10. **Mirror Feature with Non-Trivial Adaptation** — similar feature, not copy-paste.
11. **Multi-Dialect / Multi-Backend** — different backends, different edge cases.
12. **Observability / Instrumentation** — logs/metrics/tracing, error paths included.
13. **Invariant Enforcement Across Boundaries** — invariant across modules/handlers.

## Unfair anti-patterns (never)

Hidden requirements; tests on internals; vague edge cases; external-only knowledge; brittle exact error strings; over-prescriptive spec; pointing to exact files/functions; filler complexity; tests mandating one execution order; spec vs repo conventions conflict; leaking investigation path in prompt.

## Evaluating Difficulty

| Level | Notes |
| --- | --- |
| **Low** | Single module, shallow, under 300 LOC real change — **not suitable** |
| **Medium** | Pattern-following multi-file — **usually not suitable** unless strong amplifiers + scope |
| **Hard** | Cross-cutting, convention inference, **≥3 amplifiers** — acceptable **floor** |
| **Very Hard** | Type-level, protocol layers, async, multi-backend — **preferred** |
| **Extreme** | **4+ amplifiers** or rare depth — still fair and fully specified |

Target: **Hard** minimum; prefer **Very Hard**–**Extreme**.

## Deep research checklist

Complete this before writing the feasibility report. Skipping items here directly reduces problem quality.

**Codebase structure**
- [ ] Read the full top-level directory layout; identify every major subsystem
- [ ] Read `CONTRIBUTING.md`, `CHANGELOG.md`, architecture or design docs if present
- [ ] Identify the primary public API surfaces and how they are consumed internally
- [ ] Understand the module/package boundaries and how subsystems communicate

**Test suite**
- [ ] Read the full existing test suite structure (framework, folder layout, naming conventions)
- [ ] Note any fixture or helper patterns the tests rely on
- [ ] Identify what the tests assert on — public API, output format, error messages

**Recent activity**
- [ ] Scan the last 20–30 merged PRs: what patterns do maintainers accept? What do they push back on?
- [ ] Check for any open or merged PR already addressing the same issue (hard disqualifier)
- [ ] Read all comments on the issue thread — not just the body

**Issue-specific code paths**
- [ ] Trace every code path the issue touches; read those files in full
- [ ] Identify which subsystems are affected directly and which are affected transitively
- [ ] Note any existing error handling, configuration, or type patterns the solution must follow
- [ ] Build the per-file implementation sketch required by §LOC estimation protocol — name each file to change, what changes, and the evidence-anchored LOC estimate

## LOC estimation protocol (hard gate before any PROCEED)

The feasibility report's LOC figure is a **commitment, not a guess**. A challenge that passes feasibility and then comes in under 600 LOC at construction time wastes the entire authoring cycle — that failure mode is exactly what this protocol exists to prevent. No recommendation of PROCEED (or PROCEED WITH MODIFICATIONS) is valid unless every step below is completed and recorded in the feasibility report.

### What counts toward the 600-LOC minimum

Count only lines that will actually appear in `solution.patch`:

- **Counted**: added or modified lines of real implementation code — source files, type definitions, config plumbing the solution genuinely requires.
- **Not counted**: test files (those belong to `test.patch`), generated code, lockfiles, vendored dependencies, comment-only or blank filler lines, formatting-only churn, license headers, import-reordering noise.
- **Measurement**: once a solution draft exists, measure with `git diff --numstat <pinned-commit>` in `repo/`, excluding test paths — never eyeball it.

### Evidence requirements for the estimate

A LOC estimate is valid only when backed by a **per-file implementation sketch** produced from actually reading the traced code paths (not from the issue text alone):

| File (existing path, or named new file) | Change description | Est. LOC | Evidence anchor |
| --- | --- | --- | --- |

Every row's *evidence anchor* must point at something concrete and measured in the repo: the line count of an analogous existing function or feature, the diff size of a similar merged PR, or the size of the interface/trait/protocol surface being implemented. A bare "~600 LOC" with no per-file table is an **invalid estimate** and the feasibility report must be redone.

### Calibration against analogous merged PRs

Before finalizing the estimate, find **1–3 merged PRs in the same repo** with comparable scope and measure their real diff sizes (GitHub PR diff stats, or locally `git diff --shortstat <merge>^1 <merge>`). Record each PR and its LOC in the report.

- If the closest analogous PR landed well under 600 LOC, the issue as-stated almost certainly will too. Do **not** assume your version will be bigger — fold scope expansions into the spec first, then re-estimate.
- If no analogous PR exists, say so explicitly and anchor every table row to measured in-repo code instead.

### Safety margin (mandatory)

Feasibility-time estimates are systematically optimistic: idiomatic solutions shrink as they are written. Apply a **1.3× margin** to the 600-LOC floor:

- Evidence-backed estimate **≥ 800 LOC** → the 600-LOC threshold may be marked **YES**.
- Estimate **600–799 LOC** → mark **BORDERLINE**. The recommendation must be at most **PROCEED WITH MODIFICATIONS**, with named scope expansions folded into the planned spec *before* construction starts — not deferred as "we can expand later".
- Estimate **< 600 LOC** → mark **NO**. Recommendation is **DO NOT PROCEED** unless concrete, ticket-realistic scope expansions credibly lift the evidence-backed estimate to ≥ 800.

### Reconciliation checkpoints during construction

`solution.patch` is the **first** construction deliverable — drafted before `Description.md` precisely so LOC is confirmed against real code before any description or tests are written.

1. **After the solution outline, before writing any code**: recompute the per-file table against the outline. If the margin rule no longer holds, stop and renegotiate scope with the user — do not start coding toward a known miss.
2. **After `solution.patch` is generated (before `Description.md`)**: measure actual LOC with the measurement command and record **estimated vs. actual** in `handoff.md`. Present the actual to the user and confirm it clears the **600-LOC minimum** before proceeding to `Description.md` and tests. An actual below 600 after a PROCEED is a *feasibility* defect, not a construction defect — fix scope first, then note in the feasibility report what the estimate missed so the next estimate is calibrated. Do not write the description or tests against a known shortfall.

## Scope maximization

After the research checklist is complete, actively look for the highest-difficulty version of the problem that remains fair and fully specifiable. Work through all 13 difficulty amplifiers and ask: does the issue as-stated activate this? Could a natural, ticket-realistic expansion activate it?

Target is **Very Hard or Extreme** — meeting the Hard floor is not a success condition. Scope expansions must be natural engineering extensions, not padding. Examples of natural expansions:
- "The feature also needs to propagate through the config/CLI layer" (activates Configuration Propagation)
- "Error paths through the new code must be handled consistently" (activates Error Handling Completeness)
- "The existing similar feature in the Y backend must be mirrored" (activates Mirror Feature with Non-Trivial Adaptation)
- "The change must remain backward compatible with old serialization format" (activates Backward-Compatible Migration)

Document each considered expansion and whether it was included, with reasoning, in the feasibility report.

## Feasibility report template

Working document at workspace root: `feasibility-report.md` (not a platform submission file). Use the structure below; interpret **§** markers as the sections with the same names above (Evaluating Difficulty; Difficulty Amplifiers).

```markdown
## Feasibility Report

### Issue Summary
[1-2 sentence summary of what the issue asks for]

### Repository Quick Facts
- **Name**: [owner/repo]
- **Stars**: [count]
- **License**: [license — must be on the allowed list]
- **Language**: [primary language]
- **Last Active**: [date of most recent commit]
- **Commit Hash**: [latest or pinned immutable commit hash to use]

### Estimated Difficulty
[One of: Low / Medium / Hard / Very Hard / Extreme — see §Evaluating Difficulty]
[2-4 sentences explaining why, referencing specific codebase characteristics]

### Difficulty Amplifiers Present
[List which of the 13 fair difficulty amplifiers apply — see §Difficulty Amplifiers.
For each, write one sentence explaining how it applies to THIS specific issue.
Olympus-quality challenges should normally engage **at least three** distinct amplifiers; if fewer, explain why the issue is still structurally hard (e.g. unusually deep correctness constraints).]

### Estimated Solution Scope

Per-file implementation sketch (required — see §LOC estimation protocol; a bare total with no table is invalid):

| File | Change description | Est. LOC | Evidence anchor |
| --- | --- | --- | --- |
| [path or named new file] | [what changes] | [N] | [measured analogous code / PR / interface size] |

- **Files modified**: ~[N]
- **Lines of code (net new/changed)**: ~[N] (sum of table; YES on the 600-LOC threshold requires ≥ 800 per the safety margin)
- **Analogous merged PRs measured**: [PR link or merge hash — N LOC each; or "none found, anchors are in-repo measurements"]
- **Estimated agent messages**: ~[N] (rough engineering estimate for a strong human or agent — not a submission quota)

### Meets Minimum Thresholds?
- [ ] ≥ 600 lines of real, purposeful code — [YES (estimate ≥ 800) / BORDERLINE (600–799) / NO (< 600) — per §LOC estimation protocol]
- [ ] ≥ 5 files modified — [YES / NO / BORDERLINE]
- [ ] ≥ 120 agent messages (estimated) — [YES / NO / BORDERLINE]
- [ ] ≥ 3 distinct difficulty amplifiers — [YES / NO / BORDERLINE]

### Red Flags
[List any concerns. Check ALL of the following:]
- Does an existing open or merged PR already solve this?
- Is the spec ambiguous or underspecified?
- Is this too easy (single-file fix, obvious approach)?
- Is the scope too narrow to hit **600** LOC naturally without padding?
- Does it require external knowledge not available in the repo?
- Will tests need to assert on internals rather than behavior?
- Does the problem contradict the repo's design philosophy?
[If none, write "None identified."]

### Scope Expansion Suggestions (if borderline)
[If the issue as-is is borderline on thresholds, suggest NATURAL expansions:
- "Also handle error cases for X"
- "Extend to also cover the Y variant"
- "Include Z configuration option support"
Only suggest expansions that would be natural in a real engineering ticket.]

### Recommendation
[PROCEED / PROCEED WITH MODIFICATIONS / DO NOT PROCEED]
[Explain reasoning. If modifications needed, be specific about what to change.]
```

## Hint system (optional)

Rare. Only if fair, fully specified, still structurally very hard. Hint must be inferable; steer **where to look**, not **what to type**. Never replace fixing vague spec or broken harness.

## Problem description format

`Description.md` contains **only** the problem-statement prose — no Title, repo link, commit hash, or category (those live in `deliverables/metadata.md`, see §Required deliverable files above). The body reads **the way a maintainer writes a GitHub issue** — natural prose, full sentences:

- **Open with the ask itself** ("Add X to Y", "Fix Z when …") and let the first line stand on its own without the title. Skip the motivation and the "what the repo currently lacks" preamble.
- **Length**: aim for **≤ 600 words**; **700 words is the hard limit**. Tighter is better.
- **No headings and no spec-sheet formatting.** Do not section into `Goal` / `Expected Behavior` / `Constraints` / `Out of Scope`, and keep it out of spec-sheet territory: **no bulleted requirement lists, no `##`/`###` headers, no code snippets doing the describing.** Continuous prose in short paragraphs. (The approved frostdb example is pure prose — model that shape.)
- **No motivation, backstory, or rationale.** State only what the feature/enhancement does and how it observably behaves.
- **No Constraints section and no Out-of-Scope section.** If a compatibility boundary or error behavior is genuinely needed to test the feature, fold it into the prose as part of describing what the feature does — do not break it out as a separate section.
- **Describe behavior, not implementation** — don't spell out what a developer in the repo would find on their own (internal class names, helpers, field names, file layout). **But use judgment:** if a detail is genuinely part of the contract and the task can't be pinned down without it, state it. A task nobody can implement is worse than one that names a field.
- Everything in §Problem description rules still applies: self-contained, unambiguous, behavior-focused, observable (what not how), no solution leakage.
- **ASCII only, no exceptions.** The portal hard-rejects any non-ASCII character (em dash `—`, en dash `–`, curly quotes `'`/`'`/`"`/`"`, ellipsis `…`, non-breaking space, etc.) with `Description contains non-ASCII character '<char>' at position <N>`. Write with plain hyphens `-`, straight quotes `'`/`"`, and `...` instead. Before finalizing `Description.md`, scan the body for non-ASCII characters and replace every one — do not rely on habit, since em dashes creep in by default.

The test contract is derived directly from this prose (see SKILL.md §Test authoring), so every behavior a test will assert must be stated or directly implied here. Unstated behavior is treated as out of scope by default — that is the intended effect of dropping the explicit Out-of-Scope section, not a reason to under-specify the behavior itself.

## Olympus scoring (21 checklist items)

Submissions are evaluated against 21 qualitative items. Quality score must be ≥ 5/7 for acceptance. **Structural minimums** (600+ LOC, 5+ files, test harness rules, etc.) are **additional** gates — passing the checklist does not excuse missing those minimums unless an explicit, reviewed exception applies.

### Problem (Items 1-7)

1. Requirements complete and self-contained
2. No ambiguities, fully deterministic
3. Concise and not prescriptive (describes what, not how)
4. Matches real-world repo scope
5. Aligns with repo design philosophy
6. No irrelevant context
7. Clear writing and formatting

### Tests (Items 8-15)

8. New tests highlight missing or incorrect behavior (fail without solution, pass with)
9. Tests are deterministic (stable across multiple runs)
10. Assertions verify the correct output (precise expected outcomes)
11. Tests validate behavior, not internals (public APIs and observable output)
12. Tests follow repo structure (naming, folder, framework conventions)
13. Tests cover required behavior AND edge cases
14. Test suite is concise (focused, non-redundant)
15. Tests do not check unspecified behavior

### Solution & Code (Items 16-21)

16. Solution meets all requirements
17. No regressions, code follows patterns
18. No unexplained defensive code
19. No irrelevant changes
20. API contracts remain stable
21. No AI slop

## Evaluation notes

- Do not tune difficulty to a target agent success rate.
- **False positives fail the Test Fairness check.** An agent that passes the tests without meeting the task means the tests under-cover the requirements (test gap) or the description over-states them (description gap). Close the gap with the requirement↔test traceability matrix (see §Test requirements and SKILL.md) *before* submitting — each iteration of the agent-run + false-positive panel is slow, so catch coverage holes at authoring time via the Test Fairness suggestions icon.
- Quick Check for Dockerfile + `test.sh` + JUnit before large batches.
- **Solvability is a submit gate: at least one agent must solve the challenge before it can be submitted.** If no agent ever passes, the tests are probably too strict, the description is missing something, or the task is unfair — iterate before submitting. Start with a **small batch** of agent runs to get a read (100% pass → too easy; 0% on a vague sentence or unfair test → catch it before a full run) rather than spending a full batch up front.
- **Be your own reviewer first.** Walk your agent runs the way a reviewer will. Failures should be because the task is genuinely hard — not because a sentence was ambiguous, a requirement was hidden, or a test asked for something the description never stated. If runs show agents tripping on something unintended, fix it now; after submission it comes back as a revision. Also sanity-check LOC: if agents solve in noticeably fewer lines than your solution, your effective count is likely under the bar.
- Bypass (narrow): test name collision; slightly under thresholds with strong justification — no generic padding.

## JUnit XML (`test.sh`)

Write **one** JUnit XML file to the **exact** path from `--output_path` (create parents if needed).

**pytest**

```bash
pytest tests/ -v --junitxml="$OUTPUT_PATH"
```

**vitest**

```bash
vitest run tests/ --reporter=junit --outputFile="$OUTPUT_PATH"
```

**jest** (jest-junit in image)

```bash
JEST_JUNIT_OUTPUT_DIR="$(dirname "$OUTPUT_PATH")" \
JEST_JUNIT_OUTPUT_NAME="$(basename "$OUTPUT_PATH")" \
  npx jest tests/ --reporters=jest-junit
```

**go test** (go-junit-report in image)

```bash
go test -v -count=1 ./pkg/... -timeout 10m 2>&1 \
  | go-junit-report -set-exit-code > "$OUTPUT_PATH"
```

The `-v` flag is required — `go-junit-report` parses verbose output.

**mocha** (mocha-junit-reporter in image)

```bash
mocha tests/ --timeout 10000 \
  --reporter mocha-junit-reporter \
  --reporter-options mochaFile="$OUTPUT_PATH"
```

**deno**

```bash
deno test tests/ --junit-path="$OUTPUT_PATH"
```

## Common rejection reasons (short)

Existing PR solves it; made-up feature; vague or over-prescriptive spec; internal-focused tests; hidden tested requirements; **false positives — tests under-cover the stated requirements (a requirement with no red test) or the description over-states what the tests need (a test with no requirement)**; copy-paste challenge structure; misaligned feature; ambiguous behavior; vague “same as X” without listing behavior; too shallow for 600 LOC / 5 files / real cross-cutting work.
