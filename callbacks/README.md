# Callbacks Post

This service allows you to post callback URLs and an optional Authorization header for updates.

### Fields
| Field | Required | Type/Format |Max Len| Example(s) | Description|
|-------|----------|-------------|---------|---------|------------|
| `authorization` | No | string |1024| `Basic dXNlcjpwYXNz` | An optional HTTP Authorization header value, including the type (`Basic` or `Bearer`) to send on callbacks. |
| `correctionUrl` | No | string |255| `https://my.uri.net/correct` | The absolute URL for us to post citation data corrections to.  See below |
| `paymentUrl` | No | string |255| `https://my.uri.net/pay` | The absolute URL for us to post citation payment to.  See below |
| `noticeUrl` | No | string |255| `https://my.uri.net/notice` | The absolute URL for us to post notices of payment due (i.e. Letters and SMS).  See below |
| `disputeUrl` | No | string |255| `https://my.uri.net/dispute` | The absolute URL for us to post dispute notifications.  See below |
| `reassignmentUrl` | No | string |255| `https://my.uri.net/reassign` | The absolute URL for us to post reassignments to collection agencies.  See below |
| `workedUrl` | No | string |255| `https://my.uri.net/worked` | The absolute URL for us to post when starting work on a citation.  See below |
| `closedUrl` | No | string |255| `https://my.uri.net/closed` | The absolute URL for the webhook handler for *Closed* callbacks.  See below |
| `undeliverableUrl` | No | string |255| `https://my.uri.net/undeliverable` | The absolute URL for us to post when a mailed notice is returned as undeliverable.  See below |

> **Mail Data Callback:** An additional callback is available for print/mail service providers and government clients who handle their own mailing.  This callback is not generally available due to DPPA restrictions.  See [Mail Data Callback](mail-data.md) for details.

### Example

```json
{
  "authorization": "Basic dXNlcjpwYXNz",
  "correctionUrl": "https://my.uri.net/correct",
  "paymentUrl": "https://my.uri.net/pay",
  "noticeUrl": "https://my.uri.net/notice",
  "disputeUrl": null,
  "reassignmentUrl": null,
  "workedUrl": "https://my.uri.net/worked",
  "undeliverableUrl": "https://my.uri.net/undeliverable"
}
```

---

# Operational Contract

All callbacks share the following behavior.

### HTTP Details
- **Method:** `POST`
- **Content-Type:** `application/json` (UTF-8)
- **Payload format:** Always a JSON array (`[]`), even when delivering a single item.
- **Authorization:** If an `authorization` value was configured, it is sent as the `Authorization` HTTP header on every callback request.

### Batching
Callbacks are sent in batches of up to 100 items per POST request, per client.  If there are more pending items, multiple requests are sent in sequence.  For example, 250 pending items would result in 3 POST requests (100 + 100 + 50).  Each batch is independently tracked — if one batch fails, only that batch's items are retried.

*The maximum batch size is current behavior and subject to change.*

### Frequency
The callback processor runs approximately every 5 minutes.  Callbacks are sent for events that have occurred since the last successful delivery.

*The processing frequency is current behavior and subject to change.*

### Retry Behavior
- **HTTP 2xx** — Callback is marked as delivered and will not be sent again.
- **HTTP 400 (Bad Request)** — Callback is acknowledged and will NOT be retried.  Malformed data is treated as a permanent condition.
- **Any other response (4xx, 5xx, timeout, connection failure)** — Callback remains pending and will be retried on the next processing run.

There is no exponential backoff; retries occur at the normal processing frequency.

### Expected Response
Your endpoint should return HTTP 200 (or any 2xx) to acknowledge receipt.  The response body is not inspected — only the status code matters.

---


# Callback Events

## Payment
We will post a JSON Array of payment updates, when we receive and process payments.

### Fields
| Field | Required | Type/Format |Max Len| Example(s) | Description|
|-------|----------|-------------|---------|---------|------------|
| `referenceId` | Yes | string |50| `6B547-F4684` | The internal reference identifier, unique to your source, that was supplied with the original Citation post. |
| `date` | Yes | string (date) |20| `2021-11-15` | An [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601) date stamp, payment was processed.|
| `amount` | Yes | decimal |-9999,9999| `10.00` | The amount that was paid toward the citation. |


### Example

```json
[{
  "referenceId": "6B547-F4684",
  "date": "2021-11-15",
  "amount": 22.50
},{
  "referenceId": "8CC21-BB433",
  "date": "2021-11-15",
  "amount": 10.00
}]
```

## Correction
We will post a JSON Array of citation corrections when data sent with the citation was entered incorrectly and does not match the photo evidence provided.

### Processing pause after corrections

When a correction callback is sent, all outbound notices (letters and SMS messages) for the affected citation are **paused for 1 hour**. This window gives your system an opportunity to review the corrected data and take action before any new notices are generated.

For example, if a corrected license plate number is sent, your system should verify whether the updated plate had a valid payment or permit at the time the violation was issued. If a match is found — meaning the citation was issued in error — you should call the [Status endpoint](../status/README.md) to set the citation to `Void` before the pause expires. If no action is taken within the 1-hour window, normal notice processing resumes automatically.

### Fields
| Field | Required | Type/Format |Max Len| Example(s) | Description|
|-------|----------|-------------|---------|---------|------------|
| `referenceId` | Yes | string |50| `6B547-F4684` | The internal reference identifier, unique to your source, that was supplied with the original Citation post. |
| `plate` | No | string |20| `ABC123` | The corrected license plate number.|
| `state` | No | string |2| `ID` | The corrected license plate state.|
| `make` | No | string |50| `Toyota` | The corrected vehicle make.|
| `void` | No | boolean || `true` | True if the citation should be voided/waived and closed without payment. |


### Example

```json
[{
  "referenceId": "6B547-F4684",
  "plate": "ABC123",
  "state": "ID",
  "make": "Toyota"
},{
  "referenceId": "6B547-22988",
  "make": "Chrysler"
},{
  "referenceId": "8CC21-BB433",
  "void": true
}]
```


## Notice
We will post a JSON Array of sent notices (letters and SMS messages) to the parker or responsible party. This is the primary means of knowing when letters and text messages are sent.

### Fields
| Field | Required | Type/Format |Max Len| Example(s) | Description|
|-------|----------|-------------|---------|---------|------------|
| `referenceId` | Yes | string |50| `6B547-F4684` | The internal reference identifier, unique to your source, that was supplied with the original Citation post. |
| `type` | Yes | string |10| `letter` | The type of notice communication: `letter` or `sms`.  Other options such as `email` may be supported in the future. |
| `date` | Yes | string (date) |10| `2021-11-15` | An [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601) date stamp, notice was sent.|
| `contentUrl` | No | string |255| `https://storage.net/kfg354` | The full URL that can be requested to pull the content of the communication (usually a PDF).  Most content will be protected and require the same API credentials to be supplied.  Present for `letter` notices; not included for `sms`. |
| `deliverTo` | No | string |255| `33901` | Additional information about the notice delivery.  For `letter` this will contain the first 5 digits of the postal code. For `sms` this will contain the last 4 digits of the phone number. |
| `sequence` | Yes | number || 1 | The letter or SMS sequence number, in the range of 1 - 3. |


### Letter Example

```json
[{
  "referenceId": "6B547-F4684",
  "type": "letter",
  "date": "2021-11-15",
  "contentUrl": "https://storage.net/kfg354",
  "deliverTo": "33901",
  "sequence": 1
},{
  "referenceId": "6C558-F5692",
  "type": "letter",
  "date": "2021-11-15",
  "contentUrl": "https://storage.net/kfg355",
  "deliverTo": "90210",
  "sequence": 2
}]
```

### SMS Example

```json
[{
  "referenceId": "6B547-F4684",
  "type": "sms",
  "date": "2021-11-18",
  "deliverTo": "4567",
  "sequence": 1
},{
  "referenceId": "6C558-F5692",
  "type": "sms",
  "date": "2021-11-18",
  "deliverTo": "8901",
  "sequence": 1
}]
```


## Dispute
We will post a JSON Array of disputes submitted by the customer/parker on our dispute portal. Customers may post additional comments and attach files that can be viewed using the URL.

Currently, only `open` (newly created) disputes are sent via callback.  Dispute update/resolution callbacks (with statuses such as `uphold`, `void`, `reduce`, `moot`) may be supported in the future.

### Fields
| Field | Required | Type/Format |Max Len| Example(s) | Description|
|-------|----------|-------------|---------|---------|------------|
| `uuid` | Yes | string (uuid) |36| `9649c808-6f06-48d2-...` | The unique identifier for this dispute created by our portal. |
| `referenceId` | Yes | string |50| `6B547-F4684` | The internal reference identifier, unique to your source, that was supplied with the original Citation post. |
| `createdUtc` | Yes | string (timestamp) |20| `2023-12-22T13:44:07Z` | An [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601) date/time stamp, that the dispute was created.|
| `status` | Yes | string |10| `open` | The current status of the dispute.  Currently only `open` is sent via callback. |
| `reason` | Yes | string |255| `Already_Paid` | A code indicating a common dispute reason, options are `Not_My_Car`, `Already_Paid`, `Waived`, `Permit`, `Machine_Broken`, `Other`. |
| `text` | No | string |1024|| The full comment entered by the customer when the dispute was submitted. |
| `privateUrl` | Yes | string |255| `https://dispute.net/1234567` | A URL for operator staff to view, comment and resolve the dispute. *DO NOT SHARE WITH CUSTOMER*. |
| `customer.name` | No | string |50| `John Doe` | The name submitted by the customer when creating the dispute. |
| `customer.email` | No | string |255| `johndoe@gmail.com` | The email address submitted by the customer when creating the dispute. |

### Example

```json
[{
  "uuid": "9649c808-6f06-48d2-a583-7cdcb1d2654d",
  "referenceId": "6C558-F5692",
  "createdUtc": "2023-12-22T13:44:07Z",
  "status": "open",
  "reason": "Already_Paid",
  "text": "I paid via the mobile app when I parked.",
  "privateUrl": "https://dispute.net/1234567",
  "customer": {
    "name": "John Doe",
    "email": "johndoe@gmail.com"
  }
}]
```

## Reassignment
We will post a JSON Array of citation reassignments when violations are sent to a collection agent.

### Fields
| Field | Required | Type/Format |Max Len| Example(s) | Description|
|-------|----------|-------------|---------|---------|------------|
| `referenceId` | Yes | string |50| `6B547-F4684` | The internal reference identifier, unique to your source, that was supplied with the original Citation post. |
| `assignedUtc` | Yes | string (timestamp) |20| `2024-07-15T19:19:07Z` | An [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601) date/time stamp, that violation was reassigned and forwarded.|
| `agency` | Yes | string |100| `Kinum` | The name of the collection agency or law firm. |


### Example

```json
[{
  "referenceId": "6B547-22988",
  "assignedUtc": "2024-07-15T19:19:07Z",
  "agency": "Kinum"
}]
```

## Worked
We will post a JSON Array of worked events when processing begins on a citation.

### Fields
| Field | Required | Type/Format |Max Len| Example(s) | Description|
|-------|----------|-------------|---------|---------|------------|
| `referenceId` | Yes | string |50| `6B547-F4684` | The internal reference identifier, unique to your source, that was supplied with the original Citation post. |
| `workedUtc` | Yes | string (timestamp) |20| `2024-07-15T19:19:07Z` | An [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601) date/time stamp, that violation was first worked.|

### Example

```json
[{
  "referenceId": "6B547-22988",
  "workedUtc": "2024-07-15T19:19:07Z"
}]
```

## Closed
We will post a JSON Array of closed events when a citation is closed or resolved.

### Fields
| Field | Required | Type/Format |Max Len| Example(s) | Description|
|-------|----------|-------------|---------|---------|------------|
| `referenceId` | Yes | string |50| `6B547-F4684` | The internal reference identifier, unique to your source, that was supplied with the original Citation post. |
| `closedUtc` | Yes | string (timestamp) |20| `2024-07-15T19:19:07Z` | An [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601) date/time stamp, that the citation was closed.|

### Example

```json
[{
  "referenceId": "6B547-22988",
  "closedUtc": "2024-07-15T19:19:07Z"
}]
```

## Undeliverable
We will post a JSON Array of undeliverable events when a mailed notice is returned by the postal carrier as undeliverable.  Each undeliverable letter produces its own callback — a citation with multiple undeliverable letters will generate a separate event for each.

### Fields
| Field | Required | Type/Format |Max Len| Example(s) | Description|
|-------|----------|-------------|---------|---------|------------|
| `referenceId` | Yes | string |50| `6B547-F4684` | The internal reference identifier, unique to your source, that was supplied with the original Citation post. |
| `undeliverableDate` | Yes | string (date) |10| `2024-07-15` | An [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601) date stamp, that the letter was reported undeliverable by the postal carrier. |
| `reason` | No | string |255| `Attempted - No Access to Delivery Location` | Free-text reason provided by the postal carrier.  May be `null` if no reason was given. |
| `sequence` | Yes | number || 1 | The letter number in the mailing sequence for this citation (e.g. 1 = first notice, 2 = second notice). |

### Example

```json
[{
  "referenceId": "6B547-22988",
  "undeliverableDate": "2024-07-15",
  "reason": "Attempted - No Access to Delivery Location",
  "sequence": 1
},{
  "referenceId": "8CC21-BB433",
  "undeliverableDate": "2024-07-18",
  "reason": "Vacant - Uncollectable",
  "sequence": 2
}]
```
