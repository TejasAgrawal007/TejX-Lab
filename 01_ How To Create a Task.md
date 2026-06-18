Read the @client to understand how to create a task, and then create a task based on the issue: 

First, inspect the issue, repository, existing tests, and current open or merged PRs to verify that the issue is still present and not already fixed. Also verify that this issue can be tested deterministically without network access at runtime. If the issue needs live network, JSR, HTTPS, or any external service during test execution, stop and report that this issue is not suitable for Olympus.

After that, clone the repository and checkout one exact immutable commit SHA. Do not use a branch name, tag, or latest commit reference.

Write strict test cases following the repository’s existing test structure and naming conventions. Do not put the test in a random folder if the repo already has a proper place for that kind of test. The tests must fail completely on the base commit if the bug exists, and must pass only after the correct solution is applied. The tests should validate public observable behavior only, not internal implementation details. Do not add hidden requirements in the tests that are not clearly mentioned in the problem description.

Install all dependencies during setup/build and run the tests to verify whether the issue is still present in the selected commit. The tests must be deterministic, offline-safe, and should not depend on timing, randomness, ordering, machine state, or network access.

If the bug still exists and can be tested fairly, create a folder named task-1 containing these files:

1. problem.md

The first line should be the title. After that, write a concise and precise problem description. The description must be self-contained, clear, deterministic, and unambiguous. It should describe the expected behavior, not the implementation. Do not leak the solution. Do not repeat the same point again and again. Include all edge cases that the tests will check, so there are no hidden requirements.

At the end of problem.md, include:

* GitHub repository link
* GitHub issue link
* exact immutable commit SHA used for the task

2. Dockerfile

Create a Dockerfile based on the Olympus/Venus documentation and the provided sample. Use the correct base image for the repository language. Install all dependencies during the build phase. The container must work with --network none at runtime. The Dockerfile must work before test.patch or solution.patch is applied. Use /bin/bash as the entrypoint.

3. test.patch

Create a valid unified git diff containing only test changes. It must include:

* test.sh at the repository root
* the new or modified test file used to validate the bug

The test.sh file must support:
./test.sh --output_path results.xml base
./test.sh --output_path results.xml new

Base mode should run relevant existing regression tests and must pass on the base commit. New mode should run the challenge tests and must fail on the base commit. Both modes must write JUnit XML to the exact output path passed through --output_path.

Generate the test patch using:
git diff --cached > task-1/test.patch

4. solution.patch

Create a valid unified git diff containing the complete reference solution. The solution patch must not modify test.sh, test files, or Dockerfile. It should only include implementation changes. The solution must pass both base and new tests, follow the repo’s existing patterns, avoid unrelated changes, avoid API breaks unless required, and avoid filler code, defensive slop, or artificial line inflation.

Generate the solution patch using:
git diff --cached > task-1/solution.patch

Before final output, verify this full flow:

* checkout the exact selected commit
* apply task-1/test.patch
* run ./test.sh --output_path /tmp/base.xml base
* run ./test.sh --output_path /tmp/new.xml new
* confirm base passes and new fails
* apply task-1/solution.patch
* run ./test.sh --output_path /tmp/base_after.xml base
* run ./test.sh --output_path /tmp/new_after.xml new
* confirm both base and new pass after solution
* confirm the tests work with --network none
* confirm no existing PR already solves the exact issue

While writing problem.md and test cases, ensure high quality, strict alignment between problem and tests, and include only the necessary information. The task should be hard but solvable, fair for agents, and should not depend on hidden assumptions.
