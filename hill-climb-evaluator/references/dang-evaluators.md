# Dang evaluators

Longer examples for the `hill-climb-evaluator` skill. The contract is in `SKILL.md`: one caller argument `run: String!`, an optional `ws: Workspace!`, and one JSON string with `schema` 1. The examples use Dang's built-in `JSON` helpers, which need no import. Treat the container commands as placeholders for the project's own workload.

## Cached setup, a guardrail and a protected helper

Layout assumed by the example:

- `src/` is the code being climbed. It is editable.
- `evaluator/measure.sh` is the scoring helper. It runs the workload and prints the elapsed milliseconds and the peak memory in kilobytes, one per line. It is protected.
- `evaluator/fixtures/input.txt` and `evaluator/expected/output.txt` are the fixture and the oracle. They are declared and protected.

```dang
type OsmiaDev {
  """
  Build once, measure the workload, check the output. Returns the evaluator JSON.
  """
  pub evaluateWorkload(run: String!, ws: Workspace!): String! {
    # Deterministic setup. It does not depend on run, so the engine caches it.
    let built = container
      .from("golang:1.24-alpine")
      .withDirectory("/work", ws.directory("/", include: ["src/**", "go.mod", "evaluator/**"]))
      .withWorkdir("/work")
      .withExec(["go", "build", "-o", "/bin/tool", "./src"])

    # The measured exec. run changes its cache key on every call.
    let measured = built
      .withEnvVariable("RUN", run)
      .withExec(["sh", "evaluator/measure.sh", "/bin/tool", "evaluator/fixtures/input.txt", "/tmp/out.txt", "/tmp/stats"])

    let stats = measured.file("/tmp/stats").contents.trimSpace.split("\n")
    let elapsedMs = stats[0]!.toInt
    let peakKb = stats[1]!.toInt

    let actual = measured.file("/tmp/out.txt").contents
    let expected = built.file("evaluator/expected/output.txt").contents
    let ok = actual == expected

    let reason = if (ok) {
      "output matches evaluator/expected/output.txt"
    } else {
      "output differs from evaluator/expected/output.txt"
    }

    JSON.encode({{
      schema: 1,
      valid: ok,
      reason: reason,
      metrics: [
        "wall_ms": {{ value: elapsedMs, unit: "ms", direction: "lower" }},
        "peak_rss_mb": {{ value: peakKb / 1024, unit: "MB", direction: "lower" }}
      ]
    }})
  }
}
```

Points the example shows:

- The build step has no `run`, so it is cached between measurements and does not add noise. Only the measured exec reruns.
- The scoring helper and the oracle sit under `evaluator/`, inside the protected paths, and the code under measurement sits under `src/`, outside them.
- The correctness verdict comes from comparing with the protected oracle, so a faster but wrong build is `valid: false`.
- A second metric, `peak_rss_mb`, gives the climb spec something to hold as a guardrail while the primary metric climbs.

## Reading a result back

A reviewer or a test can decode an evaluator's output into a type. Decoding is type-driven, so declare the shape and ask for it.

```dang
type Metric {
  value: Float!
  unit: String!
  direction: String!
}

type Result {
  schema: Int!
  valid: Boolean!
  reason: String!
  metrics: Map[Metric!]!
}

let result = JSON.decode(text, strict: true) :: Result!
```

`strict: true` rejects any key the type does not declare, which catches a misspelled field. Check `result.schema == 1` and that every `direction` is `"higher"` or `"lower"` before trusting the output.

## An invalid result

The same document with `valid` false for a candidate whose output is wrong:

```json
{
  "schema": 1,
  "valid": false,
  "reason": "output differs from evaluator/expected/output.txt",
  "metrics": {
    "wall_ms": { "value": 3.2, "unit": "ms", "direction": "lower" },
    "peak_rss_mb": { "value": 12, "unit": "MB", "direction": "lower" }
  }
}
```

On a climbing attempt, the service discards this candidate. During calibration, which sets the baseline, the same output means the metric is broken, and the climb holds.
