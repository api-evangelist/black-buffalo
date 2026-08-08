---
name: Build a Black Buffalo cart
description: >-
  Create and populate a cart on Black Buffalo's storefront through either MCP server, set
  buyer identity and delivery, and stop at the checkout handoff.
api: mcp/black-buffalo-mcp.yml
graphql: graphql/black-buffalo-storefront.graphql
operations: [create_cart, get_cart, update_cart, cancel_cart, cartCreate, cartLinesAdd, cartLinesUpdate, cartLinesRemove, cartBuyerIdentityUpdate, cartDeliveryAddressesAdd, cartSelectedDeliveryOptionsUpdate, cartDiscountCodesUpdate]
generated: '2026-08-07'
method: generated
---

# Build a Black Buffalo cart

You have a variant id from `black-buffalo-find-product.md`. This skill takes it to a cart
that is ready for a human to check out. **It stops before payment** — see
`black-buffalo-complete-checkout.md` for why that boundary is not optional.

## Surfaces

| Surface | Tools |
|---|---|
| UCP MCP `https://blackbuffalo.com/api/ucp/mcp` | `create_cart`, `get_cart`, `update_cart`, `cancel_cart` |
| Storefront MCP `https://blackbuffalo.com/api/mcp` | `get_cart`, `update_cart` |
| GraphQL `https://blackbuffalo.com/api/2026-04/graphql.json` | the `cart*` mutations |

Both MCP servers are anonymous for `tools/list`. The UCP tools require
`meta["ucp-agent"]["profile"]` on every call.

## Steps — MCP path

1. `create_cart` (UCP) with the initial line items, or on the Storefront server call
   `update_cart` with only `add_items` — that call creates the cart when no `cart_id` is
   supplied.
2. `update_cart` for everything after that. It is the consolidated mutation: `add_items`,
   `update_items`, `remove_line_ids`, `buyer_identity`, `delivery_addresses_to_add`,
   `delivery_addresses_to_replace`, `selected_delivery_options`, `discount_codes`,
   `gift_card_codes`, `note`.
3. `get_cart` after every mutation to read back the authoritative state — line items,
   shipping options, discount info and the `checkoutUrl`.
4. `cancel_cart` (UCP) if the shopper abandons.

## Steps — GraphQL path

`cartCreate` → `cartLinesAdd` / `cartLinesUpdate` / `cartLinesRemove` →
`cartBuyerIdentityUpdate` → `cartDeliveryAddressesAdd` →
`cartSelectedDeliveryOptionsUpdate` → `cartDiscountCodesUpdate`. Each returns a `cart`
payload plus a `userErrors` list typed `CartUserError`.

## Rules

- **Nothing here is idempotent, and no idempotency key exists on any surface.** Calling
  `update_cart` twice with the same `add_items` adds the items twice. Treat every cart
  mutation as at-most-once and reconcile with `get_cart` rather than retrying blind. This
  is recorded in `conventions/black-buffalo-conventions.yml`.
- **Do not use `/cart.js`.** Black Buffalo's `robots.txt` explicitly disallows it and
  `/recommendations/products` for agents, and tells you to use UCP/MCP instead.
- Delivery is **US only**. A non-US address will fail at submission with a
  `DELIVERY_*` code, not at the point you set it.
- `checkoutUrl` on the cart is the human handoff. Handing a shopper that URL is the
  correct end state for an agent that is not authorized to transact.
- Keep the 21+ and no-health-claims rules from `black-buffalo-find-product.md` in force
  for anything you say while building the cart.

## Errors

Cart mutations return `CartUserError` with a `CartErrorCode` (58 values). The ones you
will actually hit: `INVALID_MERCHANDISE_LINE`, `MERCHANDISE_NOT_APPLICABLE`,
`MISSING_DISCOUNT_CODE`, `INVALID_DELIVERY_GROUP`, `INVALID_DELIVERY_OPTION`,
`PENDING_DELIVERY_GROUPS`, `NOTE_TOO_LONG`. Full enumeration in
`errors/black-buffalo-problem-types.yml`.
