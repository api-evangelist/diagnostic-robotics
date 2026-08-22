# Diagnostic Robotics

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
> **Not from the company, and here with a question?** You are welcome here — we would rather be the
> front line and point you the right way than have a good report go nowhere. What this repository
> can answer is narrow, though, so it is worth knowing who you are actually looking for:
>
> - **A question about how the API works, an account, billing, or a bug in the service** — that is
>   the company's own support, not us. We profile this API; we do not operate it and cannot see
>   your account.
> - **A bug in an open-source project we only catalog** — file it on that project's own repository.
>   This has happened with a real and correct bug report that reached us instead of the people who
>   could fix it, which helped nobody.
> - **Anything about this listing itself** — the description, the tags, the rating, a missing or
>   wrong artifact — is ours. Open an issue here.
> - **Not sure, or something general about API Evangelist or APIs.io** — open an issue on the
>   [APIs.io Inbox](https://github.com/api-search/inbox) and we will route it.
>
> **This repository contains no software, and we will never ask you to download anything.** There is
> no build, release, installer, or binary here — only text and machine-readable API descriptions, so
> there is nothing here that can be "corrupt" or need "repairing". Any issue, comment, or email
> claiming otherwise and offering a download link is not from us and is hostile. Do not follow the
> link; it is a lure. Report it to GitHub and, if you like, tell us at
> [info@apievangelist.com](mailto:info@apievangelist.com) so we can take it down.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

Diagnostic Robotics is a clinical AI company (Tel Aviv / Boston) whose platform turns medical claims, EHR
and FHIR data into predictive risk stratification, actionable care next steps, HCC/RAF risk-adjustment gap
closure and automated care operations for health plans, providers and care organizations.

- Website: https://diagnosticrobotics.com/
- Developer portal: https://docs.diagnosticrobotics.com/
- Trust center: https://diagnosticrobotics.com/about/trust-center
- Status page: https://diagnosticrobotics.statuspage.io

## APIs profiled

| API | Spec | Ops | Auth |
|---|---|---|---|
| Precision Population Health (Proactive Patient Risk Feed) | OpenAPI 3.0.2 | 15 | OAuth 2.0 client credentials |
| Patient Questionnaire | Swagger 2.0 | 13 | `x-client` API key |
| Symptom Search Service | Swagger 2.0 | 2 | `x-client` API key |

All three contracts were harvested from the company's public Stoplight workspace (`wk:2242`,
`docs.diagnosticrobotics.com`) and are stored verbatim in `openapi/_original/`.

## What this profile records

- `openapi/` - three harvested contracts plus verbatim originals
- `authentication/`, `scopes/` - OAuth 2.0 + API key profile; zero scopes exist, and the declared Auth0 tenant no longer resolves
- `conventions/`, `errors/`, `data-model/`, `lifecycle/`, `conformance/` - derived runtime semantics, error envelope, entity graph, versioning and standards posture
- `sandbox/` - live Stoplight Prism mock servers, the only way to exercise this API without a contract
- `components/` - the four iframe widget surfaces
- `security/`, `well-known/` - domain security probe, trust center, and a recorded absence of every `/.well-known/` document
- `plans/`, `rate-limits/`, `packages/` - honest zeros, each with the URLs that were checked
- `skills/`, `llms/`, `overlays/`, `mcp/` - generated agent-facing artifacts grounded in real operationIds

## Notable findings

- The API host answers **HTTP 200 with a React SPA shell for every `/.well-known/` path** and for `/openapi.json`. Those 200s are not documents and are recorded as misses.
- The published OpenAPI's `authorizationCode` flow points at `digital-outreach.us.auth0.com`, a tenant that returns `404 Unknown host`.
- The Statuspage tenant is live but `updated_at` is 2020-06-29 and the indicator has read "Partial System Outage" ever since.
- Both questionnaire specs publish the literal placeholder host `env-name.diagnosticrobotics.com`.
- No idempotency contract, no documented rate limit, no `429`, no SDK, no pricing.
