# Query Get

Retrieves detailed information about one or more previously posted citations. Unlike other endpoints in this API, this is a **GET** request with query parameters rather than a POST with a JSON body.

### Request

```
GET /api/query?id={referenceId}&id={referenceId}
```

| Parameter | Required | Type   | Example(s)    | Description |
|-----------|----------|--------|---------------|-------------|
| `id`      | Yes      | string | `6B547-F4684` | The `referenceId` from the original Citations post. May be repeated to query multiple citations in a single request. |

### Example Request

```yaml
GET /api/query?id=6B547-F4684&id=8CC21-BB433 HTTP/1.1
Host: push.parkpliant.com
Authorization: Basic <base64(username:password)>
Accept: application/json
```

### Response Fields

The response is a JSON array of citation objects. Each object contains the following fields:

| Field              | Type     | Example(s)                   | Description |
|--------------------|----------|------------------------------|-------------|
| `referenceId`      | string   | `6B547-F4684`                | The internal reference ID from the original post. |
| `referenceNum`     | string   | `T-12345`                    | The human-readable reference/citation number. |
| `plate`            | string   | `ABC1234`                    | The license plate. |
| `state`            | string   | `WA`                         | The license plate state. |
| `make`             | string   | `FORD`                       | The vehicle make. |
| `body`             | string   | `SUV`                        | The vehicle body style. |
| `color`            | string   | `Red`                        | The vehicle color. |
| `vin`              | string   | `*5678`                      | The vehicle VIN (full or last 4 prefixed with `*`). |
| `unpaidParking`    | decimal  | `10.00`                      | The unpaid parking charges amount. |
| `originalAmountDue`| decimal  | `42.00`                      | The original amount due when the citation was posted. |
| `currentAmountDue` | decimal  | `57.00`                      | The current amount due (reflects schedule changes and payments). |
| `violationCode`    | string   | `NP`                         | The violation code from the source system. |
| `violation`        | string   | `No Advance Payment`         | The human-readable violation description. |
| `status`           | string   | `Open`                       | Current status: `Open`, `Hold`, `Paid`, or `Void`. |
| `issued`           | string   | `2021-11-11T15:06:00-4:00`   | The [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601) timestamp when the citation was issued. |
| `lotCode`          | string   | `A007`                       | The lot code where the citation was issued. |
| `trackingId`       | number   | `987654`                     | Internal tracking ID assigned by the system. |
| `imageUrls`        | string[] | *(below)*                    | Array of image URLs associated with the citation. |
| `details`          | array    | *(below)*                    | Array of charge detail line items (if posted via [Details](../details) API). |
| `actionUrls`       | object   |                              | Payment and dispute URLs, if configured. |

### Detail Item Fields

| Field              | Type     | Example(s)                   | Description |
|--------------------|----------|------------------------------|-------------|
| `details[].lotCode`| string   | `A007`                       | The lot code for this charge. |
| `details[].timestamp`| string | `2021-11-11T14:00:00-4:00`   | The timestamp for this charge (adjusted for lot timezone). |
| `details[].amount` | decimal  | `15.00`                      | The charge amount. |
| `details[].duration`| string  | `01:30:00`                   | Duration of the parking session. |

### Example Response

```yaml
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

[{
    "referenceId": "6B547-F4684",
    "referenceNum": "T-12345",
    "plate": "ABC1234",
    "state": "WA",
    "make": "FORD",
    "body": "SUV",
    "color": "Red",
    "vin": null,
    "unpaidParking": 10.00,
    "originalAmountDue": 42.00,
    "currentAmountDue": 57.00,
    "violationCode": "NP",
    "violation": "No Advance Payment",
    "status": "Open",
    "issued": "2021-11-11T15:06:00-4:00",
    "lotCode": "A007",
    "trackingId": 987654,
    "imageUrls": [
        "https://storage.parkpliant.com/images/img12728.jpg",
        "https://storage.parkpliant.com/images/img12729.jpg"
    ],
    "details": [{
        "lotCode": "A007",
        "timestamp": "2021-11-11T14:00:00-4:00",
        "amount": 15.00,
        "duration": "01:30:00"
    }],
    "actionUrls": {
        "paymentUrl": "https://unpaidparking.net/pay?plate=ABC1234",
        "disputeUrl": null
    }
}]
```

> **Note**: If a `referenceId` does not match any citation for your account, it is silently omitted from the response. The result array may contain fewer items than the number of IDs requested.

### Error Responses

| HTTP Status | Cause |
|-------------|-------|
| `200 OK`    | Successful query. Returns a JSON array of matching citations (may be empty `[]` if no matches). |
| `400 Bad Request` | The `id` query parameter was not provided. |
| `401 Unauthorized` | Missing or invalid `Authorization` header. |

#### 400 Bad Request Example

Returned when no `id` parameter is included in the query string.

```yaml
HTTP/1.1 400 Bad Request
Content-Type: application/json; charset=utf-8

{
    "error": "Missing 'id' parameter"
}
```
