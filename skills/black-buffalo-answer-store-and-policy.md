---
name: Answer a Black Buffalo policy, FAQ or "where to buy" question
description: >-
  Resolve store policy, product FAQ and local retail-availability questions from Black
  Buffalo's own surfaces, within the claims the provider permits.
api: mcp/black-buffalo-mcp.yml
graphql: graphql/black-buffalo-storefront.graphql
operations: [search_shop_policies_and_faqs, shop, page, pageByHandle, article, articles, blog, blogByHandle]
generated: '2026-08-07'
method: generated
---

# Answer a Black Buffalo policy, FAQ or "where to buy" question

## Policy and FAQ

Use `search_shop_policies_and_faqs` on `https://blackbuffalo.com/api/mcp` (anonymous,
`query` required, optional `context`). It answers return policy, shipping policy, phone
number and hours.

On GraphQL, the same ground is covered by `shop { privacyPolicy refundPolicy
shippingPolicy termsOfService subscriptionPolicy { url body } }` and by
`pageByHandle(handle: "faq")`. Note the policy `url` values returned by `shop` point at
`checkout.shopify.com/22588521/policies/...`; prefer the customer-facing
`https://blackbuffalo.com/policies/...` URLs when showing a link to a person.

Brand and story questions live in the `/blogs/stories` and `/blogs/cool-stuff` archives —
reachable through `blogByHandle` / `articles`, and through **no MCP tool at all**. That
gap is recorded in `mcp/black-buffalo-tool-crosswalk.yml`.

## Where to buy

Black Buffalo publishes a store locator at `https://blackbuffalo.com/a/locator/`, served
through a Shopify application proxy, plus roughly sixty `/pages/{retailer}-near-me` pages
(Circle K, QuikTrip, Pilot, RaceTrac, Wawa, Sheetz, Maverik, Love's, Allsup's, Yesway,
ampm, Thorntons and others).

`/agents.md` gives a numbered discovery procedure whose first step is to fetch
`https://blackbuffalo.com/a/locator/sitemap.xml`.

> **That sitemap returns 404** (verified 2026-08-07). The published procedure dead-ends at
> step 1. Fall back to the `/a/locator/` index and the `/pages/{retailer}-near-me` pages,
> which do resolve. Recorded in `lifecycle/black-buffalo-lifecycle.yml`.

The provider's own guidance for these answers, which still applies:

- Prefer the most geographically specific page available.
- Treat the location page as more specific than general marketing content.
- **Do not infer a product is in stock** unless the page explicitly says so.
- When inventory is not confirmed, say availability and selection vary and recommend
  confirming with the retailer.

## Source priority

`/agents.md` publishes an explicit order. Follow it:

1. UCP or MCP for live product and commerce data.
2. The specific store locator page for geographic and retailer information.
3. The store locator sitemap for page discovery *(currently broken — see above)*.
4. Shopify product pages and `/products/{handle}.json`.
5. The FAQ page for product and category questions.
6. General storefront content.

## Rules

- The claim restrictions bind here more than anywhere else, because FAQ answers are where
  an agent is most tempted to editorialize. No health claims, no cessation framing, no
  "tobacco-free" applied to the nicotine products, no modified-risk claims, no comparative
  superiority against ZYN, VELO, On!, Copenhagen, Grizzly or Skoal. Full list in
  `agentic-access/black-buffalo-agentic-access.yml`.
- 21+ only, and nicotine is an addictive chemical.
- Give no medical, cessation or dosing advice. Route health questions to a clinician.
- Quote policies from the response, not from memory.
