# Lots Post

The advance posting of lots is required to process Citations.  See below for the detail of both required and supported fields.

### Fields
| Field | Required | Type/Format | Example(s) | Description|
|-------|----------|-------------|---------|------------|
| `code` | Yes | string | `A007` | A name or code you use to indicate the location in the source system. |
| `displayName` | Yes | string | `1st & Pine` | The customer friendly description of the lot |
| `ianaTimezone` | Yes | string | `America/New_York` | The [IANA canonical time zone](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones) code for the lot |
| `address` | No | string | `123 N 34th St` | The street address of the lot |
| `city` | No | string | `Seattle` | The city where of the lot address |
| `state` | No | string | `WA` | The state or province of the lot address |
| `zip` | No | string | `78912` | The zip or postal code of the lot address  |
| `latitude` | No | decimal | `47.60621` | The latitude of the lot in decimal form |
| `longitude` | No | decimal | `-122.33207` | The longitude of the lot in decimal form |

### Example

```yaml
[{
    "code": "A007",
    "displayName" : "1st & Pine",
    "ianaTimezone": "America/Chicago"
},{
    "code": "Q301",
    "displayName" : "34th & Vine",
    "address" : "123 N 34th St",
    "city" : "Somewhere",
    "state" : "CA",
    "zip" : "78912",
    "ianaTimezone": "America/Los_Angeles"
}]
```


 
