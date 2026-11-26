---
name: design-sketch
description: Scopes a change before any code exists by tracing the systems it touches, sketching two different shapes and choosing between them. Load when starting non-trivial work, a design or scoping session, or any change where jumping straight to code would lock in the wrong shape. Does not cover writing up finished work or reports.
---

# Design sketch

Decide the shape of a change before writing code. A shape chosen after the code exists is usually the shape the first draft happened to have.

## 1. Trace every system the change touches

List each system, module, service, data store and caller that reads, writes or depends on what you are changing. Read them rather than guessing. For each one, note what it owns and what it expects from its neighbours.

Why: a design drawn from the one file in view misses the consumers that break.

## 2. Sketch two different shapes

Write two sketches that differ in structure, not in naming or detail. A sketch gives:

- the types and their fields,
- the function or interface signatures,
- the module or service boundaries,
- who owns each piece of data.

Keep each sketch short enough to compare at a glance. If the two sketches would produce the same code, one of them is not a second shape: find a different split of responsibility.

## 3. Pick one shape

Choose the shape where:

- each piece of state has one owner,
- each task has one way to do it,
- callers need to know the least.

State the choice against these three tests. Do not pick by which sketch you drew first.

Why: state with two owners drifts out of sync, and callers that must know internals break when those internals change.

## 4. Redesign rather than patch

If implementing the chosen shape needs workarounds, and then more workarounds, stop. Repeated workarounds mean the shape is wrong. Go back to step 2 with what you learned and sketch again, instead of adding another patch.

## 5. Record the rejected shape and why

Keep a record with three parts:

- the shape you chose,
- the shape you rejected,
- the reason, tied to the tests in step 3.

Put the record where the reader of the change will find it, such as the design note, the brief or the report the run asks for, and in the PR text when the task has you write it. How it reads is covered by `handoff-reports`.

## Attribution

Adapted from the `architect` skill in michael-denyer/pstack-claude (MIT, Copyright (c) 2026 Lauren Tan and Michael Denyer), `plugins/pstack/skills/architect/SKILL.md` at commit `2f6de895e2a5a7a7a6428b105047a2b75327cd16`. The licence is in [`LICENSE`](LICENSE) and the source record is in [`ATTRIBUTION.md`](ATTRIBUTION.md).
