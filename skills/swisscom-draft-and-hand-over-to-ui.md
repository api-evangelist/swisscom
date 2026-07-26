---
name: Draft a Swisscom Sign process and hand it to the Swisscom UI
description: >-
  Create a draft signing process from your backend, attach the documents, then hand the process over
  to the Swisscom Sign web client so a human finishes configuring and releasing it.
api: openapi/swisscom-sign-integration-api-openapi.json
operations: [create, attach, setup, getStatus]
generated: '2026-07-25'
method: generated
source: openapi/swisscom-sign-integration-api-openapi.json, https://sign.swisscom.ch/docs/guide/concepts
---

# Draft a Swisscom Sign process and hand it to the Swisscom UI

Use this when your system knows *what* has to be signed but a person should decide *who* signs and
*where* the signature goes. Your backend seeds the draft; the Swisscom Sign UI finishes it.

## Before you start

- OAuth 2.0 client credentials against
  `https://sign.swisscom.ch/realms/swisscom-public/protocol/openid-connect/token`;
  send `Authorization: Bearer <token>`.
- Scope `sswp:process:create` for steps 1–3, `sswp:process:read` for step 4.

## Steps

1. **Create the draft** — `create` (`POST /api/process`). Seed whatever you already know: the
   signature level, `teamId` so the process appears under the right team in the UI, `partnerId` for
   partner attribution, `validUntil`, and `properties[]` carrying your own correlation keys
   (`{key, value}`). You may leave `invitees` unset — the human will add signers in the UI.
   Persist the returned process `id`; there is no idempotency key.

2. **Attach the documents** — `attach` (`POST /api/process/{processId}/attach`). The process must be
   in `CREATED`. Base64 `content` + `contentType`, up to 40 MB per document. Set an
   `externalIdentifier` so you can match the file back to your own record later. Signature positions
   are optional here — the person completing the process can place them in the UI.

3. **Hand over to the UI** — `setup` (`POST /api/process/{processId}/setup`). Send `language`,
   `backUrl` (where to send the user if they abandon), `successUrl` (where to send them when they
   finish), and optionally `authentication` (`identity` + `namespace`) plus `scope`/`mode` for SSO.
   Returns `202` with a `url`. Redirect the user there.
   A `409` here almost always means the process has left `CREATED` — re-read it before retrying, and
   do not blind-retry `setup` on a timeout.

4. **Pick the process back up** — `getStatus` (`GET /api/process/{processId}/status`) once the user
   returns to your `successUrl`. Expect `PENDING` once they released it in the UI. From here the flow
   is identical to `swisscom-invite-and-sign.md` — poll to `COMPLETED`, then download the signed
   files and the audit record.

## Choosing between setup and release

- `setup` — a person completes configuration in the Swisscom UI. Non-interactive fields you did not
  set can still be filled in there.
- `release` — your backend has everything it needs and releases without any UI. Use that path when
  the signer set is deterministic.

Both require the process to be in `CREATED`; they are alternative exits from the draft state, not a
sequence.

## Errors

Custom JSON envelope `{id, timestamp, name, message, path, httpStatus, clientSubject, clientBody}` —
see `errors/swisscom-problem-types.yml`. Surface `clientSubject`/`clientBody`, log `id`.
