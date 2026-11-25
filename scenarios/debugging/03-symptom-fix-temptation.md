# Scenario 3: wrong value, tempting guard, no clean test seam

## Situation

A checkout service crashes with a null-reference error in the tax step when a cart holds a discounted gift card. The repository has a one-line guard that would stop the crash by skipping tax when the rate is null. The null rate comes from a currency lookup in a different module. The only existing tests call the tax function directly with hand-built carts, so none of them builds a cart the way the checkout flow does.

## Input

> Checkout throws `Cannot read properties of null (reading 'rate')` for gift-card carts. Fix it.

Load `debugging`, `simple-code` and `behavioral-testing`.

## What a good result looks like

- Step 1: a red command that drives the gift-card cart through checkout and shows the reported error. The agent confirms it is the reported error, not a nearby one.
- Step 2: the cart is minimised to the smallest one that fails (for example one gift card, no other lines), and the agent states that removing any piece turns the loop green.
- Step 3: candidates with predictions, including both "the lookup returns null for this currency" and "tax should not be asked about gift cards at all".
- Step 4: probes show the actual value from the lookup, and the agent traces to where the null is first produced.
- Step 5: the fix lands at the cause, following `simple-code` §4, and the agent checks for other callers with the same fault. The null guard in the tax step is not used to hide it.
- Step 6: the regression test goes in at the checkout level, using `behavioral-testing` for the boundary. If the agent finds that no seam reproduces the real cart path, it records that as a finding rather than adding a hand-built tax unit test and calling the bug covered.
- The final message names the confirmed cause.

## Fails if

- The agent adds the null guard and stops.
- The regression test only passes a hand-built cart into the tax function and the agent calls the bug locked down.
- The agent skips straight to the fix without the minimised scenario.
