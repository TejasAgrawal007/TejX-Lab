Act as a senior Olympus/Diamond code fairness reviewer.

I will paste my problem.md, test.patch, test.sh, and the new test file.

Review whether the tests are fair and strictly aligned with the description.

Check:

* Every tested behavior is stated in problem.md or clearly inferable from repo conventions
* Tests validate public observable behavior, not private implementation details
* Tests do not check hidden requirements
* Tests do not enforce behavior that problem.md never mentions
* Tests do not contradict existing repo conventions
* Tests are deterministic and offline-safe
* Tests do not depend on timing, randomness, ordering, CPU count, filesystem state, or network
* Tests are strict enough to catch wrong solutions
* Tests are concise and not repetitive
* Tests cover success paths, failure paths, and obvious edge cases
* test.sh base mode runs relevant regression tests and should pass
* test.sh new mode runs only challenge tests and should fail on base
* JUnit XML is written to the exact --output_path

Return:

Verdict:

* Fair
* Needs Fixes
* Unfair

For every issue:

* Severity: Critical / Major / Minor
* Evidence from problem.md or tests
* Description
* Why it matters
* Suggested fix

Also provide:

* Requirement-to-test coverage matrix
* Tests that may enforce unspecified behavior
* Missing edge-case coverage
* Any unfair assumptions
* Final recommendation
