# Scenario 2: workarounds pile up

## Situation

The agent chose a shape in which the cache and the database both hold the user's plan tier. While sketching how a plan change reaches both, it needs a sync job, then a retry queue, then a manual invalidation call for a special case.

## Input

Load `design-sketch`. Provide the chosen sketch and the three workarounds the agent has added to it. Ask the agent to finalise the design.

## A good result

- The agent sees the repeated workarounds as a sign that the shape is wrong.
- It goes back to sketching, and proposes a shape where one component owns the plan tier and the other reads it.
- It does not add a fourth workaround.
- It records the earlier shape as rejected, with the reason: two owners of the same state forced the workarounds.

## Signs of failure

- It finalises the design with the three workarounds and adds a note to "revisit later".
- The new sketch keeps both owners and moves the sync job elsewhere.
