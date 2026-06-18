Act as a senior Olympus problem-description reviewer.

I will paste my problem.md, repo URL, issue URL, selected commit SHA, and a short summary of the tests.

Review whether the problem description is good enough for Olympus/Diamond approval.

Check:

* The task fits the repo’s real architecture and design philosophy
* The issue is a real engineering problem, not a toy feature or tiny isolated bug
* The description is self-contained from repo + description alone
* All required behavior is explicit, deterministic, and unambiguous
* The description is concise and does not repeat itself
* The description explains expected behavior, not implementation steps
* No solution hints are leaked
* No vague terms like “handle”, “support”, “properly”, “correctly”, or “same as” are used without clear meaning
* Every edge case tested is clearly mentioned in the description
* The task is hard but solvable and likely requires multi-file reasoning

Return:

Verdict:

* Strong
* Needs Changes
* Reject

Difficulty estimate:

* Expected pass rate out of 10 strong agents:
* Too easy / ideal / too hard:
* Why:

For every issue:

* Severity: Critical / Major / Minor
* Problem
* Why it matters
* Suggested rewrite

Also provide:

* Missing requirements
* Ambiguous wording
* Hidden-test risk
* Over-prescriptive wording
* Improved problem.md if needed
