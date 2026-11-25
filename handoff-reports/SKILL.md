---
name: handoff-reports
description: Guides written final reports and handoffs so a reader with little context can act on them. Load when finishing a task and writing the closing report, status summary, handoff note or hand-over message for someone who did not watch the work. Does not cover spoken replies, and does not cover scoping or sketching a change before code exists.
---

# Handoff reports

A final report or handoff is read later, by someone who did not see the work. Write it so that person can understand it and act on it without opening anything else first.

**Scope.** This skill covers written reports and handoffs only. Spoken replies are out of scope: do not apply it to voice or other spoken output.

## The reader

Assume an intelligent reader with little context. They have not seen your tool calls, your reasoning or the files you opened, and they may be switching in from other work. The report must make sense on its own.

## 1. Key points first

Put the outcome in the first few sentences: what is done, what is not, and what the reader must do or decide. Order the rest so each sentence only relies on what came before it. A term, name or claim in line 1 must not need line 12 to explain it.

Why: a low-context reader often stops after the opening. If the key points are buried, the report fails for them.

## 2. No invented terms

Do not use a label, abbreviation or nickname you made up during the work. Say what the thing is in words any reader would understand. Use the same word for the same thing throughout.

Why: a coined term feels like shared vocabulary to its author, but the reader never learned it.

## 3. Link, do not retell

When the material already exists, such as a PR, an issue, a commit, a spec, a log or a file, give a link or path and one sentence on what the reader will find there. Do not copy or summarise it at length. Quote only the line the reader needs for the decision in front of them.

Why: a retelling goes stale and can disagree with the source. A link stays true.

## 4. Plain, short writing

Use short sentences with one idea each, in the active voice. Use prose where ideas connect, and keep bullets for separate items. Leave out detail the reader did not ask for, but keep the context they need to act. For sentence-level tells and filler, follow `unslop`.

## Before sending

- Could the reader act on the first three sentences alone?
- Is every term one the reader already knows or the report defines?
- Does every pointer to existing material link to it?
- Does the report say what is not done or not checked?

## Source

Written in our own words from the "Communicating clearly" guidance in Anthropic's [prompting Claude Fable 5.2](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5-2) guide. No text is copied from it.
