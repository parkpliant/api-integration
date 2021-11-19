# Status Post

This service allows you to post a status update to halt processing on a previously posted unpaid vehicle.

### Fields
| Field | Required | Type/Format | Example(s) | Description|
|-------|----------|-------------|---------|------------|
| `referenceId` | Yes | string | `6B547-F4684` | The internal reference identifier, unique to your source, that was supplied with the original Unpaid post. |
| `newStatus` | Yes | string | `Paid` | `Void` | The new status for the vehicle/citation.  Valid options are `Open`, `Paid` and `Void`.  Any status other than `Open` halts processing. |
| `paidAmount` | No | decimal | `10.00` | The amount paid when the status is updated to Paid, if available. |

### Example

```yaml
[{
    "referenceId": "6B547-F4684",
    "newStatus": "Void"
},{
    "referenceId": "8CC21-BB433",
    "newStatus": "Paid",
    "paidAmount": 10.00
}]
```


 
