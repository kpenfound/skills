# Scenario 2: Repo conventions and no PR tool

## Situation

The repo uses conventional-commit prefixes and has a PR template. The run provides no tool for the PR host. The agent can read files and has the diff.

## Input

- `CONTRIBUTING.md`: "PR titles use `type(scope): subject`. Types are `feat`, `fix`, `docs`, `chore`. Squash merges use the PR title as the commit subject."
- `.github/pull_request_template.md` with the headings `## What`, `## Why`, `## Testing` and a closing line "Closes #".
- Recent merged titles: `fix(billing): round invoice totals half up`, `feat(export): add CSV export for reports`.
- A diff that adds a `--format csv` flag to the `report` command in `cli/report.go`, with a test. Issue #88 asks for CSV output from `report`. The issue also mentions a JSON format, which the diff does not add.
- The task: "Write the PR for this branch."

## A good result

- The title follows the convention: `feat(report): add CSV output to the report command` or similar, imperative, about 72 characters or fewer, no trailing period.
- The body follows the repo's template headings. It does not impose the default layout over a template the repo has. The first lines under `## What` are one to three sentences on the need and the result. A template section the change has nothing for is deleted or left out, not filled with "N/A".
- The body says that JSON output is left out and is a follow-up. It ends with `Closes #88`, or says that the issue stays open for JSON if the agent judges that the issue is not fully closed.
- The agent outputs the title, the body and a squash-merge commit message as text. It does not try to run `gh`, ask for a token or claim that it opened the PR.

## Fails if

- The title ignores the `type(scope):` prefix.
- The template is dropped for the default layout.
- The agent says "I opened the PR" or tries to reach the host.
- JSON output is not mentioned, so a reviewer cannot tell it was left out on purpose.
