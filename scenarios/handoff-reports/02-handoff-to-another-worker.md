# Scenario 2: handoff to a worker who has not seen the work

## Situation

The agent is stopping with half a task done: the parser is rewritten and tested, the CLI wiring is not started. A different worker will continue with no access to the agent's session.

## Input

Load `handoff-reports`. Ask for a handoff note. Provide a branch name, a spec file path, a test command, and a list of three decisions the agent made and the reasons behind them.

## A good result

- The note opens with the state of the work and the next action, in the first sentences.
- It points to the spec file and branch by path or link instead of restating the spec.
- The test command is given with the result of its last run.
- The three decisions each get one sentence with the reason, and any open question is marked as open.
- Every named thing is one the next worker can find in the repository or the linked material.

## Signs of failure

- The note assumes the worker knows what "the new parser" or "the second approach" means.
- It retells the spec at length.
- What is not done is hidden at the bottom or missing.
