# Scenario 1: New PR from a finished branch

## Situation

A branch fixes a bug and has one commit. No PR exists yet. The repo has no PR template and no written title convention. The run provides `gh`, and the task says to open the PR.

## Input

- The branch diff: `src/auth/session.ts` now rejects tokens whose `expiresAt` is in the past. The diff adds `src/auth/session.test.ts` with two tests. A generated file, `src/api/schema.gen.ts`, has a one-line regenerated change.
- Issue #214, "Expired tokens still log users in".
- The head commit message: "wip expired token".
- Test output: `npm test` passed with 142 tests. The integration suite was not run.
- The task: "Open a PR for this branch."

## A good result

- The title is imperative, names the result and is sentence case, about 72 characters or fewer, with no trailing period. Example: "Reject expired tokens at login".
- The body opens with one to three sentences and no heading. They say that expired tokens were accepted and that they are now rejected.
- The body says what the change leaves out. For example, it does not add token refresh, and it does not change the lifetime of tokens.
- The body links the issue with a closing keyword: `Closes #214`.
- The body has an evidence section that names `npm test` and its result, and says the integration suite was not run.
- The body marks `schema.gen.ts` as generated and points the reviewer at `session.ts` first.
- No approach section appears because the fix is plain. No risk section appears unless the agent names a real risk. No section holds "N/A".
- The agent opens the PR with `gh` because the run provided it, and does nothing else on the PR.

## Fails if

- The body starts with a "Summary" heading.
- The body lists each changed file.
- There is no statement of what is left out, or the issue is not linked.
- The title is "Fix issue" or ends in a period.
- The evidence says "tested" with no command.
