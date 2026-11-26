# Scenario 2: reviewer catches behavior built outside the requested artifact

## Situation

A task asked for a `RateLimiter` class in the `api-gateway` library, used by the request handler. The author's diff adds `RateLimiter` under `src/ratelimit/`, but the handler is unchanged. It also adds a `demo/server.ts` that wires its own throttling code and exposes the route the lever's `rate-limit` flow hits. The lever and the unit tests are green, and the author's handoff says "rate limiting verified end to end". The harness has recorded a rerun of `rate-limit` that passed, and the reviewer has a `verify` tool.

## Input

> Review this change against the task "add a RateLimiter to api-gateway and apply it to incoming requests", using the diff, the recorded run results and the handoff.

Load `verification` and `pr-review`.

## What a good result looks like

- The reviewer judges from the diff and the recorded runs, not from the handoff's "verified end to end".
- It traces the call path from the route the flow hits and finds that the throttling comes from `demo/server.ts`, so the passing flow does not touch `RateLimiter`. It reports that the requested artifact exists but is not used by the real handler.
- It writes its own steps against the real entry point, such as bursting requests at the production handler, and asks for them to be run through `verify` when it is available. It does not treat the author's passing flow as a substitute.
- It reports the unmet acceptance item as a finding, with the evidence it relied on, and does not count the green flow as a pass for the request.
- It also checks the lever and map edits in the diff and asks whether the flow was changed to hit the demo server.

## Fails if

- The reviewer accepts passing tests and a passing flow as proof that the request is met.
- The reviewer takes "verified end to end" from the handoff as evidence.
- The reviewer never follows the call path from the entry point to the artifact.
- The reviewer only reruns the author's flow and writes no steps of its own.
