# Scenario 1: intermittent failure in CI

## Situation

A service test passes locally and fails in about one CI run in twenty. The failing assertion is a count that is off by one. Nobody has reproduced it on a laptop. The agent has the repository, a shell and the CI log of one failed run.

## Input

> `TestBatchFlush` failed in CI again: `expected 10 items flushed, got 9`. It passes when I rerun. Can you find out why and fix it?

Load `debugging`, `simple-code` and `behavioral-testing`.

## What a good result looks like

- Step 1: the agent does not start by reading the flush code and theorising. It builds a loop that raises the failure rate, such as running the test a hundred times, in parallel or under CPU load, and reports the rate it reached. It shows one command and its output.
- Step 2: the loop is cut down (fewer items, one worker, no unrelated setup) and each cut is re-run. The agent states that removing any remaining element makes the loop pass.
- Step 3: three to five ranked candidates, each with a prediction (for example "if a timer race drops the last item, a pinned clock makes the failure vanish").
- Step 4: probes are tagged with a unique prefix and map to single candidates. The agent does not log everything.
- Step 5: the fix addresses the cause the probes confirmed, not a retry or a longer sleep. The debug prefix search returns nothing.
- Step 6: a deterministic regression test fails on the unfixed code and passes on the fixed code.

## Fails if

- The agent adds a retry, sleep or tolerance without naming the cause.
- The fix is applied before any loop with a measured failure rate exists.
- Debug logs remain in the diff.
