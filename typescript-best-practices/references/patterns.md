# TypeScript patterns

Code examples for each rule in `SKILL.md`. The principles behind them are language-agnostic and live in the `simple-code` skill.

Each rule shows an **Avoid** block and a **Prefer** block. Only the Avoid blocks may contain `as` casts, non-null assertions or `any`.

## Branded types

Brand primitives so they cannot be mixed up. Validate once at the boundary. Downstream code trusts the type.

```ts
type AgentId = string & { readonly __brand: "AgentId" };

function isAgentId(input: string): input is AgentId {
  return /^[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$/.test(input);
}

function parseAgentId(input: string): AgentId {
  if (!isAgentId(input)) throw new Error(`Invalid agent id: ${input}`);
  return input;
}

function focusAgent(id: AgentId): void {
  registry.get(id)?.focus();
}
```

Match the `readonly __brand: "X"` shape. Do not invent a second convention.

## Discriminated unions

Model variants with a literal discriminant. Every variant shares the field name and each variant's value is unique, so impossible combinations cannot be represented.

Avoid: a boolean and optional fields allow contradictory states.

```ts
type DiffState = { loading: boolean; diff?: GitDiff; error?: string };
```

Prefer: only valid states exist.

```ts
type DiffState =
  | { kind: "loading" }
  | { kind: "ready"; diff: GitDiff }
  | { kind: "error"; error: string };
```

Pick one discriminant name (`kind`, `type` or `tag`) and keep it across the codebase.

## Constructive modeling

Build the type from parts that are all legal instead of restricting a loose type with runtime checks.

Non-empty, via a variadic tuple:

```ts
type NonEmpty<T> = [T, ...T[]];
```

Avoid: `T[]` plus a length check that every caller must repeat.

```ts
function pickWinner(entries: string[]): string {
  if (entries.length === 0) throw new Error("no entries");
  return entries[Math.floor(Math.random() * entries.length)];
}
```

Prefer: an empty value of the type cannot exist.

```ts
function pickWinner(entries: NonEmpty<string>): string {
  return entries[Math.floor(Math.random() * entries.length)];
}
```

Where a plain `T[]` arrives, narrow once with a guard. The fact then travels in the type.

```ts
const isNonEmpty = <T>(arr: T[]): arr is NonEmpty<T> => arr.length > 0;
```

Even length, as pairs:

```ts
type Pairs<T> = [T, T][];
```

A time range, as start plus duration.

Avoid: the invariant `start <= end` lives only in the reader's head.

```ts
type TimeRange = { start: Date; end: Date };
```

Prefer: a negative range cannot be written, and `end` is derived when needed.

```ts
type TimeRange = { start: Date; durationMs: number };

const rangeEnd = (range: TimeRange): Date =>
  new Date(range.start.getTime() + range.durationMs);
```

Keep `durationMs` a plain number. Brand it only if a raw number could be passed where a duration is expected. Pick the representation that makes the bad state unconstructable, then expose the reading you need on top, such as `pairs.flat()` or `rangeEnd()`.

## Simplest total type

Do not strengthen everything. Keep `T[]` when every operation on it is total.

```ts
const sum = (xs: number[]) => xs.reduce((a, b) => a + b, 0);
```

An empty array sums to 0, so the loose type is correct.

Strengthen when the loose type forces a lie at a use site. The tells are `!`, `arr[0] as T` and a "should never happen" throw.

Avoid: partiality smuggled past the compiler with a non-null assertion.

```ts
function newestSession(sessions: Session[]): Session {
  return sessions.at(0)!;
}
```

Prefer: strengthen the input and the assertion disappears.

```ts
function newestSession(sessions: NonEmpty<Session>): Session {
  return sessions[0];
}
```

Weakening the result to `Session | undefined` is the other total signature.

## `unknown` over `any`

External data is always `unknown`. Narrow before use.

Avoid:

```ts
function handle(input: any) {
  return input.foo.bar;
}
```

Prefer:

```ts
function handle(input: unknown) {
  if (typeof input === "object" && input !== null && "foo" in input) {
    return input.foo;
  }
  return undefined;
}
```

External sources include RPC payloads, `JSON.parse`, `postMessage`, IPC, file contents, environment variables and database results.

## Schemas before hand-rolled guards

Before writing a property-by-property type guard for external data, look for the repository's runtime schema library and existing schemas. Let one schema own validation and derive the TypeScript type from it. Do not maintain a schema, a duplicate interface and a guard that can drift apart.

```ts
import { z } from "zod";

const UserSchema = z.object({
  id: z.string().uuid(),
  role: z.enum(["admin", "member"]),
});

type User = z.infer<typeof UserSchema>;

function parseUser(input: unknown): User {
  return UserSchema.parse(input);
}
```

Use `safeParse` when failure is an expected branch. Use the equivalent inference helper when the repository uses another schema library. Do not add a new schema dependency for one guard. This rule prefers the schema system the codebase already trusts.

## No `as` casts

Every `as` is a potential runtime crash. The compiler must verify the claim, so parse or narrow instead.

Avoid: an `as` cast asserts a shape nothing checked.

```ts
const user = data as User;
```

Avoid: a guard that checks one property and claims the whole type.

```ts
function isUser(data: unknown): data is User {
  return typeof data === "object" && data !== null && "id" in data;
}
```

Prefer: one schema validates and owns the type.

```ts
import { z } from "zod";

const userSchema = z.object({ id: z.string(), name: z.string() });
type User = z.infer<typeof userSchema>;

function parseUser(data: unknown): User {
  return userSchema.parse(data);
}
```

When the type comes first, annotate the validator with the type it proves. The compiler then rejects a validator that proves less than the type. Remove `name` from the object below and the assignment fails to compile.

```ts
import { z } from "zod";

type User = { id: string; name: string };

const userSchema: z.ZodType<User> = z.object({ id: z.string(), name: z.string() });
```

When removing an `as` from existing code, find why TypeScript cannot infer the type:

- Missing discriminant: add one and switch to a discriminated union.
- Overly wide source type, such as `Record<string, unknown>`: narrow it.
- Untyped boundary: parse with the schema that owns the shape. Add a schema only where none exists.
- Genuinely inexpressible: use a branded type from a verifying guard, or `satisfies`.

## Narrowing hierarchy

From best to last resort:

1. **Discriminated union switch or `if`.** The compiler narrows automatically.
2. **`in` operator.** `"key" in obj` narrows to the variants containing that key.
3. **`typeof` or `instanceof`.** For primitives and class instances.
4. **User-defined type guard.** When the above are not enough.

```ts
type Shape =
  | { kind: "circle"; radius: number }
  | { kind: "rect"; width: number; height: number };

function area(s: Shape): number {
  if ("radius" in s) return Math.PI * s.radius ** 2;
  return s.width * s.height;
}
```

## Type guards

A guard must actually verify the claim. A lying guard is worse than `as`.

```ts
function isCircle(s: Shape): s is Shape & { kind: "circle" } {
  return s.kind === "circle";
}
```

Prefer discriminant narrowing when it is available.

## Exhaustiveness

In default arms, assign the discriminant to a `never`-typed local. These examples use the `Shape` union from the narrowing section.

Value-returning switch:

```ts
function area(s: Shape): number {
  switch (s.kind) {
    case "circle":
      return Math.PI * s.radius ** 2;
    case "rect":
      return s.width * s.height;
    default: {
      const _exhaustive: never = s;
      return _exhaustive;
    }
  }
}
```

Statement switch:

```ts
function draw(s: Shape): void {
  switch (s.kind) {
    case "circle":
      drawCircle(s);
      break;
    case "rect":
      drawRect(s);
      break;
    default: {
      const _exhaustive: never = s;
      void _exhaustive;
    }
  }
}
```

Use the return style in value-returning switches and the void style in statement switches.

## `satisfies` over `as`

`satisfies` validates a value without widening its literal types.

Avoid: the cast widens the value and loses the literal types.

```ts
const config = { theme: "dark", cols: 3 } as Config;
```

Prefer: the value is checked against `Config` and `config.theme` stays the literal `"dark"`.

```ts
const config = { theme: "dark", cols: 3 } satisfies Config;
```

## Boundary validation

Validate once where data crosses in. Trust types inside. The `simple-code` skill says where boundaries are.

- **Wire formats** (protobuf, JSON-RPC): parse with `ignoreUnknownFields` so forward-compatible messages do not break old clients.
- **Persisted JSON:** store a versioned blob and handle a failed parse at the read.
- **Inside the system:** do not re-validate deep in call chains.

```ts
type Settings = { version: 1; theme: "light" | "dark" };

function readSettings(raw: string): Settings | undefined {
  try {
    const parsed = SettingsSchema.safeParse(JSON.parse(raw));
    return parsed.success ? parsed.data : undefined;
  } catch {
    return undefined;
  }
}
```

## Schema-derived types

When a `.proto`, OpenAPI spec, GraphQL schema or database migration already defines a shape, derive from the generated types instead of duplicating them.

Avoid: a duplicate shape that drifts when the schema changes.

```ts
type CheckSummary = {
  totalCount: number;
  checks: { name: string; status: string }[];
};

function renderChecks(s: CheckSummary) {
  return s.checks.map((check) => check.name);
}
```

Prefer: derive from the generated schema type.

```ts
import type { ChecksMessage } from "./generated/checks";

function renderChecks(s: Pick<ChecksMessage, "totalCount" | "checks">) {
  return s.checks.map((check) => check.name);
}
```

Reach for `Pick`, `Omit`, `Parameters`, `ReturnType`, `Awaited` and `typeof` before writing a new interface.

## Object args

Avoid: swap two positional arguments and it still compiles.

```ts
openFile(uri, 10, 1, 10, 1);
```

Prefer: order-independent and self-documenting.

```ts
openFile({
  uri,
  selection: {
    startLineNumber: 10,
    startColumn: 1,
    endLineNumber: 10,
    endColumn: 1,
  },
});
```

Skip this on hot paths such as per-frame render, tokenizers, parsers and anything in a tight loop where the allocation cost matters.

## Real tests

Run the real dependency instead of a hand-written mock when it can run locally. Use the test framework's own primitives and its disposal checks.

```ts
import { afterEach, expect, test } from "vitest";

const stores: Store[] = [];

afterEach(async () => {
  await Promise.all(stores.splice(0).map((store) => store.close()));
});

test("a saved note is readable after reopening the store", async () => {
  const path = await makeTempPath();
  const first = await openStore(path);
  stores.push(first);
  await first.save({ id: "n1", body: "hello" });
  await first.close();

  const second = await openStore(path);
  stores.push(second);

  expect(await second.get("n1")).toEqual({ id: "n1", body: "hello" });
});
```

The test opens a real store on a temporary path and asserts observable behavior. The `behavioral-testing` skill covers where to draw test boundaries.

## Structured telemetry

Log structured fields with enough context to debug from an id. Do not call `console.log` in shipped code.

Avoid:

```ts
console.log("failed to sync", err);
```

Prefer:

```ts
logger.error("sync failed", { syncId, workspaceId, err });
```
