# Status Post

Updates the status of a previously posted citation. Any status other than `Open` halts processing.

> **Note**: The `payments` array is the preferred method for paid citations and **required** if your contract includes contingency fees.

### Fields
| Field            | Required     | Type/Format | Max Len | Example(s)     | Description |
|------------------|--------------|-------------|---------|----------------|-------------|
| `referenceId`    | Yes          | string      | 50      | `6B547-F4684`  | Internal reference ID from the original Citations post. |
| `newStatus`      | Yes          | string      | 4       | `Paid`         | New status: `Open`, `Hold`, `Paid`, or `Void`. |
| `amountDue`      | No           | decimal     | 0–9999  | `50.00`        | Updated total due (for `Open` status fee changes). |
| `paidAmount`     | No           | decimal     | 0–9999  | `10.00`        | Total paid-to-date (for `Paid` status). Not needed if `payments` provided. |
| `payments`       | No*          | array       |         | *(below)*      | Array of payments received to-date. **If included, must be `[]` (empty array) or contain payments. Do not send `null` or omit if no payments.** |
### Payments Array Fields

| Field          | Required     | Type/Format | Example(s) | Description |
|----------------|--------------|-------------|------------|-------------|
| `payments.date`| Yes (if used)| date        | `2021-08-30`| Payment date in [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601) format (time ignored). |
| `payments.amount`| Yes (if used)| decimal   | `10.00`    | Amount of the specific payment. |
### Example

```yaml
[{
  "referenceId": "6B547-F4684",
  "newStatus": "Void"
},{
  "referenceId": "4C2E-8D83",
  "newStatus": "Paid",
  "paidAmount": 15.00
},{
  "referenceId": "4C2E-8D55",
  "newStatus": "Open",
  "amountDue": 50.00
},{
  "referenceId": "8CC21-BB433",
  "newStatus": "Paid",
  "payments": [{
    "date": "2021-08-30",
    "amount": 10.00
  },{
    "date": "2021-09-15",
    "amount": 15.00
  }]
}]
```

### Payment Processing Notes

- When `newStatus` is `Paid` and only `paidAmount` is provided, a single payment record is inferred from the amount.
- When `newStatus` is `Paid` and `payments` is provided, individual payment records are created. Duplicate payments (matching amount and date) are skipped.
- When `newStatus` is `Paid` with no `paidAmount` or `payments`, the full outstanding balance is assumed paid.
- Setting `newStatus` to `Void` or `Hold` halts all further processing on the citation.
- Setting `newStatus` back to `Open` with a new `amountDue` reactivates the citation at the updated amount.

### Error Responses

In addition to the [standard HTTP error codes](../README.md#error-responses), individual records in a batch may fail validation. Failed records are returned in the `errors` array of a `200 OK` response while valid records are still processed.

#### Model Validation Errors

| Error Message | Cause |
|---------------|-------|
| `ReferenceId is null or empty` | The `referenceId` field was not provided. |
| `NewStatus is not valid` | The `newStatus` value is not one of `Open`, `Hold`, `Paid`, or `Void`. |
| `AmountDue is not withing the allowed range of 0 to 9999` | The `amountDue` value is outside the allowed range. |
| `PaidAmount is not withing the allowed range of 0 to 9999` | The `paidAmount` value is outside the allowed range. |
| `{field} is null or empty` | A required field within a `payments` entry is missing. |
| `Amount is not withing the allowed range of -9999 to 9999` | A payment amount is outside the allowed range. |

#### Business Logic Errors

| Error Message | Cause |
|---------------|-------|
| `Invalid NewStatus` | The `newStatus` value could not be parsed to a valid status enum. |
| `Citation not found` | No citation matches the provided `referenceId` for your account. |
| `Citation already Closed or Voided` | The citation has already been closed or voided and cannot be updated. |
| `Cannot increase AmountDue for Citation in Collection` | The citation has been assigned to a collection agency; the amount due cannot be increased. |

#### Example Error Response

```yaml
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{
    "id": "b8c3d4e5-f6a7-8901-bcde-f12345678901",
    "count": 1,
    "errors": [{
        "index": 0,
        "ref": "UNKNOWN-REF",
        "error": "Citation not found"
    },{
        "index": 2,
        "ref": "6B547-F4684",
        "error": "Citation already Closed or Voided"
    }]
}
```

