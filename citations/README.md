
# Citations Post

The posting of citations is the basis for our services. See below for details of both required and supported fields.

> **Note**: Lot identification is conditional. Each citation must include **either** `lotCode` **or** the `lot` object, but **not both**. Providing both or neither will result in a `400` error.

### Fields

| Field                  | Required          | Type/Format | Max Len | Example(s)                        | Description |
|------------------------|-------------------|-------------|---------|-----------------------------------|-------------|
| `lotCode`              | Conditional       | string`(uppercase)`      | 50      | `A007`                            | A name or code indicating the location where the unpaid vehicle is parked. These codes will be exchanged ahead of time using the [Lots](../lots) API. |
| `issued`               | Yes               | string      |         | `2021-11-11T15:06:00-4:00`        | An [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601) local timestamp, including UTC offset, when the citation was issued. |
| `plate`                | Yes               | string`(uppercase)`      | 20      | `ABC-1234`                        | The license plate for the unpaid vehicle. |
| `state`                | Yes               | string`(uppercase)`      | 2       | `WA`                              | The state or province that issued the license plate. |
| `make`                 | No                | string`(uppercase)`      | 50      | `Ford`                            | The make/manufacturer of the unpaid vehicle. Accepts [NCIC VMA Codes](https://wilenet.widoj.gov/sites/default/files/public_files-2021-01/ncic_code_manual_-_dec_31_2020.pdf) or proper names. Surcharges may apply if not provided.  If unknown leave blank or do not submit the element.|
| `body`                 | No                | string      | 50      | `Truck`                           | A short term for the body style of the vehicle (e.g., `Truck`, `SUV`, `2-door`, `4-door`). |
| `color`                | No                | string      | 50      | `Red`                             | The color of the vehicle. |
| `vin`                  | No                | string`(uppercase)`      | 20      | `*5678`                           | Full vehicle VIN or last 4 digits prefixed with `*`. |
| `unpaidParking`        | No                | decimal     | 0.01–9999 | `10.00`                          | The unpaid amount of the parking charges. |
| `amountDue`            | Yes               | decimal     | 0.01–9999.99 | `20.00`                      | The total amount due on this citation. |
| `referenceNum`         | No                | string`(uppercase)`      | 50      | `C123456`                         | A reference number for the unpaid vehicle (usually a citation number).  Human-readable for printing on mailer.  If not provided we will create a unique value in our system. |
| `referenceId`          | Yes                | string`(uppercase)`      | 50      | `6B547-F4684`                     | Internal reference identifier for future updates. Required for [Status](../status) updates. |
| `violationCode`        | No                | string`(uppercase)`      | 20      | `NP`                              | The violation code in the source system. |
| `violation`            | Yes               | string      | 50      | `No Advance Payment`              | Human-readable violation description. |
| `lotEntryTime`         | No                | string      |         | `2021-11-11T14:39:00-4:00`        | [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601) timestamp when vehicle entered the lot. |
| `lotExitTime`          | No                | string      |         | `2021-11-11T15:05:00-4:00`        | [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601) timestamp when vehicle exited the lot. |
| `imageUrls`            | No                | string[]    | 255     | *(below)*                         | Array of internet-accessible URLs for images of the unpaid vehicle. |
| `actionUrls.paymentUrl`| No                | string      | 255     | `https://my.url.net/`             | Payment URL displayed on the default landing page. |
| `actionUrls.disputeUrl`| No                | string      | 255     | `https://my.url.net`              | Dispute URL displayed on the default landing page. |
| `group.id`             | No                | string      | 50      | `825`                             | Group/market identifier for large operator integration (must match Lot). |
| `group.name`           | No                | string      | 50      | `San Diego`                       | Group/market name for large operator integration (must match Lot). |
| `schedule[].asOf`      | Conditional       | string      |         | `2021-11-11T15:06:00-4:00`        | [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601) timestamp when scheduled amount becomes effective. |
| `schedule[].totalDue`  | Conditional       | decimal     |         | `20.00`                           | Total amount due as of the `asOf` timestamp (replacement value, not additive). |
| `noticeOnVehicle`      | No                | bool        |         |                                   | Indicates if a paper notice was left on the vehicle. |
| `lot`                  | Conditional       | object      |         |                                   | Lot object (required if `lotCode` not provided). |

### Lot Object Fields (when using `lot`)

| Field                | Required       | Type/Format | Max Len | Example(s)              | Description |
|----------------------|----------------|-------------|---------|-------------------------|-------------|
| `lot.code`           | Yes (if lot)   | string`(uppercase)`      | 50      | `A007`                  | Unique lot code in source system. |
| `lot.displayName`    | Yes (if lot)   | string      | 50      | `1st and Pine`          | Display name for customers. |
| `lot.address`        | No             | string      | 100     | `123 N Street`          | Street address of the lot. |
| `lot.city`           | No             | string      | 50      | `Seattle`               | City of the lot address. |
| `lot.state`          | Yes (if lot)   | string`(uppercase)`      | 2       | `WA`                    | State code of the lot address. |
| `lot.zip`            | No             | string      | 10      | `98101`                 | Postal code of the lot address. |
| `lot.ianaTimezone`   | Yes (if lot)   | string      | 50      | `America/New_York`      | [IANA time zone](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones) for the lot. |
| `lot.latitude`       | No             | decimal     | -180,180| `47.60621`             | Latitude of the lot. |
| `lot.longitude`      | No             | decimal     | -180,180| `-112.33207`           | Longitude of the lot. |

### `noticeOnVehicle`
- `true` = Paper notice left on vehicle
- `false` = LPR/AE issued notice with no paper notice left on vehicle

### Example
```yaml
[{
    "lotCode": "A007",
    "referenceId": "444354357435324",
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

### About Images
Our system expects the image URLs to be internet accessible without authentication. If the URLs you submit are short-lived or use temporary access tokens, you can add `?storeImages=true` on the endpoint URL.  This will cause our service to immediacy download the images and store them in our cloud storage.  We also accept [Data URLs](https://developer.mozilla.org/en-US/docs/web/http/basics_of_http/data_urls), for images under 500KB, containing the entire image file.

### About Schedules
Schedules are an optional feature that can be used if you offer discounts for early payment, that expire after a period of time.  If you use the schedule feature, then both the `asOf` and `totalDue` are required for each entry.  There is no practical limit for the number of schedule entires supported for a Citation.  Scheduels are applied within an hour of the `asOf` time by our system.  Schedule entries may increase or reduce the amount due.  Any call to *Status*, with a new `amountDue`, will override the amount set by the latest applied schedule, but will not prevent future schedules from being applied.  

You may supply a schedule entry for the issued date/time of the Citation with the initial amount due, or omit this an send only future changes.  Regardless, the `amountDue` on the Citation must be the currect amout due (with any early pay discounts) as of the time the Citation is sent to us.
