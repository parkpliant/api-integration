# Schedules Post

Replaces the fee **schedule** on an existing citation — the set of as-of dates and effective amounts that drive early-payment discounts or escalating fees over time.  Use this to add, change, or clear a schedule *after* the citation was originally posted.

For the underlying schedule mechanics (how and when entries are applied, how they interact with `Status` updates), see [About Schedules](../citations#about-schedules) under the Citations endpoint.

> **⚠️ This replaces the entire schedule — it is not additive.**
> Every POST deletes **all** existing schedule entries on the citation and replaces them with exactly what you send.  To change one entry, send the complete schedule.  Send an **empty array** (`[]`) to remove the schedule entirely.  Do **not** send `null`.

### Fields

| Field | Required | Type/Format | Max Len | Example(s) | Description |
|-------|----------|-------------|---------|------------|-------------|
| `referenceId` | Yes | string | 50 | `6B547-F4684` | The `referenceId` of the citation to update, from the original [Citations](../citations) post. |
| `schedule` | Yes | array | | *(below)* | The complete set of schedule entries for the citation.  Replaces any existing schedule.  May be an empty array (`[]`) to clear the schedule, but must not be `null`. |
| `schedule[].asOf` | Yes (per entry) | string (timestamp) | | `2021-11-30T02:00:00-07:00` | [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601) timestamp, including UTC offset, when this amount becomes effective. |
| `schedule[].totalDue` | Yes (per entry) | decimal | 0.01–9999.99 | `57.00` | Total amount due as of `asOf` (a replacement value, not additive). |

### Behavior

- **Immediate application:** after the schedule is saved, the most recent entry whose `asOf` is now or in the past is applied right away, setting the citation's current amount due.  Entries with a future `asOf` are applied automatically within an hour of that time.
- **Empty schedule:** sending `[]` deletes all existing entries.  The current amount due is left as-is (no past/now entry is applied).
- **Terminal citations:** a citation that is already `Paid`/closed or `Void` is rejected with `Citation already Closed or Voided` (see [Status terminal states](../status#terminal-states-and-allowed-transitions)).
- **In collections:** a citation forwarded to a collection agency cannot have its schedule altered — that returns `Cannot alter schedule for Citation in Collection`.
- **Not found:** if the `referenceId` does not match a citation for your account, the record returns `Citation not found`.

### Example Request

```yaml
POST /api/schedules HTTP/1.1
Host: push.parkpliant.com
Authorization: Basic <base64(username:password)>
Content-Type: application/json
Accept: application/json

[{
  "referenceId": "6B547-F4684",
  "schedule": [
    { "asOf": "2021-11-01T02:00:00-07:00", "totalDue": 42.00 },
    { "asOf": "2021-11-30T02:00:00-07:00", "totalDue": 57.00 }
  ]
},{
  "referenceId": "8CC21-BB433",
  "schedule": []
}]
```

In the above example the first citation's schedule is replaced with two entries, and the second citation's schedule is cleared.  As with the other Post services, the body must always be an array (`[]`), even for a single record.

### Example Response

```json
{
  "id": "c31deb20-1069-40c8-b218-e7f7e63ba56d",
  "count": 2,
  "errors": []
}
```

- `id` — transaction id; please record this value when reporting issues.
- `count` — the number of citations whose schedules were updated.
- `errors` — an array of per-record errors (same shape as the other endpoints); see [Error responses](../#error-responses).
