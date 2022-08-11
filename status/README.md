# Status Post

This service allows you to post a status update to halt processing on a previously posted citation.  

*Note: The "payments" structure is the preferred method of indication for paid citations and is required if the contract includes a contingency fee.* 

### Fields
| Field | Required | Type/Format | Example(s) | Description|
|-------|----------|-------------|---------|------------|
| `referenceId` | Yes | string | `6B547-F4684` | The internal reference identifier, unique to your source, that was supplied with the original Unpaid post. |
| `newStatus` | Yes | string | `Paid` | The new status for the vehicle/citation.  Valid options are `Open`, `Paid` and `Void`.  Any status other than `Open` halts processing. |
| `amountDue` | No | decimal | `15.00` | The total amount due now when the status `Open`. |
| `paidAmount` | No | decimal | `10.00` | The total amount paid-to-date when the status is updated to `Paid`, if available.  Not required if payments array is supplied. |
| `payments` | No | {payment} [] | *(below)* | An array of payments received to-date for the citation. |
| `payments.date` | Yes* | date | `2021-08-30` | The date the payment was received in [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601) format, time of day is ignored. |
| `payments.amount` | Yes* | decimal | `10.00` | The amount of the specific payment. |


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


 
