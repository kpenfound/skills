---
name: typescript-best-practices
description: TypeScript syntax for type design, narrowing, validation and function signatures. Load when reading or editing .ts, .tsx, .mts or .cts files, designing types, narrowing values, validating external data or choosing a function signature.
---

# TypeScript best practices

This skill is the TypeScript syntax for the principles in the `simple-code` skill. Read `simple-code` first for the reasoning behind type-system discipline and boundary discipline. This skill does not restate it.

A type error is a finding to fix in the types or the code. Never silence it inline. The `no-comments` skill owns that rule and the `lint-discipline` skill owns how to react to findings. Without them installed, still fix the cause.

| Rule                  | Summary                                                                                                                                                                                                                             |
| --------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Discriminated unions  | Model variants with a `kind` literal discriminant so impossible states cannot be represented. No optional-field bags.                                                                                                               |
| Branded types         | Brand primitives with `& { readonly __brand: "X" }` so they cannot be mixed up. Validate once at the boundary. A brand comes from a verifying type guard (`input is AgentId`), never from `as`.                                     |
| Constructive modeling | Build the shape so the illegal value cannot be constructed. `[T, ...T[]]` for non-empty, `[T, T][]` for even length, `start` plus `durationMs` for a range. Not a runtime guard.                                                    |
| Simplest total type   | Keep `T[]` while every operation on it stays total. Strengthen to `NonEmpty<T>` only where the loose type forces `!`, a cast or a "should never happen" throw.                                                                      |
| `unknown` over `any`  | External data is `unknown`. Narrow before use.                                                                                                                                                                                      |
| Schemas before guards | Before hand-writing a property-by-property type guard, use the repository's runtime schema library and infer the type from the schema, such as `z.infer`.                                                                           |
| No `as` casts         | Every `as` is a runtime crash waiting. Parse or narrow instead, so the compiler verifies the claim.                                                                                                                                 |
| Narrowing hierarchy   | Discriminant switch, then `in`, then `typeof` or `instanceof`, then a user-defined type guard.                                                                                                                                      |
| Type guards           | A guard must verify its claim. A lying guard is worse than `as` because the bug hides behind a name that says it is safe. Name guards `isX` or `hasX`.                                                                              |
| Exhaustiveness        | Assign the discriminant to a `never`-typed local in the default arm so the compiler errors when a variant is added.                                                                                                                 |
| `satisfies` over `as` | `satisfies` validates the value without widening literal types.                                                                                                                                                                     |
| Boundary validation   | Parse where data crosses in, into a named domain type. `Record<string, unknown>`, however spelled, stops at that parse. Trust types inside.                                                                                         |
| Schema-derived types  | Reach for `Pick`, `Omit`, `Parameters`, `ReturnType`, `Awaited` and `typeof` before declaring a new interface.                                                                                                                      |
| Object args           | Pass an object instead of several positional arguments so order is self-documenting. Skip it on hot paths such as per-frame render, tokenizers and parsers.                                                                         |
| Real tests            | Run the real thing instead of mocking it. Use the framework's own test primitives, including its leak and disposal checks. Mock only what cannot run locally. Test boundaries and goldens belong to the `behavioral-testing` skill. |
| Structured telemetry  | Use the structured logger with enough context to debug from an id. No `console.log` in shipped code.                                                                                                                                |

Examples for each rule are in [references/patterns.md](references/patterns.md).

## Attribution

Adapted from pstack, MIT licence, Copyright (c) 2026 Lauren Tan (see `LICENSE`), from `cursor/plugins` at commit `ec569379616a4826b98f0f0a205dd3411b573611`. Upstream paths:

- `pstack/skills/typescript-best-practices/SKILL.md`
- `pstack/skills/typescript-best-practices/references/patterns.md`
