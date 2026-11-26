---
name: verification
description: Drives a running build through a project's verification lever, keeps its feature map current, and ties every claim about running behavior to evidence. Use when driving the running app to check a change, writing or updating a lever or a feature map, reporting verification results in a done report or handoff, or reviewing a change from verification evidence.
---

# Verification

Tests say the code passes. Verification says the running build does what the change promised. It rests on three things a project provides:

- **The lever.** A Dagger function that starts the app, drives it with the arguments it is given, and returns a Directory holding `report.json` plus evidence files. The app can be a web UI, an API or a CLI, and any browser or client can sit behind the lever.
- **The feature map.** One short file per feature that says what exists, how to reach it and which flows exercise it.
- **Service-recorded runs.** The runs the harness itself records are the evidence. A run an agent makes in its own shell is not.

Authors use this skill to drive the lever, keep the map and lever current and report honestly. Reviewers use it to judge a change from recorded evidence rather than from the author's account.

## Find the project's settings first

Before running or editing anything, find out three things:

- The configured `dagger call` base: the module and options that every lever call starts with. It comes from the project's verification setting.
- The feature-map directory. It comes from the same setting and is `docs/verification` when the setting does not name one. When the setting is not visible to you, look for an existing map in the repository and use its directory. Do not create a map under `docs/verification` while another directory is configured: the harness reads only the configured directory, so a second map is never seen.
- The product lever's own docs, which give its flow names and step syntax.

When the project has no verification setting and no lever, this skill does not apply, and there is nothing to invent.

## Drive the lever

Two ways to drive it, both through lever arguments:

- **Named flows.** The map's files list the flows a feature has, and the lever's `list` function returns the same names. Run the flows for every feature the change touches.
- **Ad-hoc steps.** Write your own steps when the change reaches a path no named flow covers: a new entry point, a changed branch, an edge nobody mapped. Also write them for any path you would otherwise only claim works.

Cover the entry points of the feature, then the paths that apply from this set: success, cancel, error, empty state and persistence (does the result survive a reload, a restart, a second visit). A pass on the happy path says little about the others.

Every run starts a fresh instance of the app. State from an earlier run is gone, so a step that needs prior state sets it up inside the same run. Exploring means rerunning with more steps, not continuing a session.

Runs in your own shell are for iteration: they tell you what to fix and what to declare. They are not evidence, because nothing recorded them and a reader cannot tell what ran. The runs the service records when it reruns a mason's declared invocations, and the runs a reviewer makes through a `verify` tool when the harness offers one, are the evidence.

## The shape of an invocation

An invocation is an argument list. The harness appends it after the project's configured `dagger call` base, so the list holds only the lever's own arguments. It is not a full command line and not a shell string: no `dagger call`, no base arguments, no quoting, no pipes or redirects. A shell string is wrong because the harness appends the list as separate arguments, not as shell text, so quotes and operators reach the lever as literal characters.

An illustration of the shape, with arguments that stand in for whatever the product lever defines:

```json
["flow", "--name", "checkout"]
```

Named flows and ad-hoc steps are both just lever arguments. Their syntax belongs to the product lever's docs. Use what those docs say and do not guess at a universal step language.

A mason's **declared invocations** are a list of such argument lists, the ones it verified with. Put them in the done report. Osmia's docs give the exact done-report format and how many declared invocations the service reruns, so follow them and do not invent field names. Declare the invocations that cover the acceptance items, most important first, because a bounded number is rerun in declared order.

Keep two things apart:

- What a mason **declares** is a list of invocations for the service to rerun. It is a request, not a result.
- What the service **records** when it reruns them, or when a reviewer calls `verify`, is the result. Only the recorded result is evidence. A done report that says "all flows pass" without the recorded run behind it is a claim, not evidence.

The service also records skipped and refused invocations with reasons. A refused list started no run, so fix it and do not read the refusal as a failed feature.

## The lever's result

The Directory a run returns holds `report.json` and the evidence files it points to. The minimum an agent needs of the contract (version 1):

- `version` is `1`.
- `claims` is a list. Each claim has `name`, `status`, `detail`, `evidence` and an optional `flow`.
- `status` is `pass`, `fail` or `inconclusive`.
- `evidence` is a list of paths relative to the returned Directory: screenshots, accessibility snapshots, logs, HTTP responses, CLI output.
- `flow` names the flow the claim covers. Only this tag says which flow a run covered, so a claim without it covers no flow however its arguments read.

A missing or malformed report, an unknown status, an unsupported version or an evidence path that does not exist or escapes the Directory makes the whole run `inconclusive`. A lever that cannot reach a state reports `inconclusive` with the reason in `detail`, never `pass`, because a pass asserts something the run did not observe.

The lever also has a `list` function that returns its named flows. The map's `flows` field must name flows `list` returns. Osmia's docs hold the full contract; see [report example](references/report-example.md) for a worked `report.json`.

## Keep the map and lever current

A change to a feature's behavior, entry points or flows updates, in the same PR:

1. The feature's map file.
2. The lever, so that its flows and `list` still describe the feature.

A map that lags the code sends the next agent down paths that no longer exist, and a lever that lags passes flows that no longer test the feature.

The map lives in the configured directory, written below as `<map dir>` (`docs/verification` by default):

- `<map dir>/<feature>.md`: one file per feature.
- `<map dir>/README.md`: lists the features and the order of a full regression sweep. It is not itself a feature.

A feature file starts with frontmatter:

- `feature`: the feature's name.
- `paths`: repository globs for the code the feature lives in. The harness matches changed files against them to name the feature files that apply to a change, so keep them accurate and narrow.
- `flows`: the flow names the lever's `list` returns for this feature.

Its body has four sections, in this order:

1. **What exists.** The behavior the feature offers, in user terms.
2. **How a user reaches it.** The entry points: pages, endpoints, commands.
3. **How to drive it.** Which flows to run and which ad-hoc steps matter, written so someone can run them without reading the code.
4. **What usually breaks.** Failure modes seen before, which the next author should drive first.

[The map template](references/feature-map-template.md) shows the layout. Feature maps are user docs; whether a doc may exist at all is decided by `no-comments`, which this skill does not restate.

A new feature adds its file and a line in the README's sweep order. A removed feature removes both.

## Every claim carries evidence

Each claim about running behavior, in a done report, a handoff or a review, does two things:

- It cites its evidence: the recorded run and the evidence path that shows it.
- It is marked **measured**, **inferred** or **guessed**.

The marks mean:

- **Measured.** A recorded run observed it. The citation points at what was observed.
- **Inferred.** Reasoned from something measured or from the code, not observed itself. Say what it follows from.
- **Guessed.** Neither. Say so and say what would settle it.

`inconclusive` never counts as a pass. It means the run could not tell, and counting it as a pass turns "unknown" into "works". The same holds for a result that is missing. A missing, failed or inconclusive result on a user-visible acceptance item is reported as exactly that, with the run it came from. It is not rounded up, not folded into "mostly passing" and not left out. If a re-run can settle it, declare the rerun. If it cannot, hand the item back as unverified.

Verification results go to the mason in full. A failing or inconclusive result is a finding to act on, not something withheld.

## Review from evidence

A reviewer judges from the diff and the service-recorded runs, not from the author's handoff claims. Where the harness offers a `verify` tool, use it.

- **Write your own steps.** Rerunning the author's flows shows only what the author tried. Pick the paths the author skipped: the cancel, error, empty and persistence paths, and the entry points the diff adds. The author's flows being green says the author's paths work.
- **Check the requested artifact.** Confirm that the thing the task asked for exists in the diff and is the thing doing the work. Agents have built the tested behavior into a demo app or harness and left the requested library or component dead or missing, with every test and flow passing. Follow the call path from the entry point the user reaches to the artifact. If the flow passes without touching the requested code, the pass says nothing about the request.
- **Read lever and map edits closely.** A change that touches the lever or the map can make verification easier to pass: a flow dropped from `list`, an assertion loosened, a claim downgraded, a failing path removed from the map. Ask of each such edit whether the feature changed or the check did. Edits to these files are flagged in the review bundle, not rejected, so the judgment is yours.
- **Judge map staleness.** Compare the map prose with the diff. If entry points, flows or failure modes moved and the map did not, say so. The harness's structure check catches unreadable headers and flows missing from `list`. It cannot tell whether the prose is still true.
- **Hold the evidence rule.** A claim without evidence or a mark, a result counted as a pass when it was inconclusive, and a missing result on a user-visible acceptance item are findings.

The reviewer decides. No automatic gate turns a verification result into a block, so state the result and your judgment of it.

## Related skills

- `behavioral-testing` owns test design, boundaries and reliability. Load it when writing tests next to verification.
- `no-comments` owns whether a doc, including a feature map, may exist.
- `pr-review` grades a change against standards and spec. This skill supplies the runtime evidence for that grade.
