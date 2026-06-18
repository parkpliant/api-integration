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
| `payments`       | No*          | array       |         | *(below)*      | Array of payments received to-date.  Either omit the field entirely, or include a non-null array — `[]` is acceptable when there are no payments to report.  **Do not send `null`.** |
### Payments Array Fields

| Field          | Required     | Type/Format | Example(s) | Description |
|----------------|--------------|-------------|------------|-------------|
| `payments.date`| Yes (if used)| date        | `2021-08-30`| Payment date in [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601) format (time ignored). |
| `payments.amount`| Yes (if used)| decimal   | `10.00`    | Amount of the specific payment. |

### When to use each `newStatus`

- **`Open`** — the citation is active and eligible for processing.  Used to **release a citation from `Hold`**, or paired with an updated `amountDue` to adjust the balance — see [Adjusting `amountDue`](#adjusting-amountdue) below.  Note this only works on a citation that is still active; it **cannot** revive a citation that has already been settled as `Paid` or `Void` (see [Terminal states](#terminal-states-and-allowed-transitions)).
- **`Hold`** — pause processing of the citation without closing it.  Use this when something is in review on your side and you want to temporarily stop the mail campaign without voiding the notice.
- **`Paid`** — the citation has been paid in full.  Most parker payments flow through the ParkPliant payment portal and surface to your system via the [Payment callback](../callbacks#payment) — use `Paid` here when a payment is settled directly in your system instead.  Include `paidAmount` or `payments[]` to record the payment detail.
- **`Void`** — the citation should no longer be processed and will not be mailed.  Use this when a citation was issued in error, or when re-validation after a [Correction callback](../callbacks#correction) reveals that the corrected plate had a valid parking session.  Pushing `Void` within the 1-hour pause window after a Correction callback is what prevents an erroneous mailer from going out.

### Terminal states and allowed transitions

`Paid` and `Void` are **terminal**.  Once a citation is settled as `Paid` (closed) or `Void`, further Status updates against it are rejected with the error `Citation already Closed or Voided`.  There is no API path to reopen a closed or voided citation — if one was settled in error, contact your account representative.

Practically, that means:

| Current state | `Open` | `Hold` | `Paid` | `Void` |
|---------------|:------:|:------:|:------:|:------:|
| **Open**  | adjust amount | ✅ | ✅ | ✅ |
| **Hold**  | ✅ release | — | ✅ | ✅ |
| **Paid**  | ❌ | ❌ | ❌ | ❌ |
| **Void**  | ❌ | ❌ | ❌ | ❌ |

Other notes:
- An update that would not change anything (same status and same `amountDue`) is accepted but silently ignored — it does **not** count as an error and does **not** appear in the `errors` array.
- A citation that has been forwarded to a collection agency cannot have its balance **increased** via Status — that returns `Cannot increase AmountDue for Citation in Collection`.  Decreases (e.g. recording a payment) are still allowed.
- If the `referenceId` does not match a citation for your account, the record returns the error `Citation not found`.

### Adjusting `amountDue`

You can push an updated `amountDue` along with `newStatus: Open` to adjust the balance owing on a citation.  This is the mechanism for partial payments (where the citation isn't yet fully paid), fee waivers, court-ordered adjustments, or any other balance change that doesn't fit the predefined `schedule[]` from the original Citations post.

Interaction with schedules: if the citation has a `schedule[]` from the original post, pushing a new `amountDue` overrides the current applied amount, but **does not prevent future schedule entries from being applied**.  See [About Schedules](../citations#about-schedules) for details.

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



 
