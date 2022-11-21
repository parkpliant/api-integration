# Callbacks Post

This service allows you to post callback URLs and an optional Authorization header for updates.

### Fields
| Field | Required | Type/Format | Example(s) | Description|
|-------|----------|-------------|---------|------------|
| `authorization` | No | string | `Basic dXNlcjpwYXNz` | An optional HTTP Authorization header value, including the type (`Basic` or `Bearer`) to send on callbacks. |
| `correctionUrl` | No | string | `https://my.uri.net/correct` | The absolute URL for us to post citation data corrections to.  See below |
| `paymentUrl` | No | string | `https://my.uri.net/pay` | The absolute URL for us to post citation payment to.  See below |
| `noticeUrl` | No | string | `https://my.uri.net/noitce` | The absolute URL for us to post notices of payment due (i.e. Letters).  See below |

### Example

```yaml
{
  "authorization": "Basic dXNlcjpwYXNz",
  "correctionUrl": "https://my.uri.net/correct",
  "paymentUrl": "https://my.uri.net/pay",
  "noticeUrl": "https://my.uri.net/noitce"
}
```
 
---


# Callback Events

## Payment
We will post a JSON Array of payment updates, when we receive and process payments. 

### Fields
| Field | Required | Type/Format | Example(s) | Description|
|-------|----------|-------------|---------|------------|
| `referenceId` | Yes | string | `6B547-F4684` | The internal reference identifier, unique to your source, that was supplied with the original Citation post. |
| `date` | Yes | string (date) | `2021-11-15` | An [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601) date stamp, payment was processed.|
| `amount` | Yes | decimal | `10.00` | The amount that was paid toward the citation. |


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
| Field | Required | Type/Format | Example(s) | Description|
|-------|----------|-------------|---------|------------|
| `referenceId` | Yes | string | `6B547-F4684` | The internal reference identifier, unique to your source, that was supplied with the original Citation post. |
| `plate` | No | string | `ABC123` | The corrected license plate number.|
| `state` | No | string | `ID` | The corrected license plate state.|
| `make` | No | string | `Toyota` | The corrected vehicle make.|
| `void` | No | boolean | `true` | True is the citation should be voided/waived and closed without payment. |


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
We will post a JSON Array of sent notices to the parker or responsible party. This is the promary means of knowing when letters are sent.

### Fields
| Field | Required | Type/Format | Example(s) | Description|
|-------|----------|-------------|---------|------------|
| `referenceId` | Yes | string | `6B547-F4684` | The internal reference identifier, unique to your source, that was supplied with the original Citation post. |
| `type` | Yes | string | `Letter` | The type of notice communication.  Usually `letter` or `sms`, other options such `email` may be supported in the future. |
| `date` | Yes | string (date) | `2021-11-15` | An [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601) date stamp, notice was sent.|
| `contentUrl` | No | string | `https://storage.net/kfg354` | The full URL that can be requested to pull the content of the communication (usually a PDF).  Most content will be protected and require the same API credentials to be supplied. |


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

