---
name: lint-discipline
description: How to react to lint, type-check, test and repository check findings, and what to do when the same correction keeps recurring. Load when a check reports a finding, when tempted to silence, skip, baseline or loosen a check, when editing a baseline or `weeds.toml`, or when writing the same instruction a second time.
---

# Lint discipline

A finding is information about the code. React by changing the code, never the check. The check is how the repository detects this failure, so changing it hides the failure instead of removing it.

Adapted from pstack (MIT, Copyright (c) 2026 Lauren Tan) at `cursor/plugins` commit `ec569379616a4826b98f0f0a205dd3411b573611`: `pstack/skills/principle-encode-lessons-in-structure/SKILL.md`. The licence is in `LICENSE` beside this file.

Apply this to the findings in the files being changed. Work through unrelated findings across the repository only when asked.

## Fix the root cause

1. Read the finding and the rule behind it. Learn what failure the rule prevents.
2. Find the cause in the code, not the line the tool points at.
3. Change the code so the finding cannot occur: fix the type, remove the dead path, split the function, use the real API.
4. Run the check that reported the finding again, as narrowly as the repository allows, and confirm the finding is gone because the code changed.

A guard added to make a tool quiet is a symptom fix. If the cause is outside the files in scope, make the smallest in-scope fix and report the rest to the caller.

## Never suppress or weaken

- Never add an inline suppression. The `no-comments` skill defines what counts as one and owns the comment policy. A suppression switches the check off for that code, so the finding stays and nobody sees it.
- Never weaken a check to get past a finding: do not raise a threshold, loosen a strictness flag, disable or narrow a rule, widen an ignore pattern, exclude a path, or skip, delete or loosen a test. A weaker check stops catching the failure it was written for, for this finding and for every later one.
- Never replace a failing assertion with one that cannot fail. Such an assertion tests nothing, so the defect it caught goes unnoticed.
- A finding you disagree with is reported to the caller with your reasoning. It is not silenced.

## Exceptions live by name in `weeds.toml`

An exception exists only by name in the repository's `weeds.toml`, never inline. A named entry sits in one reviewed file and fails the check once stale, while an inline one stays hidden beside the code. Add or widen an entry only when the task or the owner asks for it. When a fix is out of scope, leave the finding in place and report the entry you would propose.

Remove stale entries. An exception, allowlist entry or baseline entry that matches nothing fails the check, so delete it as soon as the code it named is fixed or gone.

## The go-lint baseline

The baseline in `weeds.toml` holds complexity findings only: function length, cognitive complexity, nesting depth and similar. It exists for code that predates the policy.

- A finding from any other linter cannot be baselined. A baseline entry for one is a config error. Fix correctness, dead-code, commented-out-code and TODO findings in the code.
- The baseline only shrinks. Never add an entry and never widen one. The baseline exists for code that predates the policy, so growing it lets new debt in. New code that trips a complexity rule is refactored.
- An entry that matches nothing is stale and fails the check. Delete it when the function is fixed.
- Moving complexity from one function to another is not a reduction. Lower the complexity itself, by extracting a named step or removing a branch.

## Encode repeated lessons in structure

When you are about to write the same instruction a second time, or the same correction keeps recurring, the instruction is the symptom. Encode the lesson where it enforces itself, strongest form first:

1. A type that makes the wrong code unrepresentable.
2. A lint rule that rejects it.
3. A test that fails on it.
4. A repository check that fails on it.

Adding a rule or check only tightens enforcement, so it is always in bounds when the task covers it. Once the structure exists, delete the instruction it replaces.

When the lesson needs judgment and cannot be encoded, keep one short instruction in the README, `AGENTS.md`, `CLAUDE.md` or CONTRIBUTING, with an example of the failure it prevents. Make it more prominent before making it longer.

See the `simple-code` skill for the design principles that prevent most findings.
