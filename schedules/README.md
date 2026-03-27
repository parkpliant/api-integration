# Schedules Post

Updates the fee schedule for a previously posted citation. This endpoint allows you to manage payment schedules independently after a citation has been created via the [Citations](../citations) API.

> **Note**: Posting a new schedule replaces all existing schedule entries for the citation. To clear a schedule, post an empty `schedule` array.

### Fields
| Field              | Required | Type/Format | Max Len     | Example(s)                   | Description |
|--------------------|----------|-------------|-------------|------------------------------|-------------|
| `referenceId`      | Yes      | string      | 50          | `6B547-F4684`                | Internal reference ID from the original Citations post. |
| `schedule`         | Yes      | array       |             | *(below)*                    | Array of schedule entries. May be empty `[]` but must not be `null`. |
| `schedule[].asOf`  | Yes      | string      |             | `2021-12-25T22:00:00-6:00`   | [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601) timestamp when the scheduled amount becomes effective. |
| `schedule[].totalDue` | Yes   | decimal     | 0.01–9999.99 | `65.00`                     | Total amount due as of the `asOf` timestamp (replacement value, not additive). |

### Behavior
- Posting a schedule **replaces** all existing fee schedule entries for the citation.
- Schedule entries with an `asOf` time in the past are applied retroactively — the citation's current amount due is updated immediately.
- Future schedule entries are applied automatically within one hour of their `asOf` time.
- A [Status](../status) update with a new `amountDue` will override the most recently applied schedule entry, but will not prevent future entries from being applied.

### Example

```yaml
[{
    "referenceId": "6B547-F4684",
    "schedule": [
        { "asOf": "2021-12-01T00:00:00-5:00", "totalDue": 42.00 },
        { "asOf": "2021-12-15T00:00:00-5:00", "totalDue": 57.00 },
        { "asOf": "2022-01-01T00:00:00-5:00", "totalDue": 75.00 }
    ]
},{
    "referenceId": "8CC21-BB433",
    "schedule": []
}]
```

In the above example:
- The first record sets a three-step escalating schedule for citation `6B547-F4684`.
- The second record clears any existing schedule for citation `8CC21-BB433`.

### Error Responses

In addition to the [standard HTTP error codes](../README.md#error-responses), individual records in a batch may fail validation. Failed records are returned in the `errors` array of a `200 OK` response while valid records are still processed.

#### Model Validation Errors

| Error Message | Cause |
|---------------|-------|
| `ReferenceId is null or empty` | The `referenceId` field was not provided. |
| `Schedule array may be empty but not null` | The `schedule` field was `null` or omitted entirely. Use `[]` for an empty schedule. |
| `{field} is null or empty` | A required field within a schedule entry (`asOf`) is missing. |
| `{field} is not withing the allowed range of 0.01 to 9999.99` | A `totalDue` value is outside the allowed range. |
| `{field} is greater than allowed {max} chars` | The `referenceId` exceeds 50 characters. |

#### Business Logic Errors

| Error Message | Cause |
|---------------|-------|
| `Citation not found` | No citation matches the provided `referenceId` for your account. |
| `Citation already closed` | The citation has been closed and its schedule cannot be modified. |
| `Cannot alter schedule for Citation in Collection` | The citation has been assigned to a collection agency; the schedule cannot be changed. |

#### Example Error Response

```yaml
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{
    "id": "f6a7b8c9-d0e1-2345-fghi-j67890123456",
    "count": 1,
    "errors": [{
        "index": 1,
        "ref": "CLOSED-REF-001",
        "error": "Citation already closed"
    }]
}
```
