# Citations Post

The posting of citations is the basis for our services.  See below for the detail of both required and supported fields.

### Fields
| Field | Required | Type/Format |Max Len| Example(s) | Description|
|-------|----------|-------------|-------|---------|------------|
| `lotCode` | Yes* | string |50| `A007` | A name or code you use to indicate the location where the unpaid vehicle is parked. These codes will be exchanged ahead of time along with other location data using the [Lots](../lots) api. |
| `issued` | Yes | string || `2021-11-11T15:06:00-4:00` | An [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601) local time stamp, including UTC offset, when the citation was issued.|
| `plate` | Yes | string |20| `ABC-1234` | The license plate for the unpaid vehicle. |
| `state` | Yes | string |2| `WA` | The state or province that issued the license plate. (see `plate`) |
| `make` | No* | string |50| `Ford` | The make/manufacturer of the unpaid vehicle.  This field can accept [NCIC VMA Codes](https://wilenet.widoj.gov/sites/default/files/public_files-2021-01/ncic_code_manual_-_dec_31_2020.pdf) or proper names. Surcharges may apply if this field is not provided.  |
| `body` | No | string |50| `Truck` | A short term for the body-style of the vehicle, such as: 'Truck', 'SUV', '2-door', '4-door', etc.  The value should be meaningful to a human reader. |
| `color` | No | string |50| `Red` | The color of the vehicle. |
| `vin` | No | string |20| `*5678` | Can contain the full vehicle VIN or the last 4 digits of the VIN if prefixed with `*`. |
| `unpaidParking` | No | decimal |0.01,9999| `10.00` | The unpaid amount of the parking charges. |
| `amountDue` | Yes | decimal |0.01,9999.99| `20.00` | The total amount due on this citation. |
| `referenceNum` | No | string |50| `C123456` | A reference number for the unpaid vehicle, usually a citation number issued from an internal system |
| `referenceId` | No* | string |50| `6B547-F4684` | An internal reference identifier, unique to your source system, to be used future updates.  Calls to update [Status](..\status) require a matching value. |
| `violationCode` | No | string |20| `NP` | The violation code in the sources system. |
| `violation` | Yes | string |50| `No Advance Payment` | The human-readable violation description. |
| `lotEntryTime` | No | string || `2021-11-11T14:39:00-4:00` | An [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601) local time stamp, including UTC offset, when the vehicle entered the lot. |
| `lotExitTime` | No | string || `2021-11-11T15:05:00-4:00` | An [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601) local time stamp, including UTC offset, when the vehicle existed the lot. |
| `imageUrls` | No | string [] |255| *(below)* | An array of internet accessible URLs for images of the unpaid vehicle in the location. |
| `actionUrls.paymentUrl` | No | string |255| `https://my.url.net/` | An internet accessible URL for the customer to make a payment. This will be displayed on the default landing page. |
| `actionUrls.diusputeUrl` | No | string |255| `https://my.url.net` | An internet accessible URL for the customer to submit a dispute. This will be displayed on the default landing page. |
| `group.id` | No | string |50| `825` | A short identifier for the group/market for large operator integration. If supplied, this needs to match the value on the Lot. |
| `group.name` | No | string |50| `San Diego` | The name for the group/market for large operator integration. If supplied, this needs to match the value on the Lot. |
| `schedule[].asOf` | Maybe | string || `2021-11-11T15:06:00-4:00` | An [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601) local time stamp, including UTC offset, when the scheduled amount should be effective.|
| `schedule[].totalDue` | Maybe | decimal || `20.00` | The total amount due on this citation as of the `asOf` date/time for this schedule entry.  This is a replacement value not an additive value. |
| `NoticeOnVehicle`|Yes*|Int||1|Indicates if a paper notice was left on the vehicle. See below for details.|
|`lot`|Yes*|Object|||The lot Object. Required if lotCode is not present.  Both lotCode and Lot object is not allowed, needs to one or the other  |
| `lot.code`|Yes*|string|50|A007|The unique code for the lot in the source system(i.e.'L204).  This can but any length up to 50 characters|
| `lot.DisplayName`|Yes*|string|50|1st and Pine|The display name of the location for customers (i.e. '1st & Pine')|
| `lot.Address`|No|string|100|123 N Street|The street address of the lot|
| `lot.City`|No|string|50|Seattle|The city of the lot address|
| `lot.State`|Yes*|string|2|WA|The state code of the lot address|
| `lot.Zip`|No|string|10|1st and Pine|The postal code of the lot address|
| `lot.ianaTimezone` | Yes* | string |50| `America/New_York` | The [IANA canonical time zone](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones) code for the lot |
| `lot.Latitude`|No|Decimal|-180,180|47.60621|The latitude of the lot in decimal form|
| `lot.Longitude`|No|Decimal|-180,180|-112.33207|The longitude of the lot in decimal form|

`*You must specify either the lotCode field or the lot object in your citation payload, but not both.`   
- If you are using the Lots API, provide the lotCode string to indicate the location.
- If you are not using the Lots API, provide the full lot object with required details.
- `Including both` lotCode `and` lot `in the same citation is not permitted and will result in a 400 error`.
- At least one (but not both) of these fields is required for each citation.

"Either lotCode or lot collection are required, but cannot both be present in the collection. If both are part of the citation payload then a 400 error code will be returned in the response package..."

The system will validate your submission and reject any citation that includes both fields or omits both, ensuring that only one method of specifying the lot location is used.

### Example

```yaml
[{
    "lotCode": "A007",
    "issued": "2021-11-11T15:06:00-4:00",
    "plate": "ABC1234",
    "state": "WA",
    "make": "Ford",
    "noticeOnVehicle": 1,
    "amountdue": 10.00,
    "violation": "Overtime"
},{
    "lotCode": "A007",
    "issued": "2021-11-01T02:00:00-7:00",
    "plate": "BCE1234",
    "state": "AZ",
    "make": "Toyota",
    "noticeOnVehicle": 1,
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
    "noticeOnVehicle": 1,
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

### About Images
Our system expects the image URLs to be internet accessible without authentication. If the URLs you submit are short-lived or use temporary access tokens, you can add `?storeImages=true` on the endpoint URL.  This will cause our service to immediacy download the images and store them in our cloud storage.  We also accept [Data URLs](https://developer.mozilla.org/en-US/docs/web/http/basics_of_http/data_urls), for images under 500KB, containing the entire image file.

### About Schedules
Schedules are an optional feature that can be used if you offer discounts for early payment, that expire after a period of time.  If you use the schedule feature, then both the `asOf` and `totalDue` are required for each entry.  There is no practical limit for the number of schedule entires supported for a Citation.  Scheduels are applied within an hour of the `asOf` time by our system.  Schedule entries may increase or reduce the amount due.  Any call to *Status*, with a new `amountDue`, will override the amount set by the latest applied schedule, but will not prevent future schedules from being applied.  

You may supply a schedule entry for the issued date/time of the Citation with the initial amount due, or omit this an send only future changes.  Regardless, the `amountDue` on the Citation must be the currect amout due (with any early pay discounts) as of the time the Citation is sent to us.