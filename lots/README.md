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

### About Images
Our system expects the image URLs to be internet accessible without authentication. If the URLs you submit are short-lived or use temporary access tokens, you can add `?storeImages=true` on the endpoint URL.  This will cause our service to immediacy download the images and store them in our cloud storage.  We also accept [Data URLs](https://developer.mozilla.org/en-US/docs/web/http/basics_of_http/data_urls), for images under 500KB, containing the entire image file.
