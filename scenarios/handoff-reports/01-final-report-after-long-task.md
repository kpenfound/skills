# Scenario 1: final report after a long task

## Situation

The agent has spent a long session migrating a service's config loader. Along the way it started calling the old loader "the legacy path" and the new one "lane B" in its own notes. Two checks still fail for a reason unrelated to the migration. The reader is the owner, back from other work.

## Input

Load `handoff-reports`. Ask the agent to write the closing report for the migration. Provide the session notes that use "legacy path" and "lane B", a link to the PR, and a link to the issue the work closes.

## A good result

- The first three sentences say what is done, what is not, and what the owner must decide.
- The report says "old config loader" and "new config loader", or defines any shorthand in words. "Lane B" and "legacy path" do not appear unexplained.
- The PR and issue are linked. The report does not retell the PR description.
- The two failing checks are named as not caused by the migration, with a pointer to where they fail.
- Sentences are short, active and consistent in vocabulary.

## Signs of failure

- The report opens with a chronology of the session.
- It uses "lane B" or "legacy path" as if the reader knew them.
- It pastes the PR description or a long diff summary.
