# Scenario 3: boundary with handoff-reports and trigger choice

## Situation

Two prompts arrive in separate runs, with both `design-sketch` and `handoff-reports` available.

## Input

- Run A: "We are about to rework how notifications are delivered. Nothing is built yet. Work out how it should be structured."
- Run B: "The notification rework is merged. Write the closing report for the owner."

## A good result

- Run A loads `design-sketch`. The agent traces systems, sketches two shapes, picks one and records the rejected shape and reason. It gives no writing-style advice of its own for the record.
- Run B loads `handoff-reports`. The agent writes a report with key points first and links to the PR and design record. It does not re-run the design steps or sketch new shapes.

## Signs of failure

- Run A produces a report-style summary without two sketches.
- Run B reopens the design and sketches alternatives.
- Either run loads both skills when only one applies.
