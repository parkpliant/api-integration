# Letters Post

This endpoint allows an authorized **print/mail service provider** to post a generated letter PDF back to ParkPliant — attaching it to an existing mailing-letter record — or to report a non-print outcome (e.g. closed, expired, undeliverable) for that letter.

> **🔒 Restricted endpoint — do not use unless instructed to.**
> Letters is **not** part of the general integration flow and is **not** intended for parking operators or lot owners.  It is available only to print/mail service providers who receive letter data through the [Mail Data Callback](../callbacks/mail-data.md), generate the physical notices, and report the result back.  Access is arranged and enabled per-account by ParkPliant.  If you have not been explicitly onboarded for this workflow, do not call this endpoint.

## Relationship to the Mail Data Callback

This endpoint is the return path for the [Mail Data Callback](../callbacks/mail-data.md):

1. ParkPliant delivers mailing data to your configured Mail Data callback URL.  Each record includes a `letter_id`.
2. You generate the letter (PDF) and mail it.
3. You post the result back here, referencing that same id as `letterId` — either attaching the rendered **PDF**, or reporting a **status** outcome if no letter was produced.

When a PDF is attached, ParkPliant stores the original and automatically generates a **masked** copy (with protected fields redacted) for operator viewing.  Because the source data is DPPA-protected MVR data, the same restrictions described in the [Mail Data Callback](../callbacks/mail-data.md) apply.

## Authentication

Basic authentication, using the same credentials as the other Post services.  The endpoint is enabled per-account; calls from accounts that are not provisioned for it will not be able to resolve mailing-letter records.

## Fields

The request body is always a JSON **array** (`[]`), even for a single record.

| Field | Required | Type/Format | Max Len | Example(s) | Description |
|-------|----------|-------------|---------|------------|-------------|
| `referenceId` | Yes | string | 50 | `6B547-F4684` | The `referenceId` of the citation, as provided on the original [Citations](../citations) post and echoed in the Mail Data callback as `reference_id`. |
| `letterId` | Yes | string | 50 | `458201` | The mailing-letter id to update — the `letter_id` value delivered in the [Mail Data Callback](../callbacks/mail-data.md). |
| `letterUrl` | Conditional | string | | `https://my.storage.net/letters/458201.pdf` | An absolute **HTTPS** or **data** URI pointing to the generated letter **PDF**.  Required unless you are sending a status-only update (see below). |
| `status` | Conditional | string | | `Undeliverable` | The outcome of the letter.  **Required when `letterUrl` is omitted.**  See [Status values](#status-values).  Matched case-insensitively. |
| `statusNote` | Conditional | string | | `Return to sender - vacant` | A short, human-readable note giving context for `status`.  **Required when `letterUrl` is omitted.** |
| `pageCount` | No | string | | `2` | Informational only.  When a PDF is attached, ParkPliant derives the authoritative page count from the file itself; this field is not used to set it. |

> **One of two things must be present on every record:** either a valid `letterUrl`, **or** both `status` and `statusNote`.  A record with neither is rejected.

### Status values

`status` must be one of the following (case-insensitive):

| Value | Meaning |
|-------|---------|
| `Closed` | The citation has been resolved by payment or dismissal. |
| `Expired` | The notice has passed its valid response period. |
| `Error` | A processing issue occurred and the letter was not generated. |
| `Undeliverable` | The letter could not be delivered (invalid address or return-to-sender). |
| `Other` | None of the above apply; ParkPliant will review and categorize. |

## Behavior

The endpoint operates in one of two modes per record, based on whether `letterUrl` is present.

**1. Attach a PDF** (`letterUrl` provided)
- The `letterId` must resolve to an existing mailing-letter record, and that record must belong to the citation identified by `referenceId` (otherwise: `letterid not associated with ReferenceID`).
- The PDF is downloaded and validated; a non-PDF or unreachable URL is rejected.
- The original is stored and a **masked** copy is generated and stored.  The mailing letter's page count is set from the file.
- The letter's status is set to **`Printed`**, unless you also supply a `status`, in which case that status (and `statusNote`) is applied instead.
- The first time a letter is posted for a citation, the citation is marked as first-worked.

**2. Status-only update** (`letterUrl` omitted)
- Requires a valid `letterId`, a valid `status`, and a `statusNote`.
- Updates the mailing-letter record's status and note without attaching a PDF.

## Example — attach a PDF

```yaml
POST /api/letters HTTP/1.1
Host: push.parkpliant.com
Authorization: Basic <base64(username:password)>
Content-Type: application/json
Accept: application/json

[{
  "referenceId": "6B547-F4684",
  "letterId": "458201",
  "letterUrl": "https://my.storage.net/letters/458201.pdf"
}]
```

## Example — status-only update

```json
[{
  "referenceId": "6B547-F4684",
  "letterId": "458201",
  "status": "Undeliverable",
  "statusNote": "Return to sender - vacant"
}]
```

## Example Response

```json
{
  "id": "c31deb20-1069-40c8-b218-e7f7e63ba56d",
  "count": 1,
  "errors": []
}
```

- `id` — transaction id; please record this value when reporting issues.
- `count` — the number of mailing-letter records successfully updated.
- `errors` — per-record errors (same shape as the other endpoints); see [Error responses](../#error-responses).

### Common errors

| Error | Cause |
|-------|-------|
| `Either LetterUrl must be provided, or both Status and StatusNote must be specified.` | The record had neither a PDF nor a status outcome. |
| `When no letterUrl is provided, both Status and StatusNote are required.` | Status-only update missing `status` or `statusNote`. |
| `Invalid Status, valid entries are Closed, Expired, Error, Undeliverable, Other` | `status` was not one of the supported values. |
| `Missing or invalid 'letterId' in payload` | `letterId` was absent or not a positive number. |
| `Mailing Letter <id> not found` | No mailing-letter record matches the supplied `letterId`. |
| `letterid not associated with ReferenceID` | The `letterId` belongs to a different citation than the supplied `referenceId`. |
| `Invalid PDF URL '<url>'` / `Unable to download PDF from '<url>'` / `Downloaded file is not a valid PDF` | The `letterUrl` was not a valid, reachable PDF. |
