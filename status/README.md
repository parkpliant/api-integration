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



 
