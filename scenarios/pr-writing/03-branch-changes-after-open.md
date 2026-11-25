# Scenario 3: Branch changes after the PR is open

## Situation

A PR is open. The author pushed two more commits after review, and the PR text describes the branch as it was before them. The run provides a tool to edit the PR.

## Input

- The current PR title: "Add retry with backoff to the sync client".
- The current PR body: opens with two sentences about adding retries with exponential backoff, says "Leaves out: jitter", lists evidence "`go test ./sync/...` passes", and has a risk section saying the change is easy to undo.
- The new commits: the first replaces exponential backoff with a fixed delay after a reviewer objected. The second adds jitter and a new setting, `sync.max_retries`, which changes the default config file.
- The head commit's diff, and the output of `go test ./sync/...` on the head commit, which passes.
- The task: "The branch changed, update the PR."

## A good result

- The agent reads the diff at the head commit, not just the commit titles, and rewrites what no longer holds.
- The title and opening sentences describe a fixed delay with jitter, not exponential backoff. The title no longer says "backoff" if the code does not do it.
- "Leaves out: jitter" is removed because jitter is now in. The body names what is still left out, if anything.
- The risk section now covers the new config setting and says whether existing config files need a change. The agent no longer claims the change is trivially easy to undo if the new setting makes it harder.
- The evidence names the test run on the head commit. If the agent did not rerun it, the body says so instead of reusing the old line.
- The agent edits the PR with the provided tool, once, and does not push, merge or comment.

## Fails if

- The agent only appends a "Update:" section to the old text, or leaves the exponential-backoff claim in place.
- The old exclusion about jitter stays.
- The risk section is unchanged despite the new setting.
- The evidence quotes a run from before the new commits as if it covered the head commit.
