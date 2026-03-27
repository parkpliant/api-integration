# Details Post

Adds or updates detailed charge line items for a previously posted citation. This is useful for breaking down the total amount due into individual charges, such as multiple parking sessions or fees at different lots.

> **Note**: Posting details **replaces** all existing detail records for the citation. To clear details, post an empty `items` array.

### Fields
| Field              | Required | Type/Format | Max Len     | Example(s)                   | Description |
|--------------------|----------|-------------|-------------|------------------------------|-------------|
| `referenceId`      | Yes      | string      | 50          | `6B547-F4684`                | Internal reference ID from the original Citations post. |
| `items`            | Yes      | array       |             | *(below)*                    | Array of detail line items. May be empty `[]` but must not be `null`. |

### Detail Item Fields

| Field              | Required | Type/Format | Max Len     | Example(s)                   | Description |
|--------------------|----------|-------------|-------------|------------------------------|-------------|
| `items[].lotCode`  | Yes      | string      | 50          | `A007`                       | The lot code where the charge originated. Must be a valid lot for your account. |
| `items[].timestamp`| Yes      | string      |             | `2021-11-11T14:30:00-4:00`   | [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601) timestamp with UTC offset for the charge. |
| `items[].amount`   | Yes      | decimal     | 0–9999      | `15.00`                      | The charge amount for this line item. |
| `items[].duration` | No       | string      |             | `01:30:00`                   | Duration of the parking session in `[D.]HH:MM:SS` format (e.g., `01:30:00` for 1 hour 30 minutes, `1.02:00:00` for 1 day 2 hours). |

### Example

```yaml
POST /api/details HTTP/1.1
Content-Type: application/json

{
    "referenceId": "6B547-F4684",
    "items": [{
        "lotCode": "A007",
        "timestamp": "2021-11-11T14:00:00-4:00",
        "amount": 15.00,
        "duration": "01:30:00"
    },{
        "lotCode": "A007",
        "timestamp": "2021-11-11T16:00:00-4:00",
        "amount": 10.00,
        "duration": "00:45:00"
    }]
}
```

### Example Response

The Details endpoint returns a slightly different response shape than other endpoints, including `added` and `removed` counts instead of a single `count`.

```yaml
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{
    "id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "added": 2,
    "removed": 0,
    "errors": []
}
```

> **Note**: This endpoint accepts a single object (not an array) unlike most other endpoints. It also returns `404 Not Found` if the `referenceId` does not match an existing citation.

### Error Responses

In addition to the [standard HTTP error codes](../README.md#error-responses), the following validation errors may occur.

#### Model Validation Errors

| Error Message | Cause |
|---------------|-------|
| `ReferenceId is null or empty` | The `referenceId` field was not provided. |
| `Items array may be empty but not null` | The `items` field was `null` or omitted entirely. Use `[]` to clear existing details. |
| `LotCode is null or empty` | A detail item is missing the `lotCode` field. |
| `{field} is null or empty` | A required field within a detail item (`timestamp`) is missing. |
| `{field} is not withing the allowed range of 0 to 9999` | An `amount` value is outside the allowed range. |
| `{field} is greater than allowed {max} chars` | A string field exceeds its maximum length. |

#### Business Logic Errors

| Error Message | Cause |
|---------------|-------|
| `Missing LotCode` | The lot code could not be resolved after normalization. |
| `Lot '{lotCode}' not found` | The referenced `lotCode` does not exist under your account. Post the lot via the [Lots](../lots) API first. |

#### 404 Not Found
If the `referenceId` does not match any citation for your account, the endpoint returns `404 Not Found` instead of including the error in the `errors` array.

#### Example Error Response

```yaml
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{
    "id": "b2c3d4e5-f6a7-8901-bcde-f23456789012",
    "added": 0,
    "removed": 0,
    "errors": [{
        "index": 1,
        "ref": null,
        "error": "Lot 'INVALID' not found"
    }]
}
```
