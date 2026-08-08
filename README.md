# Black Buffalo

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

Black Buffalo Inc. is an American smokeless tobacco alternative company founded in 2015 and
headquartered in Chicago, Illinois, selling nicotine pouches, long cut dip and its nicotine-free
ZERO line direct to adult consumers (21+) at blackbuffalo.com and through convenience and fuel
retailers. Imperial Brands' U.S. subsidiary ITG Brands acquired the company in May 2026.

Black Buffalo runs no developer program and publishes no OpenAPI, but its Shopify-hosted
storefront exposes a substantial machine-readable surface from its own domain:

- **Storefront GraphQL API** — anonymously introspectable at `/api/2026-04/graphql.json`
  (424 types, 35 query fields, 41 mutations). SDL in `graphql/`.
- **Two live MCP servers** — `/api/mcp` (5 tools) and `/api/ucp/mcp` (13 tools), both of which
  answered an anonymous `tools/list` with full JSON Schema input contracts. Captured in `mcp/`.
- **Universal Commerce Protocol** merchant profile at `/.well-known/ucp` (UCP 2026-04-08).
- **OpenID Connect + RFC 8414** discovery for customer accounts.
- **A provider-authored `/agents.md` and `/llms.txt`** telling AI agents which surface to use,
  what claims they may not make about a regulated nicotine product, and that no agent may
  finalize payment without contemporaneous human approval.

- https://blackbuffalo.com/
