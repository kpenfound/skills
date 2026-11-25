---
name: pr-writing
description: Writes the pull request title, the pull request body and the commit message that lands the PR, so a reviewer, and the pr-review skill, can read the body as the spec. Load when opening, drafting, editing or updating a PR, when a branch has changed since its PR text was written, or when asked to write or fix a commit message.
---

# PR writing

Write three pieces of text for a change: the PR title, the PR body and the commit message that lands the PR. The body is also the spec that `pr-review` grades the diff against, so it says what the change does and what it leaves out.

This skill decides what the text contains and how it is laid out. Sentence-level style belongs to the `unslop` skill. If it is installed, apply it to every sentence you write here. Change-history wording belongs to `no-deltas`.

## Before writing

1. Read the head commit and the full diff against the base branch. Write from what the diff does, not from what you meant to do.
2. Find the repo's own conventions: a PR template, `CONTRIBUTING.md`, `AGENTS.md`, `CLAUDE.md`, and the style of recent merged PR titles and commit subjects. A convention you find overrides the defaults in this skill.
3. Find the issue the change closes. Read it, so the body answers it.

## The body

Open with one to three sentences and no heading. They say the problem and the result: what was wrong or missing, and what is true now. A reviewer who stops after this paragraph still knows what the PR is for.

State what the change deliberately leaves out. Name work a reader might expect and that this PR does not do, such as a follow-up, a path left unchanged or a case not handled. If nothing is left out, say that the change is the whole of the issue.

Link the issue the change closes with a closing keyword, such as `Closes #123`, in the form the host recognises. If no issue exists, say so in one clause and put the requirements in the opening sentences.

`pr-review` treats the title, the body and the closing issue as the spec and reports anything the diff adds that the spec does not mention. A body that names its scope and its exclusions lets that review separate scope creep from intended work.

After the opening, add only the sections the change needs. Delete a section that has nothing in it. Do not fill it with "N/A".

### Approach

Add this when the way the change works is not obvious from the diff. Pick the smallest view that makes the point: pseudocode for logic, a call tree for control flow, a shallow file tree for a refactor, a diagram for interactions between parts. Show only the calls, files and boundaries the reader needs.

### Where to start reviewing

Add this when the diff has more than a few files. Name the files that carry the logic and the order to read them. Mark generated, vendored, renamed and other mechanical files so the reviewer can skip them. Call out a behaviour change that sits inside a mechanical change.

### Evidence

Say how the change was checked. List the commands that ran and their results, with the failing-then-passing test or the before and after output when the change fixes a bug. A screenshot is the best evidence for a visual change. Then say what was not checked and why. If nothing was run, say that.

### Risk

Add this when the change can hurt something. Say whether it is easy to undo or a one-way change, such as a migration, a deleted data path or a published interface. Say what else it could affect: consumers, other services, performance, permissions. Give the order of any migration or rollout and what must ship first.

## The title

- Use the imperative mood and name the result: "Reject expired tokens at login".
- Keep it to about 72 characters or fewer, in sentence case, with no trailing period.
- Follow the repo's own conventions when it has them, such as a conventional-commit prefix (`fix(auth): reject expired tokens`), a ticket key or a title format in the PR template. The convention wins over the defaults above, including the casing.
- Make the title and the body's opening agree. If the title needs "and" to cover the change, the PR may be two changes.

## The commit message

The commit that lands the PR carries the same facts as the PR text. For a squash merge the PR title is the subject and the body is the message body, so write them to stand on their own in `git log`.

- Subject: the same rules as the title.
- Body: why the change exists and what it does and leaves out. Skip anything the diff shows line by line.
- Trailers and issue references: follow the repo's convention.

## Keep the text true to the head commit

The title, the body and the commit message describe the head commit, not an earlier state of the branch. Whenever the branch changes, whether by a new commit, a rebase, a review fix or a dropped file, reread the diff and update the title and body in the same step. Check each claim in them: the files named, the evidence listed, the exclusions and the risk. Delete a claim that no longer holds, and run again any command the evidence section cites if the code it covers changed.

## Acting on the PR

Write the text. Open, edit or update a PR only when the run provides a tool for that, such as `gh`, a host API or a function the run's instructions name. Without one, output the title, the body and the commit message as text and stop. Never ask for credentials or invent a way to reach the host, because a run without a tool has not been cleared to act on the PR.

When a tool is provided, use it only for the action the run asked for. Do not push, force-push, merge, close, label or comment unless the run asked.

## Attribution

Adapted from the `pr` skill in mattpocock/skills (MIT, Copyright (c) 2026 Matt Pocock), `skills/engineering/pr` at commit `6f7f7831a625f040a8e06a0e7954826b87ce775a`, and from the `make-pr-easy-to-review` skill in pstack-claude (MIT, Copyright (c) 2026 Lauren Tan and Michael Denyer), `plugins/pstack/skills/make-pr-easy-to-review` at commit `2f6de895e2a5a7a7a6428b105047a2b75327cd16`. Both licences are in [`LICENSE`](LICENSE) and the source record is in [`ATTRIBUTION.md`](ATTRIBUTION.md).
