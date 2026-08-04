# Swisscom (swisscom)

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

Swisscom is Switzerland's largest telecommunications provider and the incumbent mobile network operator, fixed-line carrier and IT services company for the Swiss market, majority-owned by the Swiss Confederation and also operating Fastweb in Italy. It is a facilities-based MNO that owns the radio access and core network, sells mobile, broadband, TV and enterprise IT, and monetises network-derived data.

Its API posture is unusual for a European incumbent. Alongside the expected partner-gated enterprise surface, Swisscom runs a genuine first-party API marketplace at [digital.swisscom.com](https://digital.swisscom.com/) — self-identified in its own JSON-LD as the "Swisscom APIs Portal" — listing ten productive API products served from a live gateway at `https://api.swisscom.com`. It also maintains a separate, actively released Swisscom Sign Integration API with a downloadable OpenAPI 3.1 document.

The catch: every API reference, credential and specification beyond Swisscom Sign sits behind a Swisscom login, and several products require a signed service contract before subscription. The marketplace is a storefront, not open documentation.

**APIs.json:** [https://raw.githubusercontent.com/api-evangelist/swisscom/refs/heads/main/apis.yml](https://raw.githubusercontent.com/api-evangelist/swisscom/refs/heads/main/apis.yml)

## CAMARA and GSMA Open Gateway

Swisscom is one of the **21 founding signatories** of the GSMA Open Gateway Memorandum of Understanding announced at MWC Barcelona in February 2023.

Nothing has been made callable. There is no `opengateway.swisscom.ch`, no `developers.opengateway.swisscom.*`, no `camara.swisscom.com` and no `networkapi.swisscom.com` — none of those hosts resolve. No CAMARA API appears in the ten-product marketplace catalog, and CAMARA-shaped path probes against `api.swisscom.com` (`/sim-swap/v0/check`, `/number-verification/v0/verify`, `/location-verification/v0/verify`, `/device-status/v0/roaming`, `/quality-on-demand/v0/sessions`) all return 404.

**A press release is not an implementation.** The Open Gateway commitment is real and documented; the implementation is not visible from outside.

Two Swisscom products invite confusion and should not be counted as CAMARA:

- **Phone Number Validation** is a proprietary SMS-OTP two-factor API. It is *not* CAMARA Number Verification — it proves possession of a handset by delivering a code, it does not do network-based number verification.
- **Heatmaps** is proprietary Mobility Insights population density. It is *not* CAMARA Population Density Data.

Where network-derived identity signals *have* reached developers, they did so through a third party: IPification announced in March 2021 that its one-click phone verification, SIM-swap detection and KYC services were deployed inside the Swisscom network. That is the sector pattern — the operator owns the signal, someone else owns the developer relationship.

No TM Forum Open API conformance certification was found. No NEF/SCEF exposure, network-slicing or edge/MEC API was found.

## Tags

- Telecommunications
- Switzerland
- Mobile Network Operator
- Broadband
- Network APIs
- Open Gateway
- Messaging
- SMS
- Voice
- Identity Verification
- Mobility Data
- Digital Signatures
- eSIM
- Artificial Intelligence

## Timestamps

- **Created:** 2026-07-25
- **Modified:** 2026-07-25

## APIs

### Swisscom Sign Integration API

Create, configure, release and monitor digital signature processes end to end. OAuth 2.0 client credentials against a Swisscom Keycloak realm with fine-grained `sswp:process:*` scopes. The only Swisscom API found with a publicly downloadable machine-readable specification.

- **Human URL:** [https://sign.swisscom.ch/docs/api](https://sign.swisscom.ch/docs/api)
- **Base URL:** `https://sign.swisscom.ch/system`
- **OpenAPI:** [openapi/swisscom-sign-integration-api-openapi.json](openapi/swisscom-sign-integration-api-openapi.json) — OpenAPI 3.1.0, v2.19.0, 10 paths

### Swisscom All-in Signing Service (AIS) API

AdES signatures and seals under eIDAS and ZertES, following the ETSI TS 119 432 remote signature creation profile. Swisscom Trust Services publishes the OpenAPI, plus WSDL/WADL, Postman collections and iText/PDFBox clients, on public GitHub; full integration guides stay in the partner area.

- **Human URL:** [https://github.com/SwisscomTrustServices/AIS](https://github.com/SwisscomTrustServices/AIS)
- **Base URL:** `https://ais.swisscom.com/AIS-Server/rs/v1.0`
- **OpenAPI:** [openapi/swisscom-all-in-signing-service-openapi.yml](openapi/swisscom-all-in-signing-service-openapi.yml) — OpenAPI 3.0.1, v3

### Swisscom Text Messaging (SMS) API

A2P SMS with onward delivery to 260+ operators globally and delivery-notification callbacks. Sending is restricted to Switzerland by default with a configurable monthly cost limit.

- **Human URL:** [https://digital.swisscom.com/products/text-messaging](https://digital.swisscom.com/products/text-messaging)
- **Base URL:** `https://api.swisscom.com/messaging/v1/sms`

### Swisscom Phone Number Validation API

SMS token 2FA — start a validation flow against an MSISDN, then verify the token the user received.

- **Human URL:** [https://digital.swisscom.com/products/phone-number-validation](https://digital.swisscom.com/products/phone-number-validation)
- **Base URL:** `https://api.swisscom.com/messaging/v1/tokenvalidation`

### Swisscom Receive SMS API

Inbound SMS inboxes for two-way messaging, alerting and device control.

- **Human URL:** [https://digital.swisscom.com/products/receive-sms](https://digital.swisscom.com/products/receive-sms)
- **Base URL:** `https://api.swisscom.com/messaging/sms/inboxes`

### Swisscom Heatmaps API

Estimated population density per 100m x 100m tile per hour across Switzerland, rolling two years, with socio-demographic splits. OAuth 2.0 client credentials, `scs-version` header. Free demo plan and a public Python client sample.

- **Human URL:** [https://digital.swisscom.com/products/heatmaps](https://digital.swisscom.com/products/heatmaps)
- **Base URL:** `https://api.swisscom.com/layer/heatmaps/demo`

### Swisscom Dwell Times API

Frequency distribution of observed dwell times per 500m x 500m tile per day, bucketed from 0-5 minutes to 8-24 hours.

- **Human URL:** [https://digital.swisscom.com/products/dwelltimes](https://digital.swisscom.com/products/dwelltimes)

### Swisscom Origin Destination API

Trips between two Swiss regions for weekdays and weekends over a calendar month, including the share made by train. Queryable by 1km tile, postal code or NPVM zone.

- **Human URL:** [https://digital.swisscom.com/products/origin-destination](https://digital.swisscom.com/products/origin-destination)

### Swisscom Smart Catalogs for NATEL go API

Business/wholesale provisioning across the full NATEL go subscription lifecycle — activations, number portability, options and roaming, SIM and eSIM ordering with real-time activation and PUK, inventory and reporting.

- **Human URL:** [https://digital.swisscom.com/products/smartcatalogs-natelgo](https://digital.swisscom.com/products/smartcatalogs-natelgo)

### Swiss AI Platform Inference Endpoints API

OpenAI-compatible inference for chat, multimodal, audio, embedding, reasoning and guardrail models hosted on NVIDIA SuperPod infrastructure in Switzerland — including Meta Llama, OpenAI GPT OSS, NVIDIA models and Apertus 70B from ETH Zurich and EPFL. Business customers only; a signed service contract is required before subscribing.

- **Human URL:** [https://digital.swisscom.com/products/swiss-ai-platform](https://digital.swisscom.com/products/swiss-ai-platform)
- **Base URL:** `https://api.swisscom.com/layer/swiss-ai-platform`

### Swisscom Business Identity Validator API

Legal entity and authorised signatory lookup against Swiss federal and cantonal commercial registries and selected EU business registers. Its developer site at `developer.validator.swisscom.com` is an authentication wall (HTTP 401).

- **Human URL:** [https://digital.swisscom.com/products/business-identity-validator](https://digital.swisscom.com/products/business-identity-validator)

### Swisscom Voice VoIP API

Legacy VoIP subscriber control — calls, forwardings, simultaneous ringing, phonebooks and event subscriptions. OAuth 2.0 with `read-voip-callforwardings` / `write-voip-callforwardings` scopes. Documented only as RAML-rendered HTML in a GitHub wiki; the gateway still answers 401.

- **Human URL:** [https://github.com/swisscom-api/doc/wiki](https://github.com/swisscom-api/doc/wiki)
- **Base URL:** `https://api.swisscom.com/voice/v1/voip`

### Swisscom Voice Mail API

Legacy voicemail listing and retrieval for a Swisscom MSISDN, OAuth 2.0 client-credentials or authorization-code.

- **Human URL:** [https://github.com/swisscom-api/doc/wiki](https://github.com/swisscom-api/doc/wiki)
- **Base URL:** `https://api.swisscom.com/voice/v1/voicemail`

## The dead layer

Swisscom's legacy developer programme is gone, and the remnants are still reachable enough to mislead:

- `developer.swisscom.ch` and `developers.swisscom.ch` do not resolve.
- `developer.swisscom.com` 302s to [swisscom.ch/cloudnative/](https://www.swisscom.ch/cloudnative/), a consulting marketing page.
- [docs.developer.swisscom.com](https://docs.developer.swisscom.com/) is still up but documents the public Cloud Foundry Application Cloud, decommissioned **2020-12-01**. Its "Swisscom APIs" list points at `docs-api.scapp.io` — dead (404).
- The `swisscom-developer` GitHub org holds six SDK/example repos last updated 2014-2018.
- The `swisscom-api/doc` wiki's "API specifications (external)" links point at the long-defunct rawgit.com.

## Links

- **Website:** [https://www.swisscom.ch/](https://www.swisscom.ch/)
- **Developer portal:** [https://digital.swisscom.com/](https://digital.swisscom.com/)
- **Swisscom Sign docs:** [https://sign.swisscom.ch/docs/](https://sign.swisscom.ch/docs/)
- **Messaging dashboard:** [https://sms.icp.swisscom.com/](https://sms.icp.swisscom.com/)
- **GitHub:** [swisscom](https://github.com/swisscom) · [SwisscomTrustServices](https://github.com/SwisscomTrustServices) · [swisscom-api](https://github.com/swisscom-api)
- **Vulnerability disclosure:** [https://github.com/swisscom/bugbounty](https://github.com/swisscom/bugbounty)
- **security.txt:** [https://www.swisscom.ch/.well-known/security.txt](https://www.swisscom.ch/.well-known/security.txt)
