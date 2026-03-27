# Letters Post

Attaches mailing letter PDFs or updates the status of existing letters for a previously posted citation. This endpoint supports two modes of operation: uploading a new PDF, or updating the status of a previously uploaded letter.

> **Note**: Each request must provide **either** a `letterUrl` (to upload/replace a PDF) **or** both `status` and `statusNote` (to update letter status). Providing neither will result in a validation error.

### Fields
| Field          | Required     | Type/Format | Max Len | Example(s)                          | Description |
|----------------|--------------|-------------|---------|-------------------------------------|-------------|
| `referenceId`  | Yes          | string      | 50      | `6B547-F4684`                       | Internal reference ID from the original Citations post. |
| `letterId`     | Yes          | string      | 50      | `LTR-2024-001`                      | A unique identifier for this letter in your system. |
| `letterUrl`    | Conditional  | string      |         | `https://storage.net/letter.pdf`    | An absolute HTTPS URL or data URI pointing to a PDF file. Required if `status`/`statusNote` are not provided. |
| `pageCount`    | No           | string      |         | `2`                                 | The number of pages in the PDF. If omitted, page count is determined automatically. |
| `status`       | Conditional  | string      |         | `Closed`                            | Letter status update. Required if `letterUrl` is not provided. Valid values: `Closed`, `Expired`, `Error`, `Undeliverable`, `Other`. |
| `statusNote`   | Conditional  | string      |         | `Returned to sender`                | A note explaining the status change. Required when `status` is provided. |

### Mode 1: PDF Upload

When providing a `letterUrl`, the system will download the PDF, validate it, and store both a raw and masked (PII-redacted) copy. The letter is marked as `Printed` unless a `status` is also provided.

```yaml
[{
    "referenceId": "6B547-F4684",
    "letterId": "LTR-2024-001",
    "letterUrl": "https://storage.example.com/letters/notice-001.pdf"
},{
    "referenceId": "8CC21-BB433",
    "letterId": "LTR-2024-002",
    "letterUrl": "https://storage.example.com/letters/notice-002.pdf",
    "pageCount": "3"
}]
```

### Mode 2: Status-Only Update

When no `letterUrl` is provided, both `status` and `statusNote` are required. This updates the status of a previously uploaded letter without replacing the PDF.

```yaml
[{
    "referenceId": "6B547-F4684",
    "letterId": "LTR-2024-001",
    "status": "Undeliverable",
    "statusNote": "Returned to sender - address unknown"
},{
    "referenceId": "8CC21-BB433",
    "letterId": "LTR-2024-002",
    "status": "Closed",
    "statusNote": "Confirmed delivered"
}]
```

### Valid Status Values

| Status          | Description |
|-----------------|-------------|
| `Closed`        | The letter process is complete. |
| `Expired`       | The letter has expired without response. |
| `Error`         | An error occurred during mailing. |
| `Undeliverable` | The letter could not be delivered to the address. |
| `Other`         | Other status — provide details in `statusNote`. |

### Error Responses

In addition to the [standard HTTP error codes](../README.md#error-responses), individual records in a batch may fail validation. Failed records are returned in the `errors` array of a `200 OK` response while valid records are still processed.

#### Model Validation Errors

| Error Message | Cause |
|---------------|-------|
| `ReferenceId is null or empty` | The `referenceId` field was not provided. |
| `Either LetterUrl must be provided, or both Status and StatusNote must be specified.` | Neither a PDF URL nor a status/note combination was provided. |
| `LetterUrl must be a valid absolute HTTPS or data URI pointing to a PDF file.` | The `letterUrl` is not a valid HTTPS URL or data URI. |
| `{field} is greater than allowed {max} chars` | A string field exceeds its maximum length. |
| `{field} is less than required {min} chars` | A string field is shorter than its minimum length. |

#### Business Logic Errors

| Error Message | Cause |
|---------------|-------|
| `When no letterUrl is provided, both Status and StatusNote are required.` | A status-only update was attempted without both `status` and `statusNote`. |
| `Invalid Status, valid entries are Closed, Expired, Error, Undeliverable, Other` | The `status` value is not one of the accepted values. |
| `Missing or invalid 'letterId' in payload` | The `letterId` field is missing or could not be resolved. |
| `Mailing Letter {letterId} not found` | No letter with the provided `letterId` exists in the system. |
| `letterid not associated with ReferenceID` | The `letterId` exists but belongs to a different citation than the one specified. |
| `Missing 'letterUrl' in payload` | The letter URL could not be resolved from the request. |
| `Invalid PDF URL '{letterUrl}'` | The `letterUrl` could not be parsed as a valid URI. |
| `Unable to download PDF from '{letterUrl}'` | The PDF could not be downloaded from the provided URL. |
| `Downloaded file is not a valid PDF` | The downloaded file is not a valid PDF document. |
| `Raw PDF upload failed` | The raw PDF could not be stored in cloud storage. |
| `Failed to generate masked PDF` | The PII-redacted version of the PDF could not be generated. |
| `Masked PDF upload failed` | The masked PDF could not be stored in cloud storage. |

#### Example Error Response

```yaml
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{
    "id": "d4e5f6a7-b8c9-0123-defg-h45678901234",
    "count": 1,
    "errors": [{
        "index": 1,
        "ref": "8CC21-BB433",
        "error": "Unable to download PDF from 'https://expired-link.example.com/letter.pdf'"
    }]
}
```
