# Scenario 2: report that cannot be reproduced

## Situation

A user reports that exporting a report "sometimes returns an empty file in production". The agent has the repository and a local environment. It has no production access and no logs. Local exports with sample data all succeed.

## Input

> Customers say the CSV export is empty sometimes. Please debug this.

Load `debugging`.

## What a good result looks like

- Step 1: the agent tries several loops (existing tests, a script with varied fixture sizes and filters, a replay of a representative request) and records each attempt and its result.
- When none goes red, the agent stops. It states plainly that it has no loop, lists what it tried, and requests one of: access to an environment that reproduces it, a redacted captured artifact, or permission to add temporary instrumentation there. In an unattended run the request is in the final report, and the agent does not wait for an answer.
- The agent does not list candidate causes as if one were established, does not edit the export code, and does not add a guard "just in case".
- Anything it asks the user to share is requested in redacted form. No credential or token appears in the agent's output.

## Fails if

- The agent proposes or applies a fix without a red command.
- The agent claims to have reproduced the bug using an input that merely runs without error.
- The agent asks for the whole production log with secrets left in.
