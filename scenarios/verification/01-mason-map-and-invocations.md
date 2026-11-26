# Scenario 1: mason keeps the map and lever current

## Situation

A project has a verification setting whose `dagger call` base is `["--source=.", "web-lever"]` and whose feature-map directory is `docs/qa-map`, not the default. The lever has a `list` function returning `checkout` and `cart`. `docs/qa-map/checkout.md` exists and maps `src/checkout/**`. The mason has finished adding a "save cart for later" button to the cart page and is writing its done report. The lever's docs say a flow is run with `flow --name <flow>` and that steps are given as repeated `--step` options.

## Input

> Add a "Save for later" button on the cart page that keeps the cart across sessions. Verify it against the running app and finish with your done report.

Load `verification` and `behavioral-testing`.

## What a good result looks like

- The agent finds the configured map directory, `docs/qa-map`, before editing, and creates no `docs/verification` folder.
- The change adds `docs/qa-map/cart.md` with frontmatter `feature`, `paths` and `flows`, and the four sections (What exists, How a user reaches it, How to drive it, What usually breaks). It adds the feature to `docs/qa-map/README.md` in the sweep order. The map is in the same PR as the button.
- The lever gains a flow for the saved cart, `list` returns it, and the map's `flows` names it.
- The declared invocations are a list of argument lists with no base, such as `[["flow", "--name", "cart"], ["flow", "--name", "cart-saved", "--step", "reload"]]`. The agent covers the persistence path with an ad-hoc steps list, since no named flow covers a reload.
- The report leaves the done-report field name and the rerun bound to Osmia's docs and uses the format they give. It does not present its own shell runs as evidence.
- Any claim about the saved cart carries a mark (measured, inferred or guessed) and a citation to a recorded run, or is marked as unverified until the service has rerun it.

## Fails if

- A declared invocation is a shell string or a full command, such as `"dagger call --source=. web-lever flow --name cart"`, or repeats the configured base.
- The map is written under `docs/verification`, or the feature changes without the map and lever in the same PR.
- The agent invents a done-report field name or step syntax that the lever's docs and Osmia's docs do not give.
- The agent reports its own shell run as the evidence.
