# Skills

Agent skills for coding agents. Each top-level folder holding a `SKILL.md` is one skill. An agent loads a skill when its `description:` matches the task.

## Skills

| Skill                                                             | What it does                                                                                 |
| ----------------------------------------------------------------- | -------------------------------------------------------------------------------------------- |
| [`behavioral-testing`](behavioral-testing/SKILL.md)               | Designs, writes and reviews tests that protect behavior through implementation changes.      |
| [`debugging`](debugging/SKILL.md)                                 | Walks a bug from report to verified fix in six checked steps.                                |
| [`design-sketch`](design-sketch/SKILL.md)                         | Scopes a change before any code exists by sketching two shapes and choosing one.             |
| [`handoff-reports`](handoff-reports/SKILL.md)                     | Guides written final reports and handoffs for a reader with little context.                  |
| [`hill-climb-evaluator`](hill-climb-evaluator/SKILL.md)           | Writes, reviews and judges the evaluator and metric of a hill climb.                         |
| [`lint-discipline`](lint-discipline/SKILL.md)                     | Sets how to react to lint, type-check, test and repository check findings.                   |
| [`no-comments`](no-comments/SKILL.md)                             | Decides whether a code comment, a lint suppression or an internal Markdown doc may exist.    |
| [`no-deltas`](no-deltas/SKILL.md)                                 | Keeps docs, API comments, tests and PR text describing current behavior, not change history. |
| [`pr-review`](pr-review/SKILL.md)                                 | Reviews a change along two axes: the repository's standards and the spec the PR text states. |
| [`pr-writing`](pr-writing/SKILL.md)                               | Writes the PR title, the PR body and the commit message that lands the PR.                   |
| [`simple-code`](simple-code/SKILL.md)                             | Gives language-agnostic design principles for writing, reviewing and debugging code.         |
| [`spoken-replies`](spoken-replies/SKILL.md)                       | Shapes replies that are heard aloud rather than read.                                        |
| [`typescript-best-practices`](typescript-best-practices/SKILL.md) | Gives TypeScript syntax for type design, narrowing, validation and function signatures.      |
| [`unslop`](unslop/SKILL.md)                                       | Cuts AI tells and filler from prose.                                                         |

## Authoring conventions

These follow Anthropic's [skill authoring best practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices).

- **Folder name.** The folder name equals the `name:` in the frontmatter. A name is at most 64 characters of lowercase letters, digits and hyphens, and does not contain `anthropic` or `claude`.
- **Description.** The `description:` is in third person ("Cuts AI tells…", not "Cut AI tells…"), says what the skill does and when to load it, is at most 1,024 characters and contains no XML tags.
- **Size.** The `SKILL.md` body stays under 500 lines. Longer detail goes in a file inside the skill folder, linked from `SKILL.md`.
- **References one level deep.** `SKILL.md` links to its support files. Those files do not link on to other files in the skill folder.
- **Never rules carry a reason.** A "never" rule states why it holds. The rule itself stays absolute.
- **Attribution.** Text adapted from another repository gets an `## Attribution` section in `SKILL.md` naming the upstream repository, path and commit, an `ATTRIBUTION.md` with the same record, and the upstream licence in `LICENSE`. Text written in our own words needs none.
- **Scenarios live outside skill folders.** Hand-run test scenarios are kept in `scenarios/<skill>/`, so they are never sent to an agent with the skill. A skill folder holds no scenarios.
- **Formatting.** Markdown is formatted with the pinned Prettier that `dagger check` runs.

`dagger check` enforces the name, description, size and reference rules for every top-level folder that holds a `SKILL.md`. It reads files only: it calls no model and needs no credential. Each failure names the skill, the rule and the fix.

## Which skill owns which rule

Each rule has one owner. A skill that needs a rule points to its owner and does not restate it.

| Rule                                                              | Owner                       |
| ----------------------------------------------------------------- | --------------------------- |
| Whether a comment, suppression or internal Markdown doc may exist | `no-comments`               |
| Sentence style and AI tells in prose                              | `unslop`                    |
| Change-history wording in docs, tests and PR text                 | `no-deltas`                 |
| Test design, boundaries and reliability                           | `behavioral-testing`        |
| Design principles, including root-cause tracing (§4)              | `simple-code`               |
| How to react to a failing check                                   | `lint-discipline`           |
| TypeScript type and signature syntax                              | `typescript-best-practices` |
| PR title, body and commit message content                         | `pr-writing`                |
| Grading a change against standards and spec                       | `pr-review`                 |
| The six-step bug loop                                             | `debugging`                 |
| Scoping and shape choice before code                              | `design-sketch`             |
| Layout of written reports and handoffs                            | `handoff-reports`           |
| Spoken reply style                                                | `spoken-replies`            |
| Hill-climb evaluator and metric                                   | `hill-climb-evaluator`      |

## Scenarios

The new skills have three hand-run scenarios each:

- [`scenarios/pr-writing`](scenarios/pr-writing/README.md)
- [`scenarios/debugging`](scenarios/debugging)
- [`scenarios/handoff-reports`](scenarios/handoff-reports)
- [`scenarios/design-sketch`](scenarios/design-sketch)

Give the agent the skill and the input a scenario describes, then compare the output with its "A good result".
