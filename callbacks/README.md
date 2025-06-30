# Callbacks Post

This service allows you to post callback URLs and an optional Authorization header for updates.

### Fields
| Field | Required | Type/Format |Max Len| Example(s) | Description|
|-------|----------|-------------|---------|---------|------------|
| `authorization` | No | string |1024| `Basic dXNlcjpwYXNz` | An optional HTTP Authorization header value, including the type (`Basic` or `Bearer`) to send on callbacks. |
| `correctionUrl` | No | string |255| `https://my.uri.net/correct` | The absolute URL for us to post citation data corrections to.  See below |
| `paymentUrl` | No | string |255| `https://my.uri.net/pay` | The absolute URL for us to post citation payment to.  See below |
| `noticeUrl` | No | string |255| `https://my.uri.net/notice` | The absolute URL for us to post notices of payment due (i.e. Letters).  See below |
| `disputeUrl` | No | string |255| `https://my.uri.net/dispute` | The absolute URL for us to post dispute notifications.  See below |
| `reassignmentUrl` | No | string |255| `https://my.uri.net/reassign` | The absolute URL for us to post reassignments to collection agencies.  See below |
| `workedUrl` | No | string |255| `https://my.uri.net/worked` | The absolute URL for us to post when starting work on a Citation.  See below |
| `closedUrl` | No | string |255| `https://my.uri.net/closed` | The url for the webook handler for *Closed* callbacks.  See below |

### Example

```yaml
{
  "authorization": "Basic dXNlcjpwYXNz",
  "correctionUrl": "https://my.uri.net/correct",
  "paymentUrl": "https://my.uri.net/pay",
  "noticeUrl": "https://my.uri.net/notice",
  "disputeUrl": null,
  "reassignmentUrl": null,
  "workedUrl": "https://my.uri.net/worked",
}
```
 
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

```yaml
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

### Fields
| Field | Required | Type/Format |Max Len| Example(s) | Description|
|-------|----------|-------------|---------|---------|------------|
| `referenceId` | Yes | string |50| `6B547-F4684` | The internal reference identifier, unique to your source, that was supplied with the original Citation post. |
| `plate` | No | string |20| `ABC123` | The corrected license plate number.|
| `state` | No | string |2| `ID` | The corrected license plate state.|
| `make` | No | string |50| `Toyota` | The corrected vehicle make.|
| `void` | No | boolean || `true` | True is the citation should be voided/waived and closed without payment. |


### Example

```yaml
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
We will post a JSON Array of sent notices to the parker or responsible party. This is the primary means of knowing when letters are sent.

### Fields
| Field | Required | Type/Format |Max Len| Example(s) | Description|
|-------|----------|-------------|---------|---------|------------|
| `referenceId` | Yes | string |50| `6B547-F4684` | The internal reference identifier, unique to your source, that was supplied with the original Citation post. |
| `type` | Yes | string |10| `letter` | The type of notice communication.  Usually `letter` or `sms`, other options such `email` may be supported in the future. |
| `date` | Yes | string (date) |10| `2021-11-15` | An [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601) date stamp, notice was sent.|
| `contentUrl` | No | string |255| `https://storage.net/kfg354` | The full URL that can be requested to pull the content of the communication (usually a PDF).  Most content will be protected and require the same API credentials to be supplied. |
| `deliverTo` | No | string |255| `33901` | Additional information about the notice delivery.  For `letter` this will contain the first 5-digits of the postal code. For `sms` this will contains the last 4-digits of the phone number. |
| `sequence` | Yes | number || 1 | The letter or SMS sequence number, in the range of 1 - 3. |


### Example

```yaml
[{
  "referenceId": "6B547-F4684",
  "type": "letter",
  "date": "2021-11-15",
  "contentUrl": "https://storage.net/kfg354",
},{
  "referenceId": "6C558-F5692",
  "type": "letter",
  "date": "2021-11-15",
  "contentUrl": "https://storage.net/kfg355",
}]
```


## Dispute
We will post a JSON Array of disputes submitted by the customer/parker on our dispute portal. Customers may post additional comments and attach files that can be viewed using the URL.

### Fields
| Field | Required | Type/Format |Max Len| Example(s) | Description|
|-------|----------|-------------|---------|---------|------------|
| `uuid` | Yes | string (uuid) |36| `9649c808-6f06-48d2-...` | The unique identifier for this dispute created by our portal. |
| `referenceId` | Yes | string |50| `6B547-F4684` | The internal reference identifier, unique to your source, that was supplied with the original Citation post. |
| `createdUtc` | Yes |20| string (timestamp) | `2023-12-22T13:44:07Z` | An [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601) date/time stamp, that the dispute was created.|
| `status` | Yes | string |10| `open` | The current status of the dispute: `open`, `uphold`, `void`, `reduce`, `moot`.  All disputes are initially `open` any other status is a closed status.
| `reason` | Yes | string |255| `Already_Paid` | A code indicating a common dispute reason, options are `Not_My_Car`, `Already_Paid`, `Waived`, `Permit`, `Machine_Broken`, `Other`. |
| `text` | No | string |1024| The full comment entered by the customer when the dispute was submitted. |
| `privateUrl` | Yes | string |255| `https://dispute.net/1234567` | A URL for operator staff to view, comment and resolve the dispute. *DO NOT SHARE WITH CUSTOMER*. |
| `customer.name` | No | string |50| `John Doe` | The name submitted by the customer when creating the dispute. |
| `customer.email` | No | string |255| `johndoe@gmail.com` | The email address submitted by the customer when creating the dispute. |
| `closedUtc` | No | string (timestamp) || `2024-01-15T19:01:307Z` | An [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601) date/time stamp, that the dispute was closed.|
| `reduceTo` | No | decimal |0,9999| `10.00` | The amount that was the operator reduced the amount owned to, if status is `reduce`. |

### Example

```yaml
[{
  "uuid": "9649c808-6f06-48d2-a583-7cdcb1d2654d",
  "referenceId": "6C558-F5692",
  "createdUtc": "2023-12-22T13:44:07Z",
  "status": "open",
  "reason": "Already_Paid",
  "text": "I paid at via the mobile app when i parked.",
  "url": "https://dispute.net/1234567",
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
| `assignedUtc` | Yes | string (timestamp) |20| `2024-07-15T19:19:307Z` | An [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601) date/time stamp, that violation was reassigned and forwarded.|
| `agency` | Yes | string |100| `Kinum` | The name of the collection agency or law firm. |


### Example

```yaml
[{
  "referenceId": "6B547-22988",
  "assignedUtc": "2024-07-15T19:19:307Z",
  "agency": "Kinum"  
}]
```

## Worked
We will post a JSON Array of worked events violations. 

### Fields
| Field | Required | Type/Format |Max Len| Example(s) | Description|
|-------|----------|-------------|---------|---------|------------|
| `referenceId` | Yes | string |50| `6B547-F4684` | The internal reference identifier, unique to your source, that was supplied with the original Citation post. |
| `workedUtc` | Yes | string (timestamp) |20| `2024-07-15T19:19:307Z` | An [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601) date/time stamp, that violation was first worked.|

### Example

```yaml
[{
  "referenceId": "6B547-22988",
  "workedUtc": "2024-07-15T19:19:307Z"
}]
```
