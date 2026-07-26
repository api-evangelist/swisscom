---
name: Audit and reconcile Swisscom Sign processes
description: >-
  Sweep the organization's signature processes, reconcile them against your own records, capture
  signer validation results, and archive signed documents and audit trails before Swisscom deletes them.
api: openapi/swisscom-sign-integration-api-openapi.json
operations: [findAll, getProcess, getStatus, getFile, getFileContent, getRecords]
generated: '2026-07-25'
method: generated
source: openapi/swisscom-sign-integration-api-openapi.json, https://sign.swisscom.ch/docs/guide/release-notes/2026-03
---

# Audit and reconcile Swisscom Sign processes

A read-only sweep. Everything here needs only `sswp:process:read:all` (listing) and
`sswp:process:read` (detail), so it can run on a credential with no write scope at all.

## Steps

1. **List the organization's processes** — `findAll` (`GET /api/process`, scope
   `sswp:process:read:all`). Query parameters:
   - `status` — `CREATED` | `PENDING` | `COMPLETED` | `EXPIRED`
   - `teamId` — restrict to one team
   - `validUntilWithin` — ISO 8601 window on `validUntil`, e.g. `P30D` or `PT12H`
   - `page` (zero-based), `size` (default 20; **anything above 20 returns 400**), `sort` as
     `property,(asc|desc)`, default `createdDate,DESC`
   The response is a `ProcessPage`: `content[]` plus a `page` object with `totalElements`,
   `totalPages`, `size`, `number`. Walk pages until `last` is true. Note this endpoint only arrived
   in API version 2.11.0 (March 2026).

2. **Find what needs attention.** Two useful sweeps:
   - `status=PENDING&validUntilWithin=P5D` — processes about to expire unsigned.
   - `status=COMPLETED` — processes whose output you have not archived yet.

3. **Read the detail** — `getProcess` (`GET /api/process/{processId}`, scope `sswp:process:read`).
   Reconcile on `properties[]` and on each `fileReferences[].externalIdentifier`, which are the only
   fields you control. Per signer, check:
   - `status` — including `WAITING` (not yet invited, signing-order flows), `REMOVED`, `REPLACED`
     (delegated or swapped) and `DECLINED` (declined with a reason; the whole process ends).
   - `certificateMatch` — `MATCH` means the invited name matched the signing certificate and no
     manual re-check is needed; `MISMATCH` should be routed to a human; `UNCHECKED` means the
     comparison has not run.
   Use `getStatus` instead when you only need the lifecycle state — it is a much smaller payload.

4. **Archive before Swisscom does.** Every process carries `archiveOn` and `archivedOn`. Once
   archived, `getFile` and `getFileContent` return **410 Gone** and the file is unrecoverable. Sort
   your sweep by `archiveOn` and pull anything approaching it:
   - `getFile` (`GET /api/process/{processId}/file/{fileId}`) — Base64 JSON.
   - `getFileContent` (`GET /api/process/{processId}/file/{fileId}/content`) — raw binary.
   Take `lastSignedFileId` from `fileReferences` for the signed output, `initialFileId` for the
   original. A `412` means the document is not downloadable yet — the process has not reached a
   state where the signed file exists.

5. **Store the evidence trail** — `getRecords` (`GET /api/process/{processId}/record`). Returns
   `records[]` of `AuditRecord` (`id`, `createdDate`, `action`, `status`, `extras`). Persist it with
   the signed PDF; it is what you produce if the signature is ever challenged.

## Rate and retry behaviour

Swisscom publishes no rate-limit headers or quotas for this API. Be conservative: page sequentially,
back off on `5xx`, and do not parallelise the sweep aggressively. All operations in this skill are
reads, so retries are safe.

## Errors

`400` on this path almost always means `size > 20` or a malformed `sort`. `401` means the token
expired or lacks `sswp:process:read:all`. Everything else arrives in the custom envelope
`{id, timestamp, name, message, path, httpStatus, clientSubject, clientBody}` — see
`errors/swisscom-problem-types.yml`.
