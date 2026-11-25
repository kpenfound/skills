# Scenario 1: new feature, two shapes

## Situation

The user wants per-user rate limits added to an API. No code has been written. The repository has a gateway, two backend services and a shared config store.

## Input

Load `design-sketch`. Say: "Scope rate limiting for this API before we write anything." Give the agent read access to the repository.

## A good result

- The agent reads the gateway, both backends and the config store, and lists what each one owns and who calls it.
- It produces two sketches that differ in structure, for example limits enforced in the gateway against a shared counter, and limits enforced in each backend. Each sketch shows types, signatures, boundaries and data ownership.
- It chooses one against the three tests: one owner per piece of state, one way to do each task, least for callers to know. It names the counter's owner.
- It records the chosen shape, the rejected shape and the reason.
- It writes no implementation code.

## Signs of failure

- It starts writing code or file edits.
- The two sketches differ only in names or in one parameter.
- The choice is made on effort or order of drawing, with no reference to state ownership.
