Act as a senior Olympus Diamond-tier reviewer and environment linter.

I will paste my task folder contents:

* problem.md
* Dockerfile
* test.patch
* solution.patch
* command outputs
* Castor run results if available

Review the full task for Olympus/Diamond readiness.

Check:

Environment:

* Dockerfile uses the correct Olympus base image
* Dependencies are installed during build
* Runtime works with --network none
* /bin/bash entrypoint is used
* Dockerfile works before test.patch or solution.patch is applied
* test.patch applies cleanly
* solution.patch applies cleanly after test.patch
* test.sh exists at repo root
* test.sh supports:
  ./test.sh --output_path results.xml base
  ./test.sh --output_path results.xml new
* JUnit XML is written to the exact output path

Test execution:

* base mode passes on original commit
* new mode fails on original commit
* after solution.patch, base mode passes
* after solution.patch, new mode passes
* tests are deterministic and no-network

Solution quality:

* Solution satisfies every requirement in problem.md
* No regressions
* No unrelated changes
* No API breaks unless required
* No filler code, artificial line inflation, unnecessary defensive code, or AI slop
* Solution follows repo patterns
* Task is system-level and likely requires multi-file reasoning

Diamond readiness:

* Task is hard but solvable
* Ideal expected result is around 1–2 passes out of 10 strong agents
* Castor-only runs are used
* 10+ runs completed
* 1–5 successful runs
* Every failed run has fairness analysis
* Every failed run has root-cause analysis

Return:

Verdict:

* Approve
* Changes Requested
* Reject

Checklist:

* Description quality: Pass/Fail
* Test quality: Pass/Fail
* Env/Docker: Pass/Fail
* Solution quality: Pass/Fail
* Code fairness: Pass/Fail
* Diamond readiness: Pass/Fail

For every issue:

* Severity: Critical / Major / Minor
* Evidence
* Description
* Why it matters
* Suggested fix

Also provide:

* Missing test coverage
* Hidden requirement risks
* Ambiguous wording
* Env linter risks
* Code fairness risks
* Remaining blockers
* Final recommendation
