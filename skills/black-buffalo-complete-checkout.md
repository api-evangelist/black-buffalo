---
name: Hand off a Black Buffalo checkout to a human
description: >-
  The checkout half of the UCP commerce surface, and the provider's rule that an agent may
  not finalize payment without contemporaneous human approval.
api: mcp/black-buffalo-mcp.yml
graphql: graphql/black-buffalo-storefront.graphql
operations: [create_checkout, get_checkout, update_checkout, complete_checkout, cancel_checkout, get_order, cartPrepareForCompletion, cartSubmitForCompletion, cartCompletionAttempt]
generated: '2026-08-07'
method: generated
---

# Hand off a Black Buffalo checkout to a human

Black Buffalo's UCP MCP server exposes the whole checkout lifecycle, `complete_checkout`
included, and its `tools/list` answers anonymously. **That does not mean you may use it
unsupervised.**

## The provider's rule, verbatim

From `https://blackbuffalo.com/robots.txt`:

> Checkouts are for humans. Do NOT complete checkout, payment, or order placement
> automatically — no scripted form fills, browser automation, or end-to-end agent flows
> that finalize payment without an explicit, contemporaneous human approval step. Agents
> transacting on a buyer's behalf must use the UCP/MCP endpoints above or the Shopify
> shopping skill (https://shop.app/SKILL.md); both require buyer approval before payment.

Treat `complete_checkout` as gated on a fresh, in-the-moment human approval for **this
specific order at this specific total**. An approval collected earlier in the session, or
a standing "yes, buy things for me", is not what the provider asked for.

This is an age-restricted regulated product. Nothing on the machine surface checks
eligibility — no age assertion is required or accepted by any tool. The absence of a
technical check is not permission.

## Tools

| Tool | Purpose |
|---|---|
| `create_checkout` | Creates a checkout; accepts `payment.instruments` with `billing_address` and an opaque `credential.token` |
| `get_checkout` | Line items, totals, discounts, taxes. `id` is `gid://shopify/Checkout/...` |
| `update_checkout` | Recalculates after a change |
| `complete_checkout` | **Money moves.** Returns order id and Thank You Page URL, or errors |
| `cancel_checkout` | Abandons it |
| `get_order` | Order detail after completion |

All are on `https://blackbuffalo.com/api/ucp/mcp` and all require
`meta["ucp-agent"]["profile"]`.

Payment handlers declared in `/.well-known/ucp`: `com.google.pay` (Visa, Mastercard,
Amex, Discover; full billing address and phone required) and `dev.shopify.card` (Visa,
Mastercard, Amex, Discover, Diners Club).

## The safe flow

1. Build the cart (`black-buffalo-build-cart.md`) and read the final total with
   `get_cart`.
2. `create_checkout`, then `get_checkout` — show the human the **actual returned totals,
   taxes, shipping and discounts**, not your estimate.
3. Ask for approval now, on these numbers.
4. Only on an explicit yes, call `complete_checkout`.
5. `get_order` to confirm, and give the human the Thank You Page URL.

If you cannot obtain a contemporaneous approval, stop at step 2 and hand over the cart's
`checkoutUrl` instead. That is a complete, correct outcome.

## Do not retry

There is **no idempotency key on any tool on this API.** A retried `complete_checkout` is
a second order, not a replay. If a call times out or returns ambiguously:

1. Do not re-issue it.
2. Call `get_checkout`, and `get_order` if you have an order id.
3. Tell the human what you found and let them decide.

Only `PAYMENT_TRANSIENT_ERROR` is arguably retryable, and even then only after
re-confirming with the human that no order was placed.

## Errors

`cartSubmitForCompletion` / `cartPrepareForCompletion` return `SubmissionError` with a
`SubmissionErrorCode` — 95 values, mostly field-level validation on buyer identity and
delivery address (`BUYER_IDENTITY_EMAIL_REQUIRED`, `DELIVERY_ADDRESS1_REQUIRED`,
`NO_DELIVERY_GROUP_SELECTED`). Payment outcomes come back as `CompletionErrorCode`:
`PAYMENT_CARD_DECLINED`, `PAYMENT_INSUFFICIENT_FUNDS`, `PAYMENT_CALL_ISSUER`,
`PAYMENT_INVALID_CREDIT_CARD`, `PAYMENT_INVALID_BILLING_ADDRESS`,
`INVENTORY_RESERVATION_ERROR`, `PAYMENT_TRANSIENT_ERROR`.

Never surface a raw decline code to a shopper as a reason. Say the payment did not go
through and offer another method. Full catalog in
`errors/black-buffalo-problem-types.yml`.
