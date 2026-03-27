# Images Post

This endpoint supports posting additional images to existing citations.  See below for the detail of both required and supported fields.

### Fields
| Field | Required | Type/Format |Max Len| Example(s) | Description|
|-------|----------|-------------|---------|---------|------------|
| `referenceId` | Yes | string |50| `6B547-F4684` | The internal reference identifier, unique to your source, that was supplied with the original Citations post. |
| `imageUrls` | No | string [] |255| *(below)* | An array of internet accessible URLs for images of the unpaid 

### Example

```yaml
[{
    "referenceId": "6B547-F4684",
    "imageUrls": [ 
        "https://my.site.net/plate-images/img76985.jpg", 
        "https://blob.azure.com/my-company/unpaid/img84879.jpg"
    ]
}]

```

### Error Responses

In addition to the [standard HTTP error codes](../README.md#error-responses), individual records in a batch may fail validation. Failed records are returned in the `errors` array of a `200 OK` response while valid records are still processed.

#### Model Validation Errors

| Error Message | Cause |
|---------------|-------|
| `ReferenceId is null or empty` | The `referenceId` field was not provided. |
| `ImageUrls[{i}] is not an absolute, https Uri, or data Uri` | An image URL is not a valid absolute HTTPS URL or data URI. |
| `{field} is greater than allowed {max} chars` | A string field exceeds its maximum length. |
| `{field} is less than required {min} chars` | A string field is shorter than its minimum length. |

#### Business Logic Errors

| Error Message | Cause |
|---------------|-------|
| `Citation not found` | No citation matches the provided `referenceId` for your account. |
| `Inaccessible Image URL '{url}'` | An image URL could not be reached or downloaded (when `?storeImages=true`). |

#### Example Error Response

```yaml
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{
    "id": "e5f6a7b8-c9d0-1234-efgh-i56789012345",
    "count": 0,
    "errors": [{
        "index": 0,
        "ref": "UNKNOWN-REF",
        "error": "Citation not found"
    }]
}
```

### About Images
Our system expects the image URLs to be internet accessible without authentication. If the URLs you submit are short-lived or use temporary access tokens, you can add `?storeImages=true` on the endpoint URL.  This will cause our service to immediacy download the images and store them in our cloud storage.  We also accept [Data URLs](https://developer.mozilla.org/en-US/docs/web/http/basics_of_http/data_urls), for images under 500KB, containing the entire image file.