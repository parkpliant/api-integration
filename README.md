# ParkPliant - Push API Integration

Our solutions support push of data via a flexible API.

- All data is pushed through HTTPS POST "application/json" to our API.
- All calls are authenticated via Basic authentication ("Authorization" header).
- Each push can contain from 1 to 1000 records of data.
- Our server will generally return one of the following responses
  - 200 – OK
    - This is a success response that means the transaction was accepted and queued for processing
    - It will include "application/json" content with a transaction id from our server and count of records received.
  - 204 - No Content
    - The request was properly formatted except for an empty payload, the request was discarded.
  - 400 – Bad Request
    - The request was improperly formatted, the payload was discarded
  - 401 – Unauthorized
    - The request came without authentication or with invalid credentials

----

## Integration Quickstart

Typical order of operations for a new integration:

1. **Register your callback URLs** via [POST /callbacks](callbacks).  Sandbox callback registration is independent of production — when you move to production, you will register your URLs again against the production base URL.
2. **Register lots** via [POST /lots](lots), or include a `lot` object on the first citation for that lot.  A lot must exist (in either form) before its first citation is accepted.
3. **Push citations** via [POST /citations](citations).
4. **Send Status updates** via [POST /status](status) when the state of a citation changes in your system (paid, voided, amount adjusted, on hold, etc.).
5. **Handle inbound callbacks** for events that occur on our side.  At minimum we recommend implementing the **Payment**, **Correction**, and **Closed** callbacks — see [Callbacks](callbacks) for the full list and recommended-minimum guidance.

----

### Open API (Swagger) Documentation
https://push.sandbox.parkpliant.com/api/swagger/ui (Sandbox)

https://push.parkpliant.com/api/swagger/ui (Production)


## API Endpoints

*Post endpoints accept only HTTPS POST requests with a content-type "application/json".  Attempting to visit them in your browser will usually result in a 404 or 405 error.*

**Citations Post Service**

https://push.parkpliant.com/api/citations

**Status Post Service**

https://push.parkpliant.com/api/status

**Lots Post Service**

https://push.parkpliant.com/api/lots

**Callback Post Service**

https://push.parkpliant.com/api/callbacks

**Images Post Service**

https://push.parkpliant.com/api/images

**Query Get Service**

https://push.parkpliant.com/api/query

----

### Example Request
```yaml
POST /api/citations HTTP/1.1
Host: push.parkpliant.com
Connection: keep-alive
Content-Length: <body-length>
Authorization: Basic <base64(username:password)>
Content-Type: application/json
Accept: application/json, text/json

[{
    "lotCode": "A007",
    "issued": "2021-11-11T15:06:00-4:00",
    "plate": "ABC1234",
    "state": "WA",
    "make": "FORD",
    "noticeOnVehicle": true,
    "amountdue": 10.00,
    "violation": "Overtime"
},{
    "lotCode": "A007",
    "issued": "2021-11-01T02:00:00-7:00",
    "plate": "BCE1234",
    "state": "AZ",
    "make": "TOYOTA",
    "noticeOnVehicle": true,
    "body": "Truck",
    "color": "Silver",
    "amountDue": 42.00,
    "referenceId": "444354357435324",
    "violation": "No Advance Payment",
    "imageUrls": [ "https://s3.amazon.com/my-account/image12728.jpg" ],
    "actionUrls": {
        "paymentUrl": "https://unpaidparking.net/pay?plate=BCE1234"
    },
    "schedule": [
        { "asOf": "2021-11-01T02:00:00-7:00", "totalDue": 42.00 },
        { "asOf": "2021-11-31T02:00:00-7:00", "totalDue": 57.00 }
    ]
},{
    "issued": "2021-12-10T22:00:00-6:00",
    "plate": "CAF132",
    "state": "ID",
    "make": "GMC",
    "noticeOnVehicle": false,
    "body": "SUV",
    "color": "Red",
    "amountDue": 32.50,
    "violation": "Expired Permit",
    "referenceNum": "A123456",
    "referenceId": "3274354364",
    "imageUrls": [ 
        "https://my.site.net/plate-images/img76925.jpg", 
        "https://blob.azure.com/my-company/unpaid/img84279.jpg"
    ],
    "schedule": [
        { "asOf": "2021-12-25T22:00:00-6:00", "totalDue": 65.00 },
        { "asOf": "2022-01-25T22:00:00-6:00", "totalDue": 95.00 }
    ],
        "lot": {
            "code": "FL1012",
            "displayName": "2th and Vine",
            "Address": "1375 East 2th Street",
            "City": "Cleveland",
            "State": "OH",
            "Zip": "44113",
            "ianaTimezone": "America/New_York"
        }
}]
```

In the above example:
- 3 citations were sent, sending 3 posts with one record each is also fine, though less efficient.  The post must always be an array ( [] in JSON ), even if the array only contains one element.

----

### Example Response
```yaml
HTTP/1.1 200 OK
Content-Length: 55
Content-Type: application/json; charset=utf-8

{
    "id": "c31deb20-1069-40c8-b218-e7f7e63ba56d",
    "count": 3,
    "errors": []
}
```
In the above example:
- The `id` is a unique identifier supplied for the transaction, that can be used for transaction tracing.  We recommend recording this value when possible, although is it not required.

----

## Operational Details

### Idempotency
- `referenceId` is the unique key for a citation in your source system.  Posting the same `referenceId` twice will be rejected as a duplicate, so it can safely be used to retry failed pushes without creating duplicates.
- `referenceNum` is intentionally **not** deduped — some clients legitimately re-use ticket or notice numbers across years.

### Batch size and rate
- A single post may contain 1–1000 records.  In practice we recommend **~500 records per post** for citations and Status updates.
- When using `?storeImages=true` (see [About Images](#about-images) under each endpoint), image fetching happens synchronously during the request.  In that mode we recommend limiting to **~50 records per post** to avoid timeouts.
- There is no rate limit on the number of posts per minute.

### Image URL handling
- Without `?storeImages=true`, image URLs are stored as-is and fetched later when letters are being prepared.  The URLs you submit must therefore remain accessible (no expiry, no auth required) for the life of the citation.
- With `?storeImages=true`, our service downloads each image at submission time and serves them from our own storage thereafter — use this when your URLs are short-lived or behind temporary access tokens.

### Error responses
A 400 response uses the same JSON shape as a successful response, with details in the `errors` array:

```json
{
  "id": "c31deb20-1069-40c8-b218-e7f7e63ba56d",
  "count": 0,
  "errors": [
    { "index": 0, "ref": "AB-123", "error": "Lot 'A007' not found" }
  ]
}
```

- `id` — transaction id; please record this value when reporting issues.
- `index` — zero-based position of the offending record in your posted array.
- `ref` — the `referenceId` you supplied for that record (or `null` if not provided).
- `error` — human-readable error message.

----

## Go-Live Checklist

Before we issue production credentials, we ask that you confirm in sandbox that your integration can perform the table-stakes operations.

- [ ] Successfully POST at least one citation to `/api/citations` (HTTP 200, no errors in the response body).
- [ ] Successfully POST a Status update on that citation — either `Paid` or `Void`.
- [ ] Recommended (not gating): receive at least one Payment, Correction, and Closed callback in sandbox.

When moving from sandbox to production, remember:
- Production credentials are issued separately from sandbox credentials.
- Production callback URLs must be registered separately — sandbox callback registration does not carry over.
- The base URL changes from `push.sandbox.parkpliant.com` to `push.parkpliant.com`.




