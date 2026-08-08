---
name: Find and compare Black Buffalo products
description: >-
  Search Black Buffalo's catalog and resolve a specific purchasable variant, using either
  of the two anonymous MCP servers or the Storefront GraphQL API — while respecting the
  provider's published restrictions on how a regulated nicotine product may be described.
api: mcp/black-buffalo-mcp.yml
graphql: graphql/black-buffalo-storefront.graphql
operations: [search_catalog, lookup_catalog, get_product, get_product_details, search, products, predictiveSearch, product, productByHandle]
generated: '2026-08-07'
method: generated
---

# Find and compare Black Buffalo products

Black Buffalo sells a small catalog across three lines — nicotine pouches, long cut dip,
and the nicotine-free **ZERO** line — in a handful of flavors (Wintergreen, Mint,
Straight, Blood Orange, Peach) and two pack sizes (single can, 5-can roll). Your job is to
get from a shopper's intent to one concrete, in-stock **variant id**.

## Read this first

These are age-restricted products. Black Buffalo's own `/agents.md` sets hard rules on
what you may say about them, and they bind you before any of the mechanics below. See
`agentic-access/black-buffalo-agentic-access.yml` for the full list. The short version:

- **21+ only.** Never recommend or describe these products as suitable for anyone under 21.
- **No health claims.** Not "safer", not "healthier", not "less harmful".
- **Not a cessation product.** Never use "quit", "quitting" or "stop smoking".
- **Do not call the nicotine products "tobacco-free."** They contain no tobacco leaf or
  stem but are regulated as tobacco products. Only the **ZERO** line is tobacco-free and
  nicotine-free.
- **No comparative superiority claims** against named competitors.
- Nicotine is an addictive chemical, and that is the framing the provider requires.

## Which surface to use

All three are anonymous — no key, no token.

| Need | Surface |
|---|---|
| Natural-language intent ("wintergreen pouches, no nicotine") | MCP `search_catalog` on either server |
| Resolve several product/variant ids at once | MCP `lookup_catalog` at `/api/ucp/mcp` |
| Full product detail with live availability | MCP `get_product` at `/api/ucp/mcp` |
| Exact, structured queries and full field control | GraphQL at `https://blackbuffalo.com/api/2026-04/graphql.json` |

Two MCP endpoints serve overlapping catalog tools:

- `https://blackbuffalo.com/api/ucp/mcp` — 13 tools, the full commerce lifecycle. Every
  tool requires `meta["ucp-agent"]["profile"]`, a resolvable agent profile URI.
- `https://blackbuffalo.com/api/mcp` — 5 tools, catalog/cart/policy only. `get_product_details`
  here takes plain `product_id`, `options`, `country`, `language`.

## Steps — MCP path

1. Call `search_catalog`. Supply a natural-language `query`, structured `filters`, or
   both — the tool requires at least one. On the UCP server you must also pass the `meta`
   object with your agent profile URI.
2. Results are paginated with a deliberately small first page. Read `pagination.cursor`
   from the response and page forward rather than asking for a huge page.
3. Resolve the exact item:
   - `get_product` (UCP) with the identifier, using `selected` and `preferences` to narrow
     to a variant and get real-time availability; or
   - `get_product_details` (Storefront) with `product_id` plus `options`. **Without
     `options` you get the first available variant**, which is frequently not the flavor
     or pack size the shopper asked for.
4. Use `lookup_catalog` when you are holding several ids — it resolves
   `gid://shopify/Product/...` and `gid://shopify/ProductVariant/...` in one request and
   groups variants under their parent product.

## Steps — GraphQL path

1. `products(first:, query:, sortKey:)` or `search(query:, types: [PRODUCT], productFilters:)`
   to list candidates. Select
   `id handle title productType tags availableForSale priceRange { minVariantPrice { amount currencyCode } } requiresSellingPlan`.
2. `product(id:)` or `productByHandle(handle:)` for detail. Select
   `variants(first: 25) { nodes { id title sku availableForSale quantityAvailable price { amount currencyCode } selectedOptions { name value } } }`.
3. `predictiveSearch` covers the typeahead case.
4. There is no MCP tool that lists collections. If you need to enumerate the
   `nicotine-pouches`, `long-cut` or `zero` collections, you must use GraphQL
   `collections` / `collectionByHandle` — this is a real gap in the tool surface, recorded
   in `mcp/black-buffalo-tool-crosswalk.yml`.

## Rules

- **Check `availableForSale` and `quantityAvailable` before you promise anything.**
- **Never quote a price you did not read from a response.** `/agents.md` publishes a list
  price, but it also says retail pricing varies by store, location, taxes and promotions,
  and that everything is subject to change.
- The store ships to **US only** (`shipsToCountries: ["US"]`) and prices in **USD**.
- `requiresSellingPlan: true` means the item cannot be bought one-time; it must go on a
  subscription selling plan.
- For "where can I buy this near me", do not guess. See
  `black-buffalo-answer-store-and-policy.md` — and note the locator sitemap the provider
  points you at currently 404s.

## Errors

GraphQL returns HTTP 200 with an `errors[]` array; check it before reading `data`. Every
response carries `extensions.cost` — the API is query-cost throttled, not header rate
limited. MCP errors also arrive inside HTTP 200 as JSON-RPC `error` objects, so inspect
the body, never the status. See `errors/black-buffalo-problem-types.yml`.
