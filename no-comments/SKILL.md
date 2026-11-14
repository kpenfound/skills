---
name: no-comments
description: Decide whether a code comment, an inline lint or type-check suppression, or an internal markdown doc may exist. Load before adding, keeping or reviewing comments, suppression directives, TODO notes, commented-out code, or markdown files that explain code, and when a change touches any of them.
---

# No comments

Comments, inline suppressions and internal markdown docs are excuses written next to code. This skill decides which of them may exist and what to do with the rest. The answer is almost always to change the code so the excuse is unnecessary.

Apply it to the files being changed or the files the caller names. Sweep the whole repository only when asked.

Adapted from pstack (MIT, Copyright (c) 2026 Lauren Tan) at `cursor/plugins` commit `ec569379616a4826b98f0f0a205dd3411b573611`: `pstack/skills/no-comments/SKILL.md` and `pstack/agents/comment-sicko.md`. The licence is in `LICENSE` beside this file.

## Allowed comments

The list is closed. Nothing else is allowed.

- Doc comments on exported identifiers in importable packages.
- Compiler and tool directives that suppress nothing. In TS/JS: triple-slash references, `@ts-check`, `@jsx*` and `@flow` pragmas, source-map comments. In Go: `//go:` directives.
- Generated files.
- Licence headers.

Wording for allowed doc comments follows the `no-deltas` skill and the sentences follow the `unslop` skill, when they are installed.

## Banned comments

- Comments inside function bodies.
- Comments on unexported code.
- Comments under `internal/` and in `main`.
- TODO, FIXME and similar markers. Do the work, or report it to the caller.
- Commented-out code. Version control keeps the old version.
- Detached comments: a trailing comment on a line of code, or a doc comment separated from its declaration by a blank line.
- Banners, section dividers, narration of what the next line does, and justifications of why the code is shaped the way it is.

## Inline suppressions

These eight forms are banned in every file kind, code, config and scripts alike:

- `//nolint`
- `lint:ignore`
- `eslint-disable`
- `@ts-expect-error`
- `@ts-ignore`
- `@ts-nocheck`
- `ast-grep-ignore`
- `# noqa`

No inline directive is ever honoured as an exception. The same goes for any other directive that switches a check off for the code after it, whatever its spelling. Delete it and fix what the check reported; the `lint-discipline` skill covers how to react to findings.

Exceptions exist only by name in the repository's `weeds.toml`, never inline. Add or widen an entry only when the task or the owner asks for it. Otherwise delete the suppression, fix the finding if the fix is in scope, and put the proposed named entry in the report.

Tool directives that suppress nothing stay allowed, as listed above. `@ts-check` turns checking on, so it is a directive, not a suppression.

## Reshape, do not explain

A comment that explains surprising code is a defect in the code. A shorter comment does not fix it. Change the code until the surprise is gone:

- Rename the symbol so its name carries the fact.
- Extract the odd condition or step into a function named for what it means.
- Replace magic values with named constants.
- Encode the constraint in a type so the wrong call does not compile.
- Delete the dead or defensive path the comment apologises for.

```ts
if (user.status === 3 && Date.now() - user.seenAt > 86400000) {
  suspend(user);
}
```

```ts
const DAY_MS = 24 * 60 * 60 * 1000;

function isDormant(user: User, now: number): boolean {
  return user.status === Status.Active && now - user.seenAt > DAY_MS;
}

if (isDormant(user, Date.now())) {
  suspend(user);
}
```

If the root cause lies outside the files in scope, make the smallest in-scope fix, delete the comment, and report the rest to the caller. See the `simple-code` skill for the design principles behind reshaping.

## Check excuse wording before judging

`IMPORTANT`, `do not remove`, `too risky`, `fine for now`, `workaround`, and long justifications are clues, not verdicts. A comment that claims a constraint may be true, stale or wrong.

1. Read the code around it.
2. Find the callers of the symbol it names and the tests that exercise it.
3. Decide from what those show:
   - The claim is false or stale: delete the comment and any dead code it protected.
   - The claim is true because of our own code: delete the comment and reshape the code so the constraint is obvious.
   - The claim is true because of something we cannot change, such as a dependency, platform or protocol: delete the comment and express the behaviour in code. A well-named wrapper function holds the quirk in one place; a test pins it.
4. When the claim stays unclear after reading the callers and tests, the comment is deleted.

## Constraint comments

A comment saying `do not remove`, `do not change the wording` or `ask before changing` states a constraint without enforcing it. Encode it as a type, a test or a check when that is cheap and within scope. Then delete the comment. When encoding is out of scope, delete the comment anyway and report the constraint as unenforced, with a sketch of the type, test or check that would enforce it.

## Internal markdown

Markdown that explains how code works rots next to the code. Put that knowledge in names, types and tests.

Always allowed:

- README
- `AGENTS.md` and `CLAUDE.md`
- CONTRIBUTING
- LICENSE
- changelogs and release notes
- user docs
- files a test reads
- skill folders installed by Weeds

Recognise those categories this way:

- User docs are files under a `docs/` tree.
- Files a test reads are markdown under `testdata/` or under a test fixture directory.
- A skill folder counts as Weeds-installed only when it carries Weeds' ownership marker. A skills folder without the marker is not covered.

A product-intent doc, or any other explanatory markdown outside those locations, stays only when it is named on the `weeds.toml` markdown allowlist. Add the entry only when the task or the owner asks for it. Any other internal doc is deleted. Move what is still true into names, types, tests or the README, then delete the doc.

Prose inside the allowed files follows the `no-deltas` and `unslop` skills, when they are installed.

## Languages

Lint enforces the comment rules for Go and TS/JS only (`.go`, `.ts`, `.tsx`, `.js`, `.jsx`, `.mjs`, `.cjs`). The suppression and markdown scans cover every file kind.

In every other language, such as Dang, TOML, YAML and shell, nothing enforces the comment rules, so apply the same policy by hand. These languages have no exported identifiers, so the allowed comments reduce to tool directives that suppress nothing (a shebang line, for example), generated-file markers and licence headers. Everything else in the banned list is deleted.

## Report

Name the files touched, the comments deleted, the code reshaped, the suppressions removed with the named entry proposed for each, the constraints left unenforced, and any work left open.
