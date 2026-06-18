# Query Get

A read-only endpoint to look up the current state of one or more citations you previously posted, by `referenceId`.  This is useful for reconciliation — confirming a citation exists on our side, checking its current status and balance, or retrieving the images and fee history we hold.

- **Method:** `GET`
- **Authentication:** Basic authentication (same credentials as the Post services).
- **Content-Type (response):** `application/json`

> Unlike the Post services, Query takes its input as **query-string parameters**, not a JSON body.

### Request Parameters

| Parameter | Required | Repeatable | Description |
|-----------|----------|------------|-------------|
| `id` | Yes | Yes | The `referenceId` of the citation to look up.  Supply the parameter multiple times (`?id=A&id=B`) to look up several citations in one request. |

- If `id` is omitted entirely, the request returns `400` with the message `Missing 'id' parameter`.
- Any `id` that does not match a citation **for your account** is **silently skipped** — it is simply absent from the returned array.  A request for 3 ids where only 2 exist returns 2 objects, not an error.  An empty result is therefore normal and means "none of the supplied ids were found."

### Example Request

```
GET /api/query?id=6B547-F4684&id=3274354364 HTTP/1.1
Host: push.parkpliant.com
Authorization: Basic <base64(username:password)>
Accept: application/json
```

### Response Fields

The response is always a JSON **array** of citation objects (one per found `id`).

| Field | Type/Format | Description |
|-------|-------------|-------------|
| `referenceId` | string | The `referenceId` you supplied on the original Citations post. |
| `referenceNum` | string | The human-readable reference/citation number (the synthetic `T-<id>` placeholder if you did not supply one). |
| `lotCode` | string | The lot code the citation is associated with. |
| `clientName` | string | The display name of the client/operator the citation belongs to. |
| `issued` | string (timestamp) | [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601) issued timestamp, including the original UTC offset. |
| `plate` | string | License plate number. |
| `state` | string | License plate state. |
| `make` | string | Vehicle make. |
| `body` | string | Vehicle body style. |
| `color` | string | Vehicle color. |
| `vin` | string | Vehicle VIN, if supplied. |
| `unpaidParking` | decimal | The original unpaid parking amount, if supplied. |
| `originalAmountDue` | decimal | The total amount due when the citation was first posted. |
| `currentAmountDue` | decimal | The current balance due, after any Status updates, payments, or applied schedule entries. |
| `violationCode` | string | The violation code, if supplied. |
| `violation` | string | The human-readable violation description. |
| `lotEntryTime` | string (timestamp) | Lot entry time, if supplied. |
| `lotExitTime` | string (timestamp) | Lot exit time, if supplied. |
| `trackingId` | number | Our internal numeric tracking id for the citation. |
| `status` | string | The current status of the citation (e.g. `Open`, `Hold`, `Paid`, `Void`). |
| `closedUtc` | string (timestamp) | UTC timestamp the citation was closed, or `null` if still open. |
| `details` | array | Session/detail records associated with the citation — see below. |
| `imageUrls` | string[] | URLs of the images currently stored for the citation. |
| `actionUrls.paymentUrl` | string | The payment URL stored for the citation, if any. |
| `actionUrls.disputeUrl` | string | The dispute URL stored for the citation, if any. |

#### `details[]` fields

| Field | Type/Format | Description |
|-------|-------------|-------------|
| `lotCode` | string | The lot code for this detail record. |
| `timestamp` | string (timestamp) | [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601) timestamp for the detail, in the citation's local offset. |
| `amount` | decimal | The amount associated with this detail record. |
| `duration` | string (duration) | The duration associated with this detail record, if any. |

### Example Response

```json
[{
  "referenceId": "6B547-F4684",
  "referenceNum": "C123456",
  "lotCode": "A007",
  "clientName": "Acme Parking",
  "issued": "2021-11-11T15:06:00-04:00",
  "plate": "ABC1234",
  "state": "WA",
  "make": "FORD",
  "body": "SUV",
  "color": "Red",
  "vin": null,
  "unpaidParking": 10.00,
  "originalAmountDue": 20.00,
  "currentAmountDue": 0.00,
  "violationCode": "NP",
  "violation": "No Advance Payment",
  "lotEntryTime": null,
  "lotExitTime": null,
  "trackingId": 884512,
  "status": "Paid",
  "closedUtc": "2021-11-20T18:04:00Z",
  "details": [],
  "imageUrls": [
    "https://storage.parkpliant.com/photos/img001.jpg"
  ],
  "actionUrls": {
    "paymentUrl": "https://unpaidparking.net/pay?plate=ABC1234",
    "disputeUrl": null
  }
}]
```
