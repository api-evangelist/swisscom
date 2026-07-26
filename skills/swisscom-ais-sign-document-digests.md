---
name: Create an AdES signature with the Swisscom All-in Signing Service
description: >-
  Sign or seal a document with the Swisscom Trust Services All-in Signing Service by submitting
  document digests over the ETSI TS 119 432 REST interface, and embed the returned signature.
api: openapi/swisscom-all-in-signing-service-openapi.yml
operations: [signDoc]
generated: '2026-07-25'
method: generated
source: >-
  openapi/swisscom-all-in-signing-service-openapi.yml,
  https://github.com/SwisscomTrustServices/AIS,
  https://github.com/SwisscomTrustServices/pdfbox-ais-client
---

# Create an AdES signature with the Swisscom All-in Signing Service

The All-in Signing Service (AIS) is Swisscom Trust Services' qualified signing back end. It is
**digest-based**: your document never leaves your infrastructure. You hash it, AIS signs the hash,
and you embed the returned signature locally.

## Before you start

- AIS is not self-service. Access requires a signed Swisscom Trust Services contract, a client
  certificate for **mutual TLS**, and a `credentialID` / claimed identity provisioned for your
  account. There is no public sandbox and no OAuth flow.
- Productive endpoint: `https://ais.swisscom.com/AIS-Server/rs/v1.0`. The published OpenAPI declares
  no `servers[]` — that host is documented in the client libraries and integration guides.
- Do not implement the PDF plumbing yourself unless you must. Swisscom publishes first-party clients
  that handle hashing, signature embedding and long-term validation: `pdfbox-ais-client` and
  `itext7-ais-client` (Java), `TrustServices.AIS.Net.Client` (NuGet), and a SetaPDF-Signer addon
  (PHP). See `packages/swisscom-packages.yml` and `cli/swisscom-cli.yml`.

## Steps

1. **Prepare the document digest.** Compute the digest(s) to be signed and record the algorithm as
   an OID — for example `2.16.840.1.101.3.4.2.1` for SHA-256. Multiple hashes can be signed in one
   request (batch signing).

2. **Call `signDoc`** — `POST /signatures/signDoc`. The request body carries:
   - `requestID` — your correlation UUID.
   - `SAD` — the signature activation data authorising the signer.
   - `credentialID` — the signing credential, e.g. `OnDemand-Qualified`.
   - `profile` — `http://uri.etsi.org/19432/v1.1.1#/creationprofile#`.
   - `signatureFormat` — e.g. `P` for PAdES.
   - `conformanceLevel` — e.g. `AdES-B-LT` when you want long-term validation material returned.
   - `documentDigests` — `{ hashAlgorithmOID, hashes: [base64, …] }`.

3. **Read the response.** `200` returns:
   - `responseID` — echoes your `requestID`; assert they match.
   - `SignatureObject[]` — one detached signature per submitted hash, in submission order.
   - `validationInfo` — `ocsp[]` and `crl[]` material. Embed this when your conformance level is
     `-LT` or higher; without it the signature will not validate long-term.

4. **Handle step-up and long-running requests.** On-demand signatures with step-up require the signer
   to authorise on their device (Mobile ID / Signature Approval app). Those are not returned
   synchronously — the client polls `/AIS-Server/rs/v1.0/pending` until the signature is ready. AIS
   publishes **no webhooks**; polling is the only completion mechanism.

5. **Embed the signature** into the PDF locally with PDFBox, iText or SetaPDF, then re-verify the
   output before storing it.

## Errors

The spec declares `4XX` (Bad request) and `5XX` (Unexpected error) response ranges, both returning
an `ErrorResponse` body — there is no per-status error catalogue and no RFC 9457 problem type. Log
your `requestID` on every call: it is the only correlation handle Swisscom support can use.

## Compliance context

Swisscom Trust Services is an accredited qualified trust service provider under eIDAS and a qualified
certification service provider under ZertES, with conformity certificates issued by the supervisory
authority KPMG, and is listed on the EU/EEA trusted list, the Swiss OFCOM list and the Adobe Approved
Trust List. Signature level (EES / FES-AES / QES) determines whether identity verification is
required. See `conformance/swisscom-conformance.yml` and `security/swisscom-trust-center.yml`.
