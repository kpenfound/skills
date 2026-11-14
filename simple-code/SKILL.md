---
name: simple-code
description: Language-agnostic design principles for writing, reviewing and debugging code. Load when shaping a change, deciding whether to add an abstraction, layer, validator or guard, tracing a bug to its cause, designing types or function signatures, or deciding where validation and error handling belong.
---

# Simple code

Six principles keep a change small, readable and correct. They apply in any language and to the change at hand. They do not call for sweeping refactors of code the task does not touch.

Comments, inline suppressions and lint findings have their own owners. The `no-comments` skill decides whether a comment, inline suppression or internal doc may exist. The `lint-discipline` skill covers how to react to lint and check findings. This skill does not restate their rules. When they are not installed, hold the same stance: reshape code instead of explaining it, and fix a finding instead of silencing it.

## 1. Laziness

Bias toward the smallest change that solves the problem.

- **Consolidate decisions.** Do not repeat the same choice in several places. Put it behind one source of truth and pass the result along as a simple value.
- **Minimize the diff.** Fewer lines beat elegant boilerplate. A new abstraction, layer or piece of plumbing needs a reason that the problem itself supplies.

## 2. Subtract before you add

When evolving a system, remove complexity first, then build. Default to subtraction.

- Sequence removal before construction.
- Cut before you polish. Reach the minimum before investing in quality.
- Design for observed usage, not speculative edge cases.
- Add no validators, parsers or guards beyond what the requirement demands.
- Delete dead code, redundant validators and stub references before building on the area.
- When a reference or helper holds no content of its own, delete it instead of leaving a stub.

Leave the design a little simpler and more capable behind the same or a smaller surface.

## 3. Minimize reader load

Maintainability is the work a reader must do to understand code. Track two independent axes:

- **Layers to trace.** The indirections between a question and its answer.
- **State to hold.** The hidden or mutable context a reader must keep in their head.

A flat file with fifty globals can be as hard to reason about as a six-layer adapter stack. Guard both. Line count, cyclomatic complexity and architectural labels are proxies. Reader load is the thing that matters.

- **Collapse layers** that cost more than they save: wrappers with one caller, adapters with no second implementation, indirection built for a need that never came. Inline them.
- **Make adjacent layers change the abstraction.** A layer that repeats the same methods and arguments adds load without compression. Collapse pass-through layers.
- **Demand interface compression.** A broad interface that hides little forces readers to learn both the surface and the implementation. Prefer boundaries that hide meaningful decisions.
- **Shrink state scope.** Prefer pure functions (returns over mutations), locals over fields, fields over module state, and module state over globals. Derive values instead of syncing copies.
- **Name the invariant at the boundary,** not in every consumer, so the reader learns it once.
- Before adding a layer or a piece of state, ask whether it reduces reader load elsewhere by at least as much as it adds.

The test: can a new reader answer "where does X come from?" and "what can change X?" in under thirty seconds? If not, cut layers or cut state.

## 4. Fix root causes

Fix the fault, not the place where it surfaced. The symptom is where a wrong state became visible. The root cause is the first place the system did something it should not do, or failed to handle a state it should handle.

A symptom fix hides the evidence and leaves the fault free to surface elsewhere. Each one leaves a workaround for a bug nobody named.

- Reproduce first.
- Ask "why" at each step until the answer is a wrong decision in the code, not a consequence of an earlier one.
- When stuck, instrument. Read the actual error and the actual state. Do not guess.
- Ask whether the state that triggered the symptom is legitimate. If the system may be in that state, the code that broke on it is the root cause, and handling the state there is the fix. If the state should never exist, a check where it broke hides the bug. Find what produced the state. Guards, retries, swallowed errors, clamped values and discarded state are the usual ways to hide a bug.
- When something fails only after a restart, suspect persisted state first: config files, caches, lock files, serialized state.
- Check for the pattern, not just the instance. Find every place with the same fault and fix them all.
- A workaround that needs a justification to be understood means the code is wrong. Reshape the code. A type error or failing check is a finding to fix at its source, never to silence inline.
- When the deciding fact took a deep dig to find, propose the cheap, always-on signal that would have shown it at first look.

## 5. Type system discipline

Make the compiler reject impossible states, mismatched primitives and unhandled variants. Define errors and special cases out of existence instead of proliferating handlers.

- **Make illegal states unrepresentable.** Model variants as a tagged union or sum type, not a bag of optional fields and flags. Build the shape so the invalid value cannot be constructed.
- **Brand semantic primitives.** `UserId` and `OrderId` are strings underneath but must not be interchangeable. Give each its own type, such as a branded type or a newtype. Validate once at creation and trust the type downstream. If two arguments share a primitive type but mean different things, brand them.
- **Exhaust variants.** A new variant should break the build at every place that must handle it.
- **External data is untyped until parsed.** RPC payloads, JSON, IPC messages, CLI arguments, config files, environment variables and database rows each get a parse function at the boundary that turns unstructured input into the typed model.
- **Derive types from authoritative schemas.** When a protocol buffer, OpenAPI spec, GraphQL schema, database migration, design-token file or other file owns a shape, derive from it instead of hand-rolling a parallel type.
- **Prefer the simplest total type.** Strengthen a loose type only where it forces a lie at a use site, such as an assertion that a value exists or a "should never happen" branch.

When the language is TypeScript and the `typescript-best-practices` skill is installed, load it for the syntax of these rules.

## 6. Boundary discipline

Place validation, type narrowing and error handling at system boundaries. Trust internal code. Keep business logic in pure functions and the shell thin and mechanical.

Scattered validation is noisy, redundant and gives a false sense of safety. Logic kept out of framework wiring can be tested without the framework.

- **At boundaries** (CLI arguments, config files, external APIs, network protocols): validate, return errors, handle defensively.
- **Inside the system:** typed data, error propagation, no re-validation. Trust the types.
- **Across the boundary:** expose domain concepts, not the boundary's private representation. Do not re-export transport, storage, framework or wire types through the public surface. Keep general-purpose mechanism inside and special-purpose policy at the edge.

In practice:

- Validate config at parse time, not inside business logic.
- Parse raw data into domain types at the boundary.
- Skip null checks deep in a call chain when the boundary already validated.
- Write parsers as pure transforms from raw input to typed state, and scoring or assessment as pure transforms from state to results.

The tests:

- Is this data crossing a system boundary right now? If not, validation is redundant.
- Can this be a pure function that the shell just calls? If yes, extract it.

## Applying the six together

1. When debugging, start with principle 4. Do not add a guard until the cause is known.
2. When adding, apply principle 2 first and principle 1 second. Remove what is in the way, then make the smallest change.
3. When data crosses in, apply principles 6 and 5. Parse once at the edge into a named type, then trust it.
4. Before finishing, apply principle 3. Count the layers and the state the change adds, and cut what does not pay for itself.

## Attribution

Adapted from pstack, MIT licence, Copyright (c) 2026 Lauren Tan (see `LICENSE`), from `cursor/plugins` at commit `ec569379616a4826b98f0f0a205dd3411b573611`. Upstream paths:

- `pstack/skills/principle-laziness-protocol/SKILL.md`
- `pstack/skills/principle-subtract-before-you-add/SKILL.md`
- `pstack/skills/principle-minimize-reader-load/SKILL.md`
- `pstack/skills/principle-fix-root-causes/SKILL.md`
- `pstack/skills/principle-type-system-discipline/SKILL.md`
- `pstack/skills/principle-boundary-discipline/SKILL.md`
