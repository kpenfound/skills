---
name: no-deltas
description: Write and review comments, documentation and tests that describe current behavior and meaningful contracts. Use when editing these artifacts or auditing for change-history commentary, development phase labels, and tests that only assert obsolete implementation details are absent.
---

# No deltas

Write for a reader who knows the current project but has never seen the patch,
its predecessor or its development plan. A comment should explain behavior,
constraints or rationale. A test should establish a meaningful contract.

Apply this guidance to the work being changed. Expand to a repository-wide audit
when requested. Preserve deliberately historical artifacts such as changelogs,
release notes and migration guides when their purpose requires describing changes.

## Comments and documentation

A **delta** describes the difference between implementations instead of giving
the reader a useful account of the current implementation.

- Explain what the code does, the invariant it preserves, or why a constraint
  exists. Keep code comments local to the nearby behavior; put feature
  explanations in documentation.
- Rewrite change narration such as "now supports," "previously," "used to,"
  "no longer," "replaces the old," and "unlike the previous implementation."
  Remove the comment if it adds nothing beyond the code.
- Use feature and behavior names in prose, identifiers, filenames and test
  names. Development phase labels and patch references do not explain a product
  contract.
- Preserve rationale that affects a current decision, including compatibility
  requirements and external constraints. Express the requirement directly.

| Change narration | Useful current explanation |
| --- | --- |
| `// We now persist the cursor before acknowledging the batch.` | `// Persist the cursor before acknowledging the batch so recovery resumes after durable work.` |
| `// Unlike the old implementation, retries don't send another payment.` | `// Reuse the operation key on retries so the provider can deduplicate the payment.` |
| `// Added cache lookup here.` | Remove it when the following cache lookup is self-explanatory. |

Words are clues, not a blacklist. "The previous attempt's operation key" can
describe a runtime relationship that matters. "The previous implementation"
usually introduces development history. Read the surrounding behavior before
editing.

## Tests

An **absence-only implementation test** checks that an obsolete name, source
fragment, file or implementation detail is gone, without establishing behavior
or a required structural contract. Avoid creating these as proof that a patch
was applied.

- Before adding a test, identify the observable behavior or invariant it protects
  and the incorrect behavior that would make it fail.
- Exercise the relevant boundary and assert its outcome. A regression test
  should expose the underlying failure even when implementation names change.
- Use a one-time source search to verify mechanical cleanup such as a rename.
  Add a permanent check only when the absence itself is an explicit maintained
  requirement, such as a packaging rule or architectural boundary.
- Preserve useful assertions when refactoring an existing test. Replace weak
  implementation checks with behavior checks where a real contract needs
  coverage; remove redundant checks when existing tests already cover it.

| Weak evidence of cleanup | Meaningful verification |
| --- | --- |
| Assert that source code does not contain `legacyAuthorize`. | Exercise a protected action without permission and verify rejection and unchanged protected state. |
| Assert that `oldCache` is absent from a source file. | Exercise invalidation and verify that the next read returns the updated value. |
| Add a test that searches every test name for a removed development phase label. | Audit the rename once, check references and confirm the intended tests remain discoverable. |

Negative assertions are useful when they express the contract. Keep tests for
invalid input, denied permissions, missing required configuration, duplicate
side-effect prevention, secret omission and unsafe state transitions. An
assertion that a secret is absent from a response can be a security requirement.
An assertion that an old helper name is absent from source usually proves only
an edit. Judge the behavior protected, regardless of assertion syntax.

## Review before finishing

Read the changed comments, documentation and tests without relying on the patch
description. Use `rg` to find suspected history phrases or retired names when
helpful, then inspect each match in context.

For each comment, ask: **Does this help explain the current behavior or a
constraint?** For each test, ask: **What meaningful failure would this catch?**

Check renamed references and preserve the intended test coverage. Follow the
repository's validation instructions, and report the checks actually performed.
Do not turn this guidance into brittle tests that ban words or match comment
wording.
