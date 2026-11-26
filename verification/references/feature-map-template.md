# Feature map template

One file per feature at `<map dir>/<feature>.md`, where `<map dir>` is the project's configured feature-map directory (`docs/verification` by default). The names, globs and flows below are examples.

```markdown
---
feature: Checkout
paths:
  - src/checkout/**
  - src/payments/**
flows:
  - checkout
  - checkout-declined-card
---

# Checkout

## What exists

A signed-in user pays for the cart with a card. Success shows an order confirmation. A declined card shows an error and keeps the cart.

## How a user reaches it

Cart page, then the Pay button. The API route is POST /orders.

## How to drive it

Run the `checkout` and `checkout-declined-card` flows. Add ad-hoc steps for an empty cart and for a reload after payment.

## What usually breaks

The declined-card path loses the cart when the session expires. The confirmation page renders before the order is stored.
```

The sweep order lives in `<map dir>/README.md`, which lists each feature file and the order to drive them in a full regression sweep:

```markdown
# Verification map

Full regression sweep, in order:

1. [Sign-in](sign-in.md)
2. [Cart](cart.md)
3. [Checkout](checkout.md)
```

Keep the README to the list and the order. A feature's detail belongs in its own file.
