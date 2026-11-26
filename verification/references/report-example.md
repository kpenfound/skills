# report.json example

A version 1 `report.json` for a lever run, returned in the Directory alongside the evidence files it names. The flow names, claim text and paths are examples; the product lever defines its own.

```json
{
  "version": 1,
  "claims": [
    {
      "name": "checkout completes with a valid card",
      "status": "pass",
      "detail": "Order confirmation page showed order number and total.",
      "evidence": ["evidence/checkout-confirmation.png"],
      "flow": "checkout"
    },
    {
      "name": "declined card shows an error and keeps the cart",
      "status": "fail",
      "detail": "Page showed the confirmation instead of the decline message.",
      "evidence": ["evidence/checkout-declined.png", "evidence/checkout.log"],
      "flow": "checkout"
    },
    {
      "name": "cart survives a reload",
      "status": "inconclusive",
      "detail": "The reload step timed out before the page rendered.",
      "evidence": ["evidence/reload.log"]
    }
  ]
}
```

Reading it:

- Per flow, any `fail` makes the flow `fail`, else any `inconclusive` makes it `inconclusive`, else it passes. Here `checkout` is `fail`.
- The third claim carries no `flow`, so it counts toward the run's status but covers no flow. The run is `fail`.
- Every `evidence` path is relative to the returned Directory and the file exists. A path that is absent or escapes the Directory makes the whole run `inconclusive`.
- `inconclusive` is never a pass. The cart-reload behavior is unknown, not working.
