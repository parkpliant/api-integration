# Lots Post

The advance posting of lots is required to process Citations.  See below for the detail of both required and supported fields.  Lots with the same `code` may be posted again to update the data.

### Fields
| Field | Required | Type/Format |Max Len| Example(s) | Description|
|-------|----------|-------------|---------|---------|------------|
| `code` | Yes | string |50| `A007` | A name or code you use to indicate the location in the source system. |
| `displayName` | Yes | string |50| `1st & Pine` | The customer friendly description of the lot |
| `ianaTimezone` | Yes | string |50| `America/New_York` | The [IANA canonical time zone](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones) code for the lot |
| `address` | No | string |100| `123 N 34th St` | The street address of the lot |
| `city` | No | string |50| `Seattle` | The city where of the lot address |
| `state` | No | string |2| `WA` | The state or province of the lot address |
| `zip` | No | string |10| `78912` | The zip or postal code of the lot address  |
| `latitude` | No | decimal || `47.60621` | The latitude of the lot in decimal form |
| `longitude` | No | decimal || `-122.33207` | The longitude of the lot in decimal form |
| `signImageUrls` | No | string [] |255| *(below)* | An array of internet accessible URLs for images of the signage posted in the location. |
| `group.id` | No | string |50| `825` | A short identifier for the group/market for large operator integration. |
| `group.name` | No | string |50| `San Diego` | The name for the group/market for large operator integration. |

### Example

```yaml
[{
    "code": "A007",
    "displayName": "1st & Pine",
    "ianaTimezone": "America/Chicago"
},{
    "code": "Q301",
    "displayName": "34th & Vine",
    "address": "123 N 34th St",
    "city": "Somewhere",
    "state": "CA",
    "zip": "78912",
    "ianaTimezone": "America/Los_Angeles",
    "signImageUrls": [
      "https://my.website.net/lot/a007/sign.jpg"
    ]
    "group": {
        "id": "825",
        "name": "San Diego"
    }
}]
```

### Error Responses

In addition to the [standard HTTP error codes](../README.md#error-responses), individual records in a batch may fail validation. Failed records are returned in the `errors` array of a `200 OK` response while valid records are still processed.

#### Model Validation Errors

| Error Message | Cause |
|---------------|-------|
| `Code is null or empty` | The `code` field was not provided. |
| `DisplayName is null or empty` | The `displayName` field was not provided. |
| `IanaTimezone is null or empty` | The `ianaTimezone` field was not provided. |
| `'{value}' is not a valid IANA time zone` | The `ianaTimezone` value is not a recognized [IANA time zone](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones). |
| `SignImageUrls[{i}] is not an absolute, https Uri, or data Uri` | A sign image URL is not a valid absolute HTTPS URL or data URI. |
| `{field} is greater than allowed {max} chars` | A string field exceeds its maximum length (e.g., `code` over 50 chars). |
| `{field} is less than required {min} chars` | A string field is shorter than its minimum length. |
| `{field} is not withing the allowed range of {min} to {max}` | `latitude` or `longitude` is outside the range of -180 to 180. |

#### Business Logic Errors

| Error Message | Cause |
|---------------|-------|
| `Missing Lot Code` | The lot code could not be resolved after normalization. |
| `Inaccessible Image URL '{url}'` | A sign image URL could not be reached or downloaded (when `?storeImages=true`). |

#### Example Error Response

```yaml
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{
    "id": "d4e5f6a7-b8c9-0123-defg-h45678901234",
    "count": 1,
    "errors": [{
        "index": 0,
        "ref": null,
        "error": "'US/Pacific' is not a valid IANA time zone"
    }]
}
```

### About Images
Our system expects the image URLs to be internet accessible without authentication. If the URLs you submit are short-lived or use temporary access tokens, you can add `?storeImages=true` on the endpoint URL.  This will cause our service to immediacy download the images and store them in our cloud storage.  We also accept [Data URLs](https://developer.mozilla.org/en-US/docs/web/http/basics_of_http/data_urls), for images under 500KB, containing the entire image file.
