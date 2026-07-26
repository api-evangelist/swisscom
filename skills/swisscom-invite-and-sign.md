---
name: Invite a signer and complete a Swisscom Sign process
description: >-
  Create a signature process, attach a PDF, release it so participants are notified, hand a signer a
  deep link, poll to completion and download the signed document plus the audit record.
api: openapi/swisscom-sign-integration-api-openapi.json
operations: [create, attach, release, open, getStatus, getFile, getRecords]
generated: '2026-07-25'
method: generated
source: openapi/swisscom-sign-integration-api-openapi.json, https://sign.swisscom.ch/docs/guide/concepts
---

# Invite a signer and complete a Swisscom Sign process

The Swisscom Sign Integration API is backend-first: your service owns the process and Swisscom owns
the signing experience. Base URL `https://sign.swisscom.ch/system` (test:
`https://test.sign.swisscom.ch/system`).

## Before you start

- Get an access token from the Keycloak realm with OAuth 2.0 **client credentials**:
  `POST https://sign.swisscom.ch/realms/swisscom-public/protocol/openid-connect/token` with
  `grant_type=client_credentials&client_id=…&client_secret=…`.
- Send it as `Authorization: Bearer <token>` on every call.
- This flow needs the `sswp:process:create` scope for the write steps and `sswp:process:read` for the
  read steps.
- Never run this from a browser. Swisscom's own documentation warns that entering client credentials
  in the browser-based API explorer exposes them; call from your backend.

## Steps

1. **Create the process** — `create` (`POST /api/process`, scope `sswp:process:create`).
   Set the signature level, the invitees (`invitees.signers[]` with `firstName`, `lastName`, and a
   verified `email` or `mobile`), and optionally `validUntil`, `teamId`, `partnerId` and free-form
   `properties`. Use `authorization: ACCOUNT` when the signer is already authenticated in your portal
   and you want to skip the SMS/email delivery step.
   Returns `201` with the process `id`. **Persist that id before doing anything else** — there is no
   idempotency key, so a retried create makes a second process.

2. **Attach the document** — `attach` (`POST /api/process/{processId}/attach`, scope
   `sswp:process:create`). The process must still be in `CREATED`. Send the PDF as Base64 `content`
   with its `contentType`, plus `signaturePositions`. Two positioning modes:
   - `locator: COORDINATE` — explicit `pageNumber`, `positionX`, `positionY`.
   - `locator: TAG` — resolve at runtime from a tag string embedded in the PDF (`\s1\` for index 0,
     `\s2\` for index 1, …), with optional `offsetX`/`offsetY` in PDF points (−200..200). Embed the
     tags in **white text** before uploading so they stay invisible.
   Maximum upload size is 40 MB. Returns `201` with a `fileId`. A repeated call attaches the document
   again — do not blind-retry.

3. **Release the process** — `release` (`POST /api/process/{processId}/release`, scope
   `sswp:process:create`). Optionally set `validUntil`, `language`, `signatureMethods` and
   `notification`. To chase pending signers, set `notification.reminder`:
   - `interval: "P3D"` — a recurring reminder every 3 days while the participant is still pending.
   - `beforeExpiry: ["P5D","P1D"]` — fixed reminders relative to `validUntil` (requires `validUntil`).
   Both are ISO 8601 durations and must be between `P1D` and `P30D`.
   Returns `200` with `participants[]` carrying permanently-valid signing URLs, or `208` if the
   process was already released — **`release` is the one repeat-safe write on this API**; `208`
   returns the same URLs, so a retry after a timeout is correct.
   Watch for `409` when reminders are configured while all notifications are suppressed, and `403`
   when the calling credential does not own the process.

4. **Deep-link a specific signer (optional)** — `open`
   (`POST /api/process/{processId}/open/{personId}`, scope `sswp:process:read`). The process must be
   `PENDING`. Set `language`, `backUrl`, `successUrl` and, for SSO, `authentication`
   (`identity` + `namespace`). Returns a signer-specific `url` you can redirect the user to.

5. **Poll to completion** — `getStatus` (`GET /api/process/{processId}/status`, scope
   `sswp:process:read`). States: `CREATED` → `PENDING` → `COMPLETED`, or `EXPIRED` when `validUntil`
   passes. There are no webhooks on this API; polling is the only option. Back off between polls.
   Use `getProcess` instead when you also need signer state — `SignerStatus` exposes `WAITING`,
   `REMOVED`, `REPLACED` and `DECLINED`, and `certificateMatch` (`UNCHECKED`/`MATCH`/`MISMATCH`)
   tells you whether the invited name matched the signing certificate.

6. **Download the signed document** — `getFile`
   (`GET /api/process/{processId}/file/{fileId}`, scope `sswp:process:read`) for Base64 JSON, or
   `getFileContent` (`GET /api/process/{processId}/file/{fileId}/content`) for raw binary. Use the
   `lastSignedFileId` from the process `fileReferences`.
   `412` means the document is not in a downloadable state yet — keep polling. `410` means the process
   was archived and the file is **gone**; download before `archiveOn`.

7. **Fetch the audit record** — `getRecords` (`GET /api/process/{processId}/record`, scope
   `sswp:process:read`). Store it with the signed PDF: it is the evidence trail.

## Error handling

Errors are custom JSON, not RFC 9457: `{id, timestamp, name, message, path, httpStatus,
clientSubject, clientBody}`. Show `clientSubject`/`clientBody` to a human, log `name` for the logical
cause, and quote `id` to Swisscom support for log correlation. `409` is Swisscom Sign's catch-all
conflict — read the body rather than assuming a duplicate. See
`errors/swisscom-problem-types.yml`.

## Testing

Sign up at `https://test.sign.swisscom.ch/`, subscribe (free with a Datatrans test card) and generate
TEST client credentials in the cockpit. End-to-end signing in test is limited to EES and AES — QES
needs the mobile app, which is not published for test. To force a GwG identification outcome, set the
signer's `lastName` to `X-MANUALTEST-HAPPYPATH`, `X-MANUALTEST-FRAUDIDENT` or
`X-MANUALTEST-CHECKPENDING` (IDnow/Intrum identifications only). See `sandbox/swisscom-sandbox.yml`.
