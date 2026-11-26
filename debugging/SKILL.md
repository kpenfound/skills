---
name: debugging
description: Walks a bug from report to verified fix in six checked steps (reproduce, minimise, list candidate causes, instrument, fix, add a regression test). Use when something is broken, throwing, failing, flaky or slow, when asked to debug or diagnose, or when a fix attempt did not hold.
---

# Debugging

A loop for bugs whose cause is not obvious. Run the steps in order. Each step ends with a check, and the next step does not start until that check holds. A skipped step needs a stated reason.

Redact every secret in the commands, output and captured files you show: write `<REDACTED>` in its place. Keep credentials in environment variables so the loop never carries them. Quote only the lines of a captured artifact that carry the signal. If redacted output is too thin to diagnose from, say so and name what would be enough. Put that request in the final report when no one is there to answer.

Root-cause principles live in `simple-code` §4, which covers fixing the fault rather than the place it surfaced, and how to treat guards and workarounds. Test design for step 6 lives in `behavioral-testing`. Load both. This skill sequences the work and does not restate them.

## 1. Reproduce

Build one command that goes red on this bug. This step is the skill. With a tight pass/fail signal, the cause can be found; without one, reading code does not substitute.

Ways to get a signal, roughly in order of preference:

1. A failing test at the seam that reaches the bug.
2. A script against a running service, or a CLI call with a fixture input diffed against known-good output.
3. A headless browser script that asserts on DOM, console or network.
4. A captured request, payload or event log replayed through the code path.
5. A throwaway harness running the smallest part of the system that holds the bug.
6. A property or fuzz loop when the symptom is "sometimes wrong output".
7. A bisection harness when the bug appeared between two known states: automate "check out the state, run, record" so `git bisect run` can drive it.
8. A differential run of the same input through two versions or configs.

Tighten the loop once it exists. Make it faster by narrowing scope and skipping unrelated setup. Make the signal sharper by asserting the specific symptom rather than "did not crash". Make it deterministic by pinning time, seeding randomness, isolating the filesystem and freezing the network.

For a non-deterministic bug, aim for a higher reproduction rate rather than a clean repro: repeat the trigger a hundred times, run in parallel, add load, narrow timing windows. A bug that fails half the time is debuggable and one that fails 1% of the time is not.

If no loop can be built, stop and say so. List what was tried and request one of: access to an environment that reproduces it, a redacted captured artifact (log dump, trace, core dump, recording with timestamps), or permission to add temporary instrumentation there. Put the request in the final report, and ask in the conversation only when a person is there to answer; never wait on a reply that cannot come. Do not move on to causes without a loop, because every later step consumes its signal.

**Check:** name the one command, show its invocation and output from a run you made, and confirm it is

- red-capable: it drives the real code path and asserts the symptom that was reported, so it can only go green once the bug is gone;
- deterministic: the same verdict every run, or a pinned, high reproduction rate;
- fast: seconds, not minutes;
- runnable unattended.

The failure it shows is the one that was reported, not a different failure nearby. A fix for the wrong bug is not a fix.

## 2. Minimise

Shrink the red scenario to the smallest one that still fails. Cut inputs, callers, configuration, data and steps one at a time, re-running the loop after each cut, and keep only what the failure needs. A smaller scenario leaves fewer parts to suspect in step 3 and becomes the regression test in step 6.

**Check:** removing any one remaining element turns the loop green. Record the exact symptom (error text, wrong output, timing) so later steps can confirm the fix addresses it.

## 3. List candidate causes

Write three to five candidate causes before testing any of them. A single candidate anchors on the first plausible idea. Rank them.

Each candidate states a prediction: "if X is the cause, then changing Y makes the bug disappear, and changing Z makes it worse". A candidate with no prediction cannot be tested, so sharpen it or drop it.

Show the ranked list to the user when one is present. They may know about a recent deploy or a cause already ruled out. Do not wait on a reply; proceed with your ranking if none arrives.

**Check:** every candidate has a prediction that one probe can confirm or refute.

## 4. Add logging or instrumentation

Each probe answers one prediction from step 3. Change one variable at a time.

Preference order:

1. A debugger or REPL inspection, when the environment supports it.
2. Targeted logs at the boundaries that tell the candidates apart.

Logging everything and searching afterwards buries the signal and slows the loop.

Tag every temporary log with a unique prefix such as `[DEBUG-a4f2]`, so cleanup is one search.

For a performance regression, logs rarely help. Take a baseline measurement first (timing harness, profiler, query plan), then bisect, and measure before fixing.

**Check:** the probe output has confirmed one candidate and ruled out the rest, or has shown what to add to the list. When no candidate survives, return to step 3 with the new evidence. Name the first place the system did something wrong, not only where the symptom appeared.

## 5. Fix

Apply the smallest change that removes the cause the probes confirmed, following `simple-code` §4. Remove the temporary instrumentation as part of the fix.

**Check:** the step 1 command, run on the original un-minimised scenario, is green, and a search for the debug prefix finds nothing.

## 6. Add a regression test

Turn the minimised scenario into a test at the seam where the bug actually occurs, using `behavioral-testing` to choose the boundary and what to assert. Write it before the fix when a correct seam exists: run it against the unfixed code and see it fail, then apply the fix and see it pass. When a mutation was needed to force the failure, diff against a pristine copy to prove the mutation landed.

When no correct seam exists, that is a finding. The structure of the code stops the bug from being locked down, so record this for the owner rather than adding a shallow test that gives false confidence.

**Check:** the regression test fails without the fix and passes with it, or the missing seam is written down.

## Finish

Before reporting done:

- The original reproduction no longer fails.
- The regression test passes, or the missing seam is documented.
- Throwaway scripts and harnesses are deleted, or moved somewhere marked as debug-only.
- The final report states the cause that turned out correct, so the next debugger learns it. Any commit or PR text the task has you write states it too.

## Attribution

Adapted from the `diagnosing-bugs` skill in mattpocock/skills (MIT, Copyright (c) 2026 Matt Pocock), `skills/engineering/diagnosing-bugs` at commit `6f7f7831a625f040a8e06a0e7954826b87ce775a`. The licence is in [`LICENSE`](LICENSE) and the source record is in [`ATTRIBUTION.md`](ATTRIBUTION.md).
