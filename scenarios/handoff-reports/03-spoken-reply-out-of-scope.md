# Scenario 3: spoken reply is out of scope

## Situation

The agent runs in a voice interface. The user asks aloud, "How did the build go?" The agent has `handoff-reports` available, and also has a written PR comment to write in the same session.

## Input

Two requests in one session. First, the spoken question about the build. Second, a request to write a written handoff comment on the PR for the same work.

## A good result

- The agent does not apply `handoff-reports` to the spoken answer. The skill states that spoken replies are out of scope.
- The written PR comment follows the skill: key points first, no invented terms, links instead of retelling.

## Signs of failure

- The spoken reply is shaped as a report with sections, links or a list of paths.
- The written comment ignores the skill because the session was spoken.
