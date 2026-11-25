---
name: no-deltas
description: Writes and reviews user docs, API doc comments, tests and pull request text so they describe current behavior and meaningful contracts. Use when editing these artifacts or auditing them for change-history wording, development phase labels, and tests that only assert obsolete implementation details are absent.
---

# No deltas

Write for a reader who knows the current project but has never seen the patch,
its predecessor or its development plan. Prose should explain behavior,
constraints or rationale. A test should establish a meaningful contract.

This skill covers the wording of text and tests that are allowed to exist:

- user docs
- API doc comments
- tests
- PR text

Whether a code comment, an inline suppression or an internal doc may exist at
all belongs to the `no-comments` skill. When it is installed, follow it first.
When it is not, do not add a comment or an internal doc just to have something
to word.

Apply this guidance to the work being changed. Expand to a repository-wide audit
when requested. Preserve deliberately historical artifacts such as changelogs,
release notes and migration guides when their purpose requires describing changes.

## Docs and PR text

A **delta** describes the difference between implementations instead of giving
the reader a useful account of the current implementation.

- Explain what the behavior is, the invariant it preserves, or why a constraint
  exists. An API doc comment states the contract of the identifier it documents.
  Feature explanations belong in user docs.
- Rewrite change narration such as "now supports," "previously," "used to,"
  "no longer," "replaces the old," and "unlike the previous implementation."
  Delete the sentence if it adds nothing beyond the code or the surrounding docs.
- Use feature and behavior names in prose, identifiers, filenames and test
  names. Development phase labels and patch references do not explain a product
  contract.
- Preserve rationale that affects a current decision, including compatibility
  requirements and external constraints. Express the requirement directly.

The left column below shows change narration on purpose. It is a counterexample.
Do not copy it.

| Change narration (counterexample)                                    | Useful current explanation                                                                                 |
| -------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------- |
| `We now persist the cursor before acknowledging the batch.`          | `The consumer persists the cursor before it acknowledges a batch, so recovery resumes after durable work.` |
| `Unlike the old implementation, retries don't send another payment.` | `Retries reuse the operation key, so the provider deduplicates the payment.`                               |
| `Added cache lookup to the profile endpoint.`                        | `The profile endpoint caches each profile for 60 seconds.`                                                 |

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
  requirement, such as a packaging rule, an architectural boundary, or the
  repository's lint and markdown checks.
- Preserve useful assertions when refactoring an existing test. Replace weak
  implementation checks with behavior checks where a real contract needs
  coverage; remove redundant checks when existing tests already cover it.

| Weak evidence of cleanup                                                        | Meaningful verification                                                                            |
| ------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| Assert that source code does not contain `legacyAuthorize`.                     | Exercise a protected action without permission and verify rejection and unchanged protected state. |
| Assert that `oldCache` is absent from a source file.                            | Exercise invalidation and verify that the next read returns the updated value.                     |
| Add a test that searches every test name for a removed development phase label. | Audit the rename once, check references and confirm the intended tests remain discoverable.        |

Negative assertions are useful when they express the contract. Keep tests for
invalid input, denied permissions, missing required configuration, duplicate
side-effect prevention, secret omission and unsafe state transitions. An
assertion that a secret is absent from a response can be a security requirement.
An assertion that an old helper name is absent from source usually proves only
an edit. Judge the behavior protected, regardless of assertion syntax.

## Review before finishing

Read the changed docs, API doc comments, tests and PR text without relying on the
patch description. Use `rg` to find suspected history phrases or retired names
when helpful, then inspect each match in context.

For each sentence, ask: **Does this help explain the current behavior or a
constraint?** For each test, ask: **What meaningful failure would this catch?**

Check renamed references and preserve the intended test coverage. Follow the
repository's validation instructions, and report the checks actually performed.
Do not turn this guidance into brittle tests that ban words or match wording.
