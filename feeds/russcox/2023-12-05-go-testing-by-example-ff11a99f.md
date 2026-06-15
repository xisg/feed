---
title: Go Testing By Example
url: https://research.swtch.com/testing
published: "2023-12-05T13:00:00Z"
feed: russcox
guid: tag:research.swtch.com,2012:research.swtch.com/testing
---

# Go Testing By Example

I opened GopherCon Australia in early November with the talk “Go Testing By Example”.
Being the first talk, there were some A/V issues, so I re-recorded it at home and have posted it here:

Here are the 20 tips from the talk:

01. Make it easy to add new test cases.

02. Use test coverage to find untested code.

03. Coverage is no substitute for thought.

04. Write exhaustive tests.

05. Separate test cases from test logic.

06. Look for special cases.

07. If you didn’t add a test, you didn’t fix the bug.

08. Not everything fits in a table.

09. Test cases can be in testdata files.

10. Compare against other implementations.

11. Make test failures readable.

12. If the answer can change, write code to update them.

13. Use [txtar](https://pkg.go.dev/golang.org/x/tools/txtar) for multi-file test cases.

14. Annotate existing formats to create testing mini-languages.

15. Write parsers and printers to simplify tests.

16. Code quality is limited by test quality.

17. Scripts make good tests.

18. Try [rsc.io/script](https://pkg.go.dev/rsc.io/script) for your own script-based test cases.

19. Improve your tests over time.

20. Aim for continuous deployment.

Enjoy!
