# Images Post

This endpoint supports posting additional images to existing citations.  See below for the detail of both required and supported fields.

### Fields
| Field | Required | Type/Format | Example(s) | Description|
|-------|----------|-------------|---------|------------|
| `referenceId` | Yes | string | `6B547-F4684` | The internal reference identifier, unique to your source, that was supplied with the original Citations post. |
| `imageUrls` | No | string [] | *(below)* | An array of internet accessible URLs for images of the unpaid 

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

### About Images
Our system expects the image URLs to be internet accessible without authentication. If the URLs you submit are short-lived or use temporary access tokens, you can add `?storeImages=true` on the endpoint URL.  This will cause our service to immediacy download the images and store them in our cloud storage.  We also accept [Data URLs](https://developer.mozilla.org/en-US/docs/web/http/basics_of_http/data_urls), for images under 500KB, containing the entire image file.