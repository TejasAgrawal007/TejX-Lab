---
name: project-olympus-challenge-author
description: >-
  Use when the user mentions Project Olympus, Olympus challenges, challenge authoring,
  feasibility reports, test.patch/solution.patch, or agent benchmark tasks on pinned
  commits. Project Olympus challenge authoring produces Castor-style evaluation tasks
  on real open-source GitHub repos: the workspace has a repo/ clone, a deliverables/
  folder with metadata.md, Description.md, Dockerfile, test.patch, and solution.patch,
  plus a handoff.md at the root that is updated after every turn. The
  typical flow runs from repository intake and eligibility checks through issue analysis
  and feasibility reporting, then challenge construction — drafting solution.patch first
  to confirm LOC, then the problem description, behavioral test design authored directly
  by this skill (never delegated to a test-writing subagent), and Dockerfile definition —
  before a final review against reference.md. Norms, gates, templates,
  patch order, Dockerfile and test details, difficulty rubric, and scoring live in
  reference.md.
---

# Project Olympus — challenge authoring

Human-facing overview. Platform task packages for agents are normally the problem snapshot and patches only, not this skill text.

## ⛔ Hard requirement: fully offline execution

**Every part of the challenge — tests, solution, Dockerfile, and harness — must run completely offline. This is a non-negotiable hard deliverable, not a best-effort guideline.**

Rules that apply at every stage of authoring:

1. **No network calls in tests.** Every test must be self-contained: all fixtures, payloads, and expected values must be embedded in the patch or checked into `repo/` at the pinned commit. Any test that requires a live HTTP request, DNS lookup, external API call, or remote resource is a **blocking defect** and must be rewritten before the challenge is considered complete.
2. **No network calls in the solution.** The implementation that `solution.patch` produces must not fetch data from the internet at runtime. If the real upstream feature needs a network call, the challenge must be scoped to a locally-testable sub-problem.
3. **Dockerfile runs with `--network none`.** The `Dockerfile` and its entrypoint must be verified to succeed with Docker's `--network none` flag — no `apt-get` at test time, no `pip install` at test time, no `curl`/`wget` inside `test.sh`. All dependencies are baked into the image at build time.
4. **Double-check every test for offline behavior.** After each `TEST-SPEC` block is drafted (see Test authoring below) and after any test is written or revised, the author explicitly audits each test for hidden network dependencies: mocked HTTP clients that fall through to real sockets, hardcoded public URLs, `localhost` assumptions that break in an isolated container, or fixture-generation scripts that phone home.
5. **Offline audit is a gate.** The offline compliance audit is a mandatory checkpoint that must pass before challenge construction can be declared complete. It is listed explicitly in the final review checklist (see below).

---

## ⛔ Hard requirement: evidence-based LOC feasibility (no optimistic go-aheads)

**The 600-LOC structural minimum is judged twice — estimated at feasibility, actual at construction. A feasibility report that says "go" and then produces a sub-600-LOC solution wastes the entire authoring cycle. The LOC estimate is therefore a hard gate governed by the LOC estimation protocol in reference.md, not a gut-feel number.**

Rules that apply to every feasibility verdict:

1. **No PROCEED without a per-file sketch.** The estimate must come from a per-file implementation sketch built by actually reading the code paths the issue touches in `repo/` — each file named, each change described, each LOC figure anchored to something measured in the repo (an analogous function's size, a similar merged PR's diff, an interface surface). A bare "~600 LOC" total with no table is an invalid estimate.
2. **Calibrate against analogous merged PRs.** Find 1–3 merged PRs of comparable scope in the same repo and measure their real diff sizes. If the closest analog landed under 600 LOC, the issue as-stated will too — expand scope *before* recommending PROCEED, never after construction has started.
3. **Apply the 1.3× safety margin.** Estimates shrink into idiomatic solutions. The 600-LOC threshold is marked YES only when the evidence-backed estimate is **≥ 800 LOC**; 600–799 is BORDERLINE (PROCEED WITH MODIFICATIONS at most, with scope expansions folded into the spec up front); under 600 is NO.
4. **Reconcile at construction checkpoints.** After the solution outline (before any code is written), recompute the per-file estimate — if the margin no longer holds, stop and renegotiate scope. After `solution.patch` exists, measure actual LOC with `git diff --numstat` against the pinned commit (excluding test paths) and record estimated vs. actual in `handoff.md`. An actual below 600 after a PROCEED is a feasibility defect.

The point of this gate is front-loading: spend the research tokens once, at feasibility, instead of discovering the shortfall after the description, tests, and half the solution are already written.

---

## ⛔ Hard requirement: requirement↔test coverage (no false positives)

**A challenge is only evaluable if its tests fully and exactly cover the requirements the description states. When they do not, agents get false-positive passes — they clear the tests without meeting the task — and the submission has broken verifiers. This is the single most common reason a challenge fails the platform's "Test Fairness" check, and it is a blocking defect, not a polish item.**

Think of the challenge as an *exam* you are both writing and grading. A fair exam has tough problems **and** a complete, faithful way to check that a passing student actually met every learning goal. Two failure modes produce a false positive, and both are your fault as author:

1. **Test gap** — the description states requirement C, but no test exercises C. An agent implements A and B, skips C, and still passes. *Fix:* add a test for C, or (if C was never really intended) delete C from the description.
2. **Description gap** — the tests only need A and B, but the description lists C as required. C is either untested dead weight (over-specification) or a hidden requirement the agent can't infer. *Fix:* either test C or remove it from the description.

Either way the rule is the same: **the set of stated requirements and the set of tested behaviors must be identical.** Not a superset, not a subset — identical.

### Requirement↔test traceability matrix (mandatory, both directions)

Before the red/green contract is declared verified, build and record in `handoff.md` a matrix that maps **every** normative requirement in `Description.md` to the test(s) that assert it, and **every** test back to the requirement it traces to:

| REQ-ID | Requirement (quoted from Description.md) | Covering test(s) | Baseline-failure category | Notes |
| --- | --- | --- | --- | --- |
| REQ-1 | "…exact sentence or bullet from the description…" | `slug_empty_input_returns_error` | assertion on public output | |

Two audits over this matrix, both of which must come back clean:

- **Forward (no test gap):** every REQ-ID has at least one covering test whose baseline failure is behavioral (red for the spec'd reason, not an import/fixture/harness error). A requirement with no red test is a false-positive hole — an agent can skip it and still pass. Add the test or cut the requirement.
- **Reverse (no description gap / no over-reach):** every test row maps to a REQ-ID quotable from `Description.md`. A test with no quotable requirement is either testing an internal, testing unspecified behavior, or asserting stricter/looser than the spec — it must be rewritten, deleted, or the description must be tightened to state that behavior. Never leave an assertion that can't be traced to a sentence in the prose.

Multi-part requirements decompose: "the flag must be accepted, propagate to the runtime, and error on conflict with `--foo`" is three testable behaviors (accept / propagate / conflict), each its own REQ-ID and each its own red test. One coarse test that only checks the happy path leaves the conflict behavior a false-positive hole even though the sentence is "covered".

### Adversarial self-check: try to pass without meeting the task

For each requirement, ask concretely: *"Could an agent produce output that passes every test touching this requirement while getting the requirement itself wrong?"* Walk the plausible partial/wrong implementations (happy path only, missing the boundary, ignoring the conflict, wrong error surface) and confirm at least one test would catch each. If a wrong implementation survives the suite, the coverage is incomplete — add the discriminating test. This is the same reasoning the platform's false-positive panel applies after agents run; doing it here, before the agent run, is what saves iteration cycles.

### Test Fairness panel — read the suggestions icon early

The platform's **Test Fairness** check surfaces a suggestions/info icon *before* any agent runs, flagging likely coverage gaps. Treat it as a free early review: open it, and for each suggestion decide **true gap vs. noise**. Fold every true gap into the matrix and add the missing test now — catching it here is cheap; catching it after an agent run and a false-positive panel is slow. Record in `handoff.md` which suggestions were acted on and which were dismissed as noise, with a one-line reason each.

This matrix and both audits are a gate: challenge construction is not complete until the forward and reverse audits pass and the Test Fairness suggestions have been triaged.

---

## Supporting material

[reference.md](reference.md) is the companion document: repo and license expectations, baseline vs new tests, patch order, deliverable file summaries, structural minimums, difficulty rubric and amplifiers, problem and test norms, hint rules, scoring checklist, JUnit examples, feasibility report template, and common rejection patterns.

## Test authoring — write tests yourself, never delegate

**Do not spawn or invoke the `behavioral-test-writer` subagent (or any other test-writing subagent) for any part of test strategy, `test.patch`, `test.sh`, or test review.** The subagent copy at [`subagents/behavioral-test-writer/SKILL.md`](subagents/behavioral-test-writer/SKILL.md) is retained only as reference documentation of the methodology below — it is not to be spawned. The author of this skill writes every test personally, directly in `repo/`, using the deep-research digest already gathered (test framework, locations, naming/fixture conventions, public entry points, `base`/`new` selection mechanism) rather than re-exploring from cold.

Follow this methodology when writing tests, scaled to the behavioral surface (a two-requirement bugfix does not need the analysis budget of a multi-option feature):

1. **Restate the behavioral contract.** Extract each normative requirement from `Description.md` into a short requirement ID (REQ-1, REQ-2, …), decomposing multi-part sentences so each independently-testable behavior gets its own ID (accept / propagate / conflict / error surface are separate rows, not one). For each, note what is observable through the public surface and what is mere implementation that must never be asserted. Flag any missing or ambiguous requirement as a spec gap rather than inventing behavior. This REQ-ID list is the left column of the requirement↔test traceability matrix (see the coverage hard requirement above) — every ID here must end up with a red-on-baseline covering test.

2. **Behavior vs. implementation doctrine.** Assert only on: return values and their public shape; raised error class/exit code/HTTP status/diagnostic code; serialized public output (stdout/stderr, response bodies, generated files, persisted records read back through a public API); side effects visible through public means; idempotency/ordering/round-trip guarantees the contract promises; state observed only through public accessors. Never assert on: private/internal names or module structure; the concrete class behind a public interface; call order/count/which internal helper ran; algorithm or data-structure choice; log lines or internal counters; exact error-message wording (unless the text itself is the contract); generated temp names, timestamps, unordered-collection iteration order, or incidental formatting; internal timing, unless a documented performance contract is under test. Litmus test for every assertion: *would it still pass after a full internal rewrite that honors every documented promise?* If no, delete or rewrite it.

3. **Design the baseline failure per test, before writing it.** Best to worst: (1) assertion failure on observable output through an existing public entry point — the gold standard; (2) public error-surface mismatch (unknown flag/config key, parse error) when that surface is itself the missing behavior; (3) compile/import/collection error from referencing new public API — last resort, since it collapses per-test signal and can break `base` mode in compiled/typed languages or import-sensitive Python modules. Never acceptable: failures from test bugs, missing fixtures/deps, wrong paths, harness errors, or network access. Every challenge test must be red on baseline for a behavioral reason — a test that passes pre-solution belongs in the upstream regression suite, not here.

4. **Build a test matrix before editing.** Cover the primary path; every boundary (empty, single, many, min/max, just-over-max, zero, negative, unicode, null, duplicate, domain-specific); negative behavior (invalid input, partial failure, incompatible config, conflicting options, cleanup/rollback, diagnostics) each as its own test; option/flag interactions with distinct observable behavior; idempotency, ordering, and round-trip guarantees the contract implies. One behavior per test — split bundled tests, merge tests whose oracle and caught mutant fully overlap. For each row, record the REQ-ID(s) it covers, the mutant it catches, and the baseline-failure category. Cross the matrix against the REQ-ID list from step 1: **every REQ-ID must appear in at least one row** (forward coverage) and **every row must name a REQ-ID** (reverse coverage). A REQ-ID with no row is a false-positive hole — fill it before writing any test.

5. **Write one standalone `TEST-SPEC` block per test before implementing it**: requirement IDs, target file, test name, public entry point, setup/fixtures, action, assertions (observable only), explicit non-goals (implementation details forbidden), cleanup/isolation, expected baseline result with category and reason, expected solved result, narrow verification command.

6. **Author tests one `TEST-SPEC` at a time.** Drive every test exclusively through public entry points; use temp dirs, local fixtures, fake transports, or deterministic stubs instead of real services; make every test deterministic and order-independent (no reliance on map/set iteration order, hash seeds, wall clock, locale, timezone, TTY/color detection, terminal width, CPU count, filesystem listing order). After adding each test, run the narrowest useful command to confirm it fails or passes for the expected reason before moving to the next. Prefix test names with a distinctive challenge slug (derive one from the problem title if none given) to avoid collisions with upstream/hidden tests, and keep names behavioral — never mechanistic (`empty_input_returns_error`, not `parser_handles_nil_guard`).

7. **Build or update the harness** (`test.sh --output_path <path> base|new` or the project's equivalent). `base` runs the upstream regression suite excluding challenge tests and must pass with the test patch applied and no solution. `new` runs only the challenge tests — verify the selection is non-empty. Always write JUnit XML to the exact output path, including on failing runs and harness crashes (create parent dirs first, capture exit code, confirm the XML exists, then exit with that code — watch for `set -e`/`pipefail` killing the script early). Pin determinism (timezone, locale, `NO_COLOR`/`CI`, seeds, parallelism) and keep the harness offline and idempotent.

8. **Verify the red/green contract.** From the pinned checkout plus test changes only: `base` must pass; `new` must fail with every challenge test individually red, for the spec'd category and reason (not syntax/import/fixture/harness errors). Confirm well-formed JUnit XML exists at the exact path in every mode, including failures. Only after test design and authoring are complete, apply `solution.patch` (already drafted earlier to confirm LOC — see flow below) and confirm `base` and `new` both pass; **never derive tests from `solution.patch`, only from `Description.md` and stable public conventions.** Run the new tests at least twice for flake and spot-check isolation. Audit every test for hidden network dependencies (mocked clients falling through to real sockets, hardcoded URLs, `localhost` assumptions, phone-home fixtures). Inspect the diff: tests/harness only, no implementation fix, no internals asserted, no leaky names/comments, no modified upstream tests, executable bit on `test.sh`. **Then run the requirement↔test coverage gate (see the coverage hard requirement above):** build the traceability matrix in `handoff.md`, confirm the forward audit (every REQ-ID has a red-on-baseline covering test) and the reverse audit (every test traces to a quotable requirement) both pass, run the adversarial self-check (no wrong-but-passing implementation survives the suite for any requirement), and triage the Test Fairness suggestions icon. Construction is not complete until these pass.

Fairness and leak hygiene apply throughout: every assertion must trace to a quotable `Description.md` sentence (or its direct consequence) — report unquotable assertions as a spec gap instead of asserting or silently dropping them; never assert stricter or looser than the spec states; test names, comments, and fixtures must never point at the fix's location or mechanism, and must carry no `// TODO`/`// FIXME`/`// see issue #N`/commented-out code.

---

## Challenge workspace layout

The **current project/workspace root** is the challenge workspace; paths below are relative to it.

| Location | Role |
| --- | --- |
| **`repo/`** | Local **git clone** of the target GitHub repository. Exploration, feasibility analysis, and patch authoring are done against this tree. |
| **`deliverables/`** | The five submission artifacts: `metadata.md`, `Description.md`, `Dockerfile`, `test.patch`, `solution.patch`. |
| **`handoff.md`** (workspace root) | **Hard deliverable.** Running log of what the agent has done, updated after every turn without exception. |

Early in a workspace, authors ensure `deliverables/` exists (possibly empty), create `handoff.md` with an initial entry, and clone the upstream repo into `repo/`.

**End-of-turn deliverable report (hard requirement).** At the end of every turn, state in the chat reply to the user exactly which deliverable file(s) in `deliverables/` (`metadata.md`, `Description.md`, `Dockerfile`, `test.patch`, `solution.patch`) were created or modified that turn — by name, not a vague "made progress" summary. If no deliverable file changed (e.g. a research-only or discussion-only turn), say so explicitly rather than omitting the line. This is in addition to, not instead of, the `handoff.md` running-log update.

---

## Typical flow

**Initialization** — The user provides a **GitHub issue link** for the task. This issue link is the single entry point: the author derives the target repository directly from it (the repository that owns the issue) rather than asking for a separate repository URL. Requiring the issue link is a deliberate **efficiency gate** — anchoring the task to a real, already-scoped issue avoids inventing tasks from scratch and cuts authoring iterations — so treat it as required even though the platform submission form lists the issue URL as optional.

**Repository intake (clone first)** — Before any issue analysis, the author clones the repository attached to the provided issue link into `repo/` (reusing or refreshing an existing clone when it already matches the same remote), and prepares the workspace as above. Cloning the repo is the **first** action and must complete before the issue is read or analyzed. Eligibility and policy checks use the local tree plus public metadata as described in reference.md.

**Deep repository research** — Before touching the issue, the author builds a thorough structural understanding of the codebase. This is not optional and must not be skimmed: read the top-level directory layout and identify every major subsystem; read `CONTRIBUTING`, `CHANGELOG`, architectural docs, and any design docs present; read the full existing test suite to internalize the testing framework, naming conventions, folder structure, and fixture patterns; scan the last 20–30 merged PRs to understand active development style, what maintainers accept, and what they push back on; identify the primary public API surfaces and how they are consumed internally. This research phase is the foundation for everything that follows — a shallow pass produces a shallow problem.

**Issue analysis, amplifier audit, and feasibility** — Only after the repository is cloned does the author turn to the issue. Using the issue link supplied at initialization, the author reads the issue body *and every comment in the thread*, identifies the concrete behavioral change requested, and explores the specific code paths in `repo/` that would be touched. Then runs an explicit amplifier audit against all 13 difficulty amplifiers in reference.md: for each amplifier, determine whether the issue as-stated activates it, and whether a *natural, ticket-realistic* scope expansion would activate additional ones. The goal is to find the highest-difficulty version of the problem that remains fair and fully specifiable — target **Very Hard or Extreme**, not the Hard floor. The author also checks that **no existing PR (open, merged, or closed) already implements the idea** and that **maintainers have not declined it** — checking issues **and GitHub Discussions**, not just PRs. The result is a feasibility assessment using the template in reference.md, which must include the full amplifier audit, the per-file LOC sketch with evidence anchors and analogous-PR calibration required by the LOC estimation protocol (hard gate above), and, where applicable, concrete scope expansion proposals that push difficulty upward. Present the difficulty rubric output **as is** — surface the rubric scoring exactly as produced, without summarizing, reweighting, or editorializing it.

**Challenge construction** — When the user wants to move forward, the author works toward the five deliverables: implementation (`solution.patch`), rubric metadata (`metadata.md`), behavioral problem text (`Description.md`), tests and harness (`test.patch`), and `Dockerfile`. **`solution.patch` is drafted first, before `Description.md`, to confirm LOC against real code.** The order is:

1. **Solution outline, then `solution.patch` — first.** Working from the issue and the feasibility analysis, sketch the per-file outline, run the pre-coding **LOC reconciliation checkpoint** (re-verify the per-file estimate against the outline), then write the solution. Once `solution.patch` exists, measure actual LOC with `git diff --numstat` against the pinned commit (excluding test paths) and log estimated vs. actual in `handoff.md`.
2. **Confirm LOC with the user before proceeding.** Present the measured actual LOC and ask the user to confirm it clears the **600-LOC minimum** (with the safety margin) before any further work. If the actual is below 600, this is a feasibility defect — stop and renegotiate scope rather than writing the description and tests against a known shortfall. Do not proceed to `Description.md` until the LOC is confirmed.
3. **`metadata.md`** — record Title, GitHub repository link, pinned commit hash, and issue category (see reference.md §Required deliverable files). **`Description.md`** — write the observable problem statement only (headingless prose, ≤ 600 words / 700 hard cap; see reference.md §Problem description format). No title, repo link, commit hash, or category in this file.
4. **Tests, authored directly** — follow the Test authoring methodology above to produce the behavioral contract, test matrix, and per-test `TEST-SPEC` blocks yourself, then write or revise tests one test at a time from those specs, deriving them from `Description.md` — **not** from the already-drafted `solution.patch`.
5. **`Dockerfile`**, then **review** against reference.md before generating final patches from the pinned commit in `repo/`.

During construction, continuously cross-check against the repo's conventions learned in the research phase — the problem description must feel like a natural ticket for this codebase, and the solution must be idiomatic to it.

At each test-writing step, the author runs the **offline audit** (rule 4 above) before proceeding to the next test. Any network dependency found is a blocking defect that must be resolved in-place — do not defer it.

**Final offline audit checkpoint** — After all five deliverables are drafted, the author performs a consolidated offline compliance pass:
- Re-read every test in `test.patch` and confirm no live network dependency exists.
- Re-read `solution.patch` and confirm no runtime network dependency exists.
- Re-read `Dockerfile` and confirm all package installs and downloads occur at image build time, not at test runtime.
- Verify `test.sh` (or equivalent harness) contains no `curl`/`wget`/network calls.

**Requirement coverage checkpoint (hard deliverable)** — After all five deliverables are drafted, re-run the requirement↔test coverage gate as a consolidated pass and confirm it in `handoff.md`:
- The traceability matrix is present and current: every requirement in the final `Description.md` has a REQ-ID and at least one covering test that is red on baseline for a behavioral reason (forward audit).
- Every challenge test traces to a quotable requirement — no test asserts unspecified or internal behavior, and none asserts stricter or looser than the spec (reverse audit).
- The adversarial self-check passed: for each requirement, no plausible wrong/partial implementation passes the suite.
- Test Fairness suggestions were triaged, with each true gap fixed and each dismissed item noted with a reason.

If the description changed after tests were written (or vice versa), the two sets have drifted — reconcile them so stated requirements and tested behaviors are identical before finalizing.

**AI hint-removal sanity check (hard deliverable)** — Before presenting any deliverable to the user, the author performs a systematic sweep to remove anything that would make it trivially easy for an AI agent to solve the challenge. This is a hard gate, not a style suggestion. The sweep covers every file in `deliverables/` and the patches:

- **`Description.md`** — Remove or rephrase any sentence that names the exact function, class, file, or line that needs to change. The description must state the *observable behavior* the agent must produce, not the implementation path to get there. Remove any inline hints like "hint:", "tip:", "note:", or parenthetical asides that point the agent at a solution.
- **`metadata.md`** — Confirm it holds only Title, repo link, commit hash, and category; no problem-statement prose or solution hints leaked into it.
- **`test.patch`** — Strip all comments that describe *why* a test exists or what implementation detail it is probing. Test method names should describe the expected behavior from a user perspective, not the internal mechanism (e.g. `testEdgeCaseHandledInParser` leaks the fix location; `testEmptyInputReturnsError` does not). Remove any `// TODO`, `// FIXME`, `// workaround`, or `// see issue #N` comments left over from authoring. Remove dead or commented-out test code.
- **`solution.patch`** — Remove all authoring comments such as `// this is the fix`, `// add null check here`, `// was missing this branch`. The solution code itself is never shown to the agent, but a leaked solution patch with a dense comment trail is a data-contamination risk if the repo or deliverables are ever inspected.
- **Commit messages and patch headers** — Ensure no patch `Subject:` line or diff header reads like a spoiler (e.g. "fix NPE in FooParser when input is null" — rewrite to something neutral like "patch" or omit).
- **No platform leakage — treat the patches like a real PR to the repo.** Nothing in `test.patch` or `solution.patch` may reference the benchmark: no `test.sh` comment referencing "the challenge", and the strings **"Shipd", "Olympus", "mars", "quest"** must not appear anywhere in the patches. If it wouldn't show up in a normal PR to this repo, it doesn't belong here. (A distinctive per-test slug for collision-avoidance is still fine — derive it from the problem's domain, not from the platform.)
- **`handoff.md`** — Strip any working notes that contain explicit solution outlines or step-by-step fix walkthroughs; keep only authoring progress entries.

After the sweep, the author re-reads `Description.md` aloud as if seeing the challenge for the first time and asks: *"Could an agent solve this purely from the description without needing to reason about the codebase?"* If yes, the description is over-specified and must be tightened.

**Docker offline verification prompt (hard deliverable)** — After the final offline audit checkpoint, the author **must** ask the user whether to run the Docker image in offline mode (`--network none`) to verify end-to-end. This prompt is mandatory and must not be skipped or deferred. The exact question to present:

> "All deliverables are drafted and the offline audit has passed. Would you like me to build and run the Docker image with `--network none` to confirm the challenge executes fully offline before we finalize?"

If the user confirms, the author builds the image and runs the container with `--network none`, captures the test output, and reports pass/fail. Any failure is a blocking defect. If the user declines, record the declination in `handoff.md` and note that the offline Docker run was skipped at user request.

---

## ⛔ Hard requirement: Dockerfile construction rules

The following rules are **absolute and non-negotiable**. Each one maps to a class of defect that has caused challenge rejections. Violating any rule is a **blocking error** that must be fixed before the Dockerfile is considered complete.

### Structure rules

| Rule | Required | Forbidden |
|------|----------|-----------|
| Working directory | `WORKDIR /app` | Any other path (`/vouch-proxy`, `/src`, `/project`, etc.) |
| Source files | `COPY . .` after `WORKDIR /app` | `RUN git clone …` / `RUN git checkout …` anywhere in the Dockerfile |
| Shell entry | `CMD ["/bin/bash"]` | `ENTRYPOINT ["/bin/bash"]` or any other `ENTRYPOINT` form |
| Tests at build time | _(none)_ | Any `go test`, `npm test`, `pytest`, `mvn test`, `./gradlew test`, or equivalent test invocation inside a `RUN` layer |
| Network at runtime | All deps installed in `RUN` layers at build time | `apt-get`, `pip install`, `npm install`, `go get`, `curl`, `wget` inside `test.sh` or any runtime entrypoint |

### Rule-by-rule rationale (memorise these — they explain every rejection)

1. **`WORKDIR /app` is mandatory.** The rubric's test harness and patch application tooling resolve paths relative to `/app`. Any other `WORKDIR` silently breaks the harness. Set it once, near the top, and never override it.

2. **Never `git clone` or `git checkout` inside the Dockerfile.** The build context already contains the repo at the pinned commit (it was cloned locally in the workspace). Use `COPY . .` to bring those files into the image. A `RUN git clone` line (a) requires network access during build, (b) may pull a different commit than intended, and (c) violates the offline requirement.

3. **Always `COPY . .` after `WORKDIR /app`.** Without this, the image has no source files — tests and builds inside the container will fail with "no such file or directory". This line is not optional and must not be omitted.

4. **No test invocations during `docker build`.** Tests are run by the evaluation harness *after* the image is built, by running a container from it. A `go test` (or equivalent) inside a `RUN` layer adds build time, can produce misleading pass/fail signals, and may fail in build environments with different constraints. Remove every test invocation from the Dockerfile — keep only dependency installation and compilation steps.

5. **Use `CMD ["/bin/bash"]`, not `ENTRYPOINT`.** `CMD` can be overridden at `docker run` time without extra flags; `ENTRYPOINT` cannot. The evaluation harness overrides the command to run `test.sh`; an `ENTRYPOINT` prevents this cleanly. Always end the Dockerfile with `CMD ["/bin/bash"]`.

### Dockerfile self-review checklist

Before writing or finalising the `Dockerfile`, the author must tick every item:

- [ ] `FROM` uses the base image named in reference.md (not an arbitrary base)
- [ ] `WORKDIR /app` is present and is the only `WORKDIR` in the file
- [ ] `COPY . .` appears immediately after `WORKDIR /app`
- [ ] There is no `git clone` or `git checkout` anywhere in the file
- [ ] There is no test invocation (`go test`, `pytest`, `npm test`, etc.) in any `RUN` layer
- [ ] The file ends with `CMD ["/bin/bash"]` (not `ENTRYPOINT`)
- [ ] Every `apt-get`, `pip install`, `go mod download`, or equivalent is in a `RUN` layer at build time, not in the entrypoint or `test.sh`
- [ ] The container runs the test harness fully **offline**: `docker run --network none <image>` executing `test.sh` succeeds (build time may use the network to pull the base image and install dependencies; runtime must not)

This checklist must be run and confirmed complete in `handoff.md` before the Dockerfile is handed off.

---

## Deliverables (summary)

| Artifact | Content |
| --- | --- |
| **`deliverables/metadata.md`** | Title, repo link, pinned commit, and category — all rubric-required metadata, and nothing else. |
| **`deliverables/Description.md`** | Observable problem statement only (headingless prose, ≤ 600 words / 700 hard cap; no title, repo link, commit, category, motivation, Constraints, or Out-of-Scope sections — see reference.md §Problem description format). |
| **`deliverables/Dockerfile`** | Reproducible environment and shell entry (`CMD ["/bin/bash"]`) built `FROM` the per-language `public.ecr.aws/d3j8x8q7/olympus-base-<language>:latest` image (see reference.md). **Must run with `--network none` at runtime** — all dependencies baked at build time. |
| **`deliverables/test.patch`** | Unified diff adding the test harness (e.g. `test.sh`) and new tests; `base` / `new` modes per reference.md. |
| **`deliverables/solution.patch`** | Implementation that satisfies the spec and makes the new tests pass. |

Patch application order, JUnit output path, and structural expectations are spelled out in reference.md.
