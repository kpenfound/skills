---
name: unslop
description: Cut AI tells and filler from prose. Load when writing or editing pull request titles, descriptions and replies, commit messages, or user-facing documentation such as READMEs, guides, release notes and changelogs.
---

# Unslop

Edit text to remove AI patterns. Scan for the patterns below, then rewrite. Preserve meaning and match the intended tone.

## Scope

This skill improves the sentences of text that is already warranted:

- **PR text.** Titles, descriptions and replies to reviewers.
- **Commit messages.** Subject and body.
- **User docs.** READMEs, guides, reference prose, release notes and changelogs.

It does not decide whether a piece of text should exist, and it does not rule on change-history wording.

- Whether a code comment or an internal document may exist belongs to the `no-comments` skill. If that skill is installed, follow it first. If it is not, do not add a comment or internal document just to have something to edit.
- Wording that narrates change history ("now supports", "previously", "no longer") belongs to the `no-deltas` skill. If it is installed, follow it for docs, API doc comments, tests and PR text. A commit message or changelog entry states what the change does. It does not need to compare against an earlier implementation.

## Process

1. Read the text as the reader who will see it cold. Name what that reader needs to do or know.
2. Scan for the patterns below and mark each hit.
3. Rewrite each hit with the concrete fact, number, name or command it was avoiding. Delete a sentence that has none.
4. Read the result once more for patterns the rewrite introduced.

## Patterns in PR text and commit messages

1. **Vague titles.** "Update stuff", "Fix issues", "Improvements", a bare ticket id. Name the outcome: "Reject expired tokens at login".
2. **Subject lines that run long or mimic prose.** Use the imperative mood, no trailing period, about 72 characters at most. Put detail in the body.
3. **Restating the diff.** File lists and line-by-line "changed X to Y" repeat what the reader can see. Say why the change exists and what behavior it produces.
4. **Missing reason.** A description that says what and never why forces the reviewer to guess. Lead with the problem or need, then the approach.
5. **Inflated claims.** "Comprehensive", "robust", "seamless", "production-ready". Say what is covered and what is not.
6. **Unverifiable testing claims.** "Thoroughly tested" tells the reviewer nothing. Name the command or scenario that ran and its result, or say that nothing was run.
7. **Empty template sections.** Delete a heading that has nothing under it. Do not fill it with "N/A" or a restatement.
8. **Process narration.** "After some investigation I discovered...", "I then went on to...". Give the finding, not the diary.
9. **Reply padding.** Open a reply to a reviewer with the answer. Skip thanks, praise and "Great catch!" before it.

## Patterns in user docs

10. **Feature lists in place of tasks.** Organize by what the reader wants to do. One task per heading.
11. **Marketing adjectives.** "Powerful", "blazing fast", "effortless", "seamless". Give a number, a command or a behavior.
12. **Steps that do not run.** Every command and snippet should be copy-pasteable and show the output the reader will see.
13. **Imperative steps in second person.** "Run `make test`." instead of "The user should then run the tests."
14. **Drifting terms.** Name each concept once and keep the name. Do not rotate synonyms for a command, a file or a setting.
15. **Internals the reader does not need.** Remove implementation detail that does not change how they use the product.

## Patterns in all text

### Content

16. **Superficial -ing phrases.** "highlighting...", "ensuring...", "reflecting...", "showcasing...", "fostering...". Delete or expand with real sources.
17. **Vague attributions.** "Experts believe", "Industry reports suggest", "Some critics argue". Name the source or delete.

### Language

18. **AI vocabulary.** Additionally, crucial, delve, enduring, enhance, fostering, garner, interplay, intricate, landscape (abstract), pivotal, showcase, tapestry (abstract), testament, underscore, vibrant. Replace with plain words.
19. **Fancy ways to say "is".** "serves as", "stands as", "boasts", "features". Say "is" or "has".
20. **"Not just X, but Y."** State the point directly.
21. **Rule of three.** Do not force ideas into groups of three. Use the natural number.
22. **Synonym cycling.** Protagonist, main character, central figure and hero in one paragraph. Pick one word and repeat it.
23. **False ranges.** "from X to Y" where X and Y are not on a meaningful scale. List the topics directly.

### Style

24. **Em dashes.** Do not use them. Use periods or commas. Do not substitute parentheses, en dashes or a hyphen standing in for a dash. If a thought needs separation, end the sentence or use a comma.
25. **Colon overuse.** A colon is fine before a list or an example. It is not a mid-sentence connector. "If you're coming from traditional automation: instead of registering event handlers, you describe conditions" gains nothing from the colon. Rewrite so the point stands on its own: "Describing when the scheduler should fire works best as plain English."
26. **Boldface overuse.** Do not bold every proper noun or acronym.
27. **Inline-header lists.** The tell is a bold label and colon that restates the line: "**Performance:** Performance improved...". Convert those to prose. A bold lead-in that ends in a period, names the item and is followed by new detail ("**Schema in TypeScript.** Tables live in one file.") is fine.
28. **Title case headings.** Use sentence case.
29. **Decorative emojis.** Remove them from headings and bullets.
30. **Curly quotes.** Use straight quotes.

### Communication artifacts

31. **Chatbot phrases.** "I hope this helps!", "Let me know if...", "Of course!", "Certainly!", "Found the smoking gun!". Remove.
32. **Sycophantic tone.** "Great question! You're absolutely right!" Respond directly.

### Filler

33. **Filler phrases.** "In order to" becomes "To". "Due to the fact that" becomes "Because". Delete "It is important to note that".
34. **Excessive hedging.** "could potentially possibly be argued that it might" becomes "may".
35. **Generic conclusions.** "The future looks bright." State specific plans or facts.

### Jargon

36. **Abstract metaphor nouns.** Substrate (base), wedge (add), vector (way, method), locus, vantage, nexus, primitive as a noun, harness as a metaphor, surface as in "API surface", bedrock, scaffolding as a metaphor, modality, paradigm, gold-plating (more than the job needs), ratchet as a metaphor (the mechanism's real name, or a limit that only tightens), evacuate for moving code (move out), endgame (the last phase), north star, flywheel. These read as technical but usually have a plainer concrete word. The list is a set of clues, not a blacklist. Pick the concrete word.

### Plain speech

37. **Say what it does, not how it feels.** "the database stays close at hand", "SQL you can read" and "types that follow your schema" name a feeling. Name the mechanism or a number: "`.toSQL()` returns the exact string sent to the database", "a column rename fails the build". Ask what the sentence tells the reader to do or know, then write that. If you cannot restate it as a concrete instruction, fact or number, cut it. If the sentence could appear unchanged in another project's docs, it says nothing about this one. Cut it.
38. **Shorten or split dense sentences.** If the reader has to backtrack to parse a sentence, break it in two or drop clauses. One idea per sentence.
39. **Active voice.** Catch "is/are/was/were + past participle" and name the actor: "queries are validated" becomes "the compiler validates queries". Passive is fine only when the actor is unknown or does not matter.
40. **Cut adverbs, or use a stronger verb.** "runs quickly" becomes "is fast" or the number. "significantly improves" becomes the measured delta. An adverb propping up a weak verb means the verb is wrong.
41. **Prefer the plain word.** "utilize" and "leverage" become "use", "facilitate" becomes "help", "numerous" becomes "many", "in the event that" becomes "if".
42. **Mannered prose.** Metaphor or flourish where a literal phrase exists: aphorisms ("wire it or delete it"), rhetorical fragments for effect, personified code ("the plan holds it"), figurative verbs ("rides along", "stands on") and stock framing phrases. "A dial worth turning" becomes "a parameter worth varying". Say what you mean. The metaphor-nouns pattern covers the noun cases.
43. **Over-compression.** Dropped articles, verbless fragments, symbol-speak and abbreviations that make the reader decode. "Parser rejects bad date → exit 2, no write" becomes "The parser rejects a bad date, exits with code 2 and writes nothing." Write whole sentences and spell out arrows and abbreviations.

## Attribution

Adapted from pstack, MIT licence, Copyright (c) 2026 Lauren Tan (see `LICENSE`), from `cursor/plugins` at commit `ec569379616a4826b98f0f0a205dd3411b573611`. Upstream path:

- `pstack/skills/unslop/SKILL.md`
