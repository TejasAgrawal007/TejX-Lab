Act as a strict senior Olympus benchmark reviewer.

I will provide:

* problem.md
* Dockerfile
* test.patch
* solution.patch
* local command outputs
* base/new test results before solution
* base/new test results after solution

Review the full submission exactly like an Olympus reviewer before running agent/Castor checks.

Primary target:
This task should be hard but solvable. The task should be difficult enough that strong agents may struggle, but it must still have at least one valid solution path proven by the reference solution.

Check these areas:

1. Problem quality

* Real-world engineering problem
* Fits repository architecture and design philosophy
* Self-contained from repo + description alone
* Clear, deterministic, and unambiguous
* Behavior-focused, not implementation-prescriptive
* No hidden requirements
* No vague wording
* No repeated or irrelevant context
* Hard enough to require long-horizon reasoning across multiple files/components

2. Test quality

* test.patch is valid and contains only test changes
* test.sh exists and supports base/new modes
* JUnit XML writes to exact --output_path
* base mode passes on original commit
* new mode fails on original commit
* after solution, both base and new pass
* tests are deterministic and offline-safe
* tests validate public observable behavior
* tests do not check private internals
* tests cover all required behavior and obvious edge cases
* tests do not enforce unspecified behavior
* tests are concise and not repetitive

3. Solution quality

* solution.patch is valid and does not modify tests or Dockerfile
* solution satisfies every requirement in problem.md
* no regressions
* follows repo conventions
* no unrelated refactors
* no API breaks unless required
* no filler code, artificial line inflation, defensive slop, or AI-style boilerplate
* solution is meaningful and system-level, not a tiny isolated fix

4. Environment quality

* Dockerfile uses correct Olympus base image
* dependencies are installed during build
* runtime works with --network none
* /bin/bash entrypoint is used
* Dockerfile works before patches are applied
* test.patch applies cleanly
* solution.patch applies cleanly after test.patch
* local verification commands prove the expected pass/fail behavior

Return:

Verdict:

* Locally Ready
* Changes Requested
* Reject

Difficulty estimate:

* Expected pass rate out of 10 strong agents:
* Too easy / ideal / too hard:
* Reason:

Checklist table:

* Repo suitability: Pass/Fail
* Description quality: Pass/Fail
* Test quality: Pass/Fail
* Solution quality: Pass/Fail
* Env/Docker: Pass/Fail
* Local verification: Pass/Fail
* Overall readiness: Pass/Fail

For every issue found:

* Severity: Critical / Major / Minor
* Evidence from provided files/logs
* Description
* Why it matters
* Suggested fix

Also include:

* Requirement-to-test coverage matrix
* Missing test coverage
* Hidden requirement risks
* Ambiguous wording risks
* Unfair test risks
* Env linter risks
* Solution/regression risks
* Remaining blockers
* Final recommendation before running agent/Castor checks
