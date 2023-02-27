# Citations Post

The posting of citations is the basis for our services.  See below for the detail of both required and supported fields.

### Fields
| Field | Required | Type/Format | Example(s) | Description|
|-------|----------|-------------|---------|------------|
| `lotCode` | Yes | string | `A007` | A name or code you use to indicate the location where the unpaid vehicle is parked. These codes will be exchanged ahead of time along with other location data using the [Lots](../lots) api. |
| `issued` | Yes | string | `2021-11-11T15:06:00-4:00` | An [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601) local time stamp, including UTC offset, when the citation was issued.|
| `plate` | Yes | string | `ABC-1234` | The license plate for the unpaid vehicle. |
| `state` | Yes | string | `WA` | The state or province that issued the license plate. (see `plate`) |
| `make` | No* | string | `Ford` | The make/manufacturer of the unpaid vehicle.  This field can accept [NCIC VMA Codes](https://wilenet.widoj.gov/sites/default/files/public_files-2021-01/ncic_code_manual_-_dec_31_2020.pdf) or proper names. Surcharges may apply if this field is not provided.  |
| `body` | No | string | `Truck` | A short term for the body-style of the vehicle, such as: 'Truck', 'SUV', '2-door', '4-door', etc.  The value should be meaningful to a human reader. |
| `color` | No | string | `Red` | The color of the vehicle. |
| `vin` | No | string | `*5678` | Can contain the full vehicle VIN or the last 4 digits of the VIN if prefixed with `*`. |
| `unpaidParking` | No | decimal | `10.00` | The unpaid amount of the parking charges. |
| `amountDue` | Yes | decimal | `20.00` | The total amount due on this citation. |
| `referenceNum` | No | string | `C123456` | A reference number for the unpaid vehicle, usually a citation number issued from an internal system |
| `referenceId` | No* | string | `6B547-F4684` | An internal reference identifier, unique to your source system, to be used future updates.  Calls to update [Status](..\status) require a matching value. |
| `violationCode` | No | string | `NP` | The violation code in the sources system. |
| `violation` | Yes | string | `No Advance Payment` | The human-readable violation description. |
| `lotEntryTime` | No | string | `2021-11-11T14:39:00-4:00` | An [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601) local time stamp, including UTC offset, when the vehicle entered the lot. |
| `lotExitTime` | No | string | `2021-11-11T15:05:00-4:00` | An [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601) local time stamp, including UTC offset, when the vehicle existed the lot. |
| `imageUrls` | No | string [] | *(below)* | An array of internet accessible URLs for images of the unpaid vehicle in the location. |
| `actionUrls.paymentUrl` | No | string | `https://my.url.net/` | An internet accessible URL for the customer to make a payment. This will be displayed on the default landing page. |
| `actionUrls.diusputeUrl` | No | string | `https://my.url.net` | An internet accessible URL for the customer to submit a dispute. This will be displayed on the default landing page. |
| `group.id` | No | string | `825` | A short identifier for the group/market for large operator integration. If supplied, this needs to match the value on the Lot. |
| `group.name` | No | string | `San Diego` | The name for the group/market for large operator integration. If supplied, this needs to match the value on the Lot. |

### Example

```yaml
[{
    "lotCode": "A007",
    "issued": "2021-11-11T15:06:00-4:00",
    "plate": "ABC1234",
    "state": "WA",
    "make": "Ford",
    "amountdue": 10.00,
    "violation": "Overtime"
},{
    "lotCode": "A007",
    "issued": "2021-11-01T02:00:00-7:00",
    "plate": "BCE1234",
    "state": "AZ",
    "make": "Toyota",
    "body": "Truck",
    "color": "Silver",
    "amountDue": 42.00,
    "referenceId": "444354357435324",
    "violation": "No Advance Payment",
    "imageUrls": [ "https://s3.amazon.com/my-account/image12728.jpg" ],
    "actionUrls": {
        "paymentUrl": "https://unpaidparking.net/pay?plate=BCE1234"
    }
},{
    "lotCode": "Q301",
    "issued": "2021-11-10T22:00:00-6:00",
    "plate": "CDF1234",
    "state": "ID",
    "make": "GMC",
    "body": "SUV",
    "color": "Black",
    "amountDue": 32.50,
    "violation": "Expired Permit",
    "referenceNum": "C123456",
    "referenceId": "32573543574354364",
    "imageUrls": [ 
        "https://my.site.net/plate-images/img76985.jpg", 
        "https://blob.azure.com/my-company/unpaid/img84879.jpg"
    ]
}]

```


 
