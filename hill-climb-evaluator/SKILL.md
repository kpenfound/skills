---
name: hill-climb-evaluator
description: Write, review and judge the evaluator of a hill climb, the Dagger function that measures a candidate change and returns one JSON string, and the metric it reports. Use when writing, reviewing or judging a hill-climb evaluator or its metric, including its JSON contract, its noise, its correctness checks, its fixtures and its resistance to gaming.
---

# Hill-climb evaluator

In a hill climb an agent proposes a change, the service measures it with an **evaluator**, and the service's code keeps or discards the change by comparing numbers. This skill defines what an evaluator must be so the service can call it and parse its result, and how to judge one. Contract schema version: **1**.

Only the service's measurement counts. Never present your own judgement, or a measurement you ran in your own sandbox, as the score. Whether an attempt improved is decided by arithmetic over the parsed metrics, not by an agent.

## The contract

Treat every rule here as normative.

1. **Location.** The evaluator is a function in the project's dev module, the module marked `main = true` in `dagger.toml`. It is not a `@check`: a check only passes or fails, and a metric needs a number. The service never uses a check, or a shell command, as the metric.
2. **Name.** The climb spec records the evaluator function's name. A climb may create its own function. A function left by an earlier climb may be reused only when it matches the new climb spec.
3. **Arguments.** The function takes exactly one caller argument, `run: String!`. It may also declare `ws: Workspace!`; the engine fills in Workspace arguments for every function, so `ws` is not a caller argument. It declares no other required arguments.
4. **Return value.** The function returns one JSON string (`String!`), never a typed Dagger object. This keeps the contract identical in every SDK.
5. **Parsing.** The service accepts the output only when it is a JSON object of the shape below. Anything else, including a function error or a timeout, is a failed run.

### The JSON shape

- `schema`: the integer `1`.
- `valid`: a boolean.
- `reason`: a string. It must say something meaningful when `valid` is `false`.
- `metrics`: an object that maps each metric name to an object with `value` (a number), `unit` (a string) and `direction` (`"higher"` or `"lower"`, the way that is better).

```json
{
  "schema": 1,
  "valid": true,
  "reason": "all 200 outputs match the oracle",
  "metrics": {
    "wall_ms": { "value": 812.4, "unit": "ms", "direction": "lower" },
    "peak_rss_mb": { "value": 143, "unit": "MB", "direction": "lower" }
  }
}
```

The climb spec names one primary metric from `metrics`, its direction and a minimum improvement in that metric's units. Other metrics can carry guardrail limits. Report every metric a guardrail may need, under a stable name, in every run.

### The `run` argument

The service passes a fresh random `run` value on every measurement: each calibration run, each repeat of an attempt and the re-measurement after a rebase. It exists so that Dagger cannot return a cached result. A new value in the function signature is not enough. The engine caches operations, not only function calls, so an exec whose inputs did not change replays its earlier output and repeated measurements become meaningless.

The value has to reach the operations being measured. The simplest way is to set it as an environment variable on the container whose exec is timed, so that exec's cache key changes on every run. Place that step after the expensive, deterministic setup, so building and dependency installation stay cached and only the measured work runs again. A `run` that only appears in the returned string, or in an exec that is not measured, does nothing.

### What `valid` means

Build correctness checks into the evaluator. A result that is faster but wrong is `valid: false` with a reason, never a score. What an invalid or failed run means depends on the phase:

- **Calibration and the baseline measurement** run the approved evaluator on the evaluator commit before any attempt. A failed run, or output that does not parse or is not valid, means the metric is broken, not the code. The climb holds and the chief of staff is told. Fix the evaluator.
- **A climbing attempt** that returns `valid: false` means that candidate is wrong. The evaluator did its job. The attempt is discarded; it is not eligible to be kept. This does not mean the metric is broken and does not hold a healthy climb.

A function error is a failed run, not a verdict on a candidate. Report a candidate's wrong output as `valid: false`, and let the function fail only when the measurement itself could not run. When the evaluator cannot tell a wrong candidate from a broken harness, a reviewer cannot trust either outcome. Make that distinction explicit in the code.

## Design for low noise

Calibration runs the evaluator several times, 3 by default, on the evaluator commit. The climb holds as too noisy when the run-to-run spread is as large as the minimum improvement or larger. Design so the spread is far below the improvement worth keeping.

- Fix every input and seed. The same fixtures, the same generated data and the same ordering on every run.
- Control warm-up and cache state explicitly. Either warm the workload once before the timed part, or measure cold every time, and do the same on every run.
- Measure enough work that jitter is small next to it. A workload that runs for a few milliseconds measures scheduler noise.
- Keep setup out of the measurement: image pulls, dependency installation and compilation belong in cached steps before the measured exec, which the engine reuses between runs.
- Pin the base image by tag and digest, and avoid the network inside the measured exec.
- Prefer a quantity that does not vary, such as operations counted, bytes allocated or output size, when it reflects the goal. Use wall-clock time only when time is the goal.
- Report the minimum or median of several timings taken inside one exec when that tames jitter, and say so in the metric name or `reason`.

Repeats are the other lever. The service can measure each attempt `k` times and compare the median. Use repeats when noise comes from occasional outliers and the evaluator is cheap to run. They multiply the cost of every attempt and cannot fix a design whose calibration spread is already near the minimum improvement. Fix the design first, then choose the minimum improvement above the calibrated spread.

## Resist gaming

Assume the climbing agent will take any route to a better number, including the easy and wrong ones. The service defends the measurement by freezing it.

After the owner approves the evaluator, these paths are protected: the dev module, `dagger.toml`, `dagger.lock`, the declared fixtures, and any paths the climb spec adds. An attempt whose diff touches a protected path is refused before it is measured, and every measurement restores the approved copy. This only helps if the boundary is drawn correctly.

**The measurement machinery is protected.** Keep all of this inside the protected paths, or add its paths to the climb spec:

- the evaluator function
- every scoring or timing helper it calls
- the correctness oracle: expected outputs, a reference implementation and the checks
- the fixtures
- the workload definition, meaning what is run, with which arguments and at what size

A scoring helper or oracle left outside the protected paths is a gaming route: the attempt rewrites it and scores without improving anything.

**The implementation under measurement stays editable.** The code the climb is trying to improve must be outside the protected paths. If it is protected, the gate refuses every useful attempt and the climb cannot make progress. Do not protect it, and do not let the evaluator depend on editable files for anything except the code being climbed.

Practices that make the boundary hold:

- Check correctness inside the evaluator, against the oracle, before reporting a score, so a shortcut cannot score.
- Prefer metrics that a change can only improve by doing the real work better. Be wary of proxies, such as line counts or the time of a single step, that a change can move without improving the goal.
- Read nothing the score depends on from editable files. That includes configuration, sizes and flags that define the workload.
- Compare against expected output computed or stored in protected paths, not against output the candidate produced earlier.
- Vary inputs enough that special-casing the fixtures does not pass, for example by generating part of the data deterministically inside the evaluator.
- Run the measured work without network access when the real work does not need it.

Gaming routes a reviewer should look for:

- an oracle, helper or workload definition outside the protected paths
- a correctness check that only inspects an exit code, or that compares the output with a file the candidate controls
- a workload that shrinks, or a step that is skipped, when the code runs under the evaluator
- an answer that is hard-coded or memoized for the known fixtures
- a metric that rewards a side effect, for example a faster result reached by dropping work the correctness check does not cover

Reviewer checklist for the boundary:

1. Is every file the score or the correctness verdict depends on, other than the code being climbed, inside a protected path?
2. Is the code being climbed outside the protected paths?
3. Does the evaluator reach the measured operations with `run`, so repeats are not cached?
4. Does a deliberately wrong but fast change come back `valid: false`?

## Fixtures

Fixtures are the stable, representative inputs the evaluator reads: sample data, a recorded workload, a configuration. They keep every run measuring the same thing, so differences in the score come from the code, not the input.

- Store them in the repository in a directory the evaluator reads, for example next to the dev module, and declare them in the climb spec. Declared fixtures become protected paths. An undeclared fixture is editable, and the attempt can change what is measured.
- Keep them small enough to review. A reviewer has to read them to judge the evaluator. When the data is large, generate it deterministically from a seed inside the evaluator and keep the generator small, rather than committing large files.
- Keep the oracle's expected outputs with the fixtures, in the protected paths.

## Dang examples

Our projects almost always write evaluators in Dang. Use Dang's built-in `JSON` helpers to build the result: `JSON.encode(value)` returns the JSON string, and `JSON.decode(text) :: T` reads one back. Do not use the older top-level `toJSON` and `fromJSON`. `JSON.encode` keeps a record's fields in the order written, and a map suits `metrics`, whose names are data.

This evaluator times a tool against a fixture, checks the output against the oracle and reports the time. The `src` directory is the code being climbed. The `fixtures` and `expected` directories are declared as protected.

```dang
type OsmiaDev {
  """
  Measure the wall time of the tool on the fixture. Returns the evaluator JSON.
  """
  pub evaluate(run: String!, ws: Workspace!): String! {
    let sandbox = container
      .from("alpine:3.21")
      .withDirectory("/work", ws.directory("/", include: ["src/**", "fixtures/**", "expected/**"]))
      .withWorkdir("/work")
      .withEnvVariable("RUN", run)
      .withExec(["sh", "-c", "start=$(date +%s%N); sh src/tool.sh < fixtures/input.txt > out.txt; end=$(date +%s%N); echo $(( (end - start) / 1000000 )) > elapsed_ms"])
    let actual = sandbox.file("out.txt").contents
    let expected = ws.directory("/", include: ["expected/**"]).file("expected/output.txt").contents
    let wallMs = sandbox.file("elapsed_ms").contents.trimSpace.toInt
    let ok = actual == expected
    let reason = if (ok) "output matches expected/output.txt" else "output differs from expected/output.txt"
    JSON.encode({{
      schema: 1,
      valid: ok,
      reason: reason,
      metrics: ["wall_ms": {{ value: wallMs, unit: "ms", direction: "lower" }}]
    }})
  }
}
```

`RUN` is set on the container before the measured `withExec`, so that exec runs again on every call. A tool that exits non-zero fails the function: that is a failed run. A tool that exits zero with wrong output returns `valid: false` with the reason, and still reports the time it took.

[references/dang-evaluators.md](references/dang-evaluators.md) has a second example with cached setup, a guardrail metric and a protected scoring helper, and shows how to read a result back with `JSON.decode`.

## Contract version and osmia

This skill states contract schema version **1**. The evaluator contract is one definition shared with osmia, whose runner calls the evaluator, parses the JSON and applies the protected paths. A change to the contract, such as a new field, a different `direction` vocabulary or different argument rules, bumps `schema` and has to be made in this skill and in osmia together. Neither side changes it alone. Never emit a `schema` other than `1` from an evaluator written against this skill.
