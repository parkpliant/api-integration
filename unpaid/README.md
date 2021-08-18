# Unpaid Post

The posting if unpaid vehicles is the basis for our services.  See below for the detail of supported fields

### Fields
| Field | Required | Type/Format | Example(s) | Description|
|-------|----------|-------------|---------|------------|
| `location` | Yes | string | `A007` | A name or code you use to indicate the location where the unpaid vehicle is parked. These codes will be exchanged ahead of time along with other location data. |
| `plate` | Yes | string | `ABC-1234` | The license plate for the unpaid vehicle. |
| `state` | Yes | string | `WA` | The state or province that issued the license plate. (see `plate`) |
| `make` | Yes | string | `Ford` | The make/manufacturer of the unpaid vehicle.  This field can accept [NCIC   VMA Codes](https://wilenet.widoj.gov/sites/default/files/public_files-2021-01/ncic_code_manual_-_dec_31_2020.pdf) or proper names.  |
| `body` | No | string | `Truck` | A short term for the body-style of the vehicle, such as 'Truck', 'SUV', '2-door', '4-door', etc.  The value should be meaningful to a human reader. |
| `color` | No | string | `Red` | The color of the vehicle. |
| `vin` | No | string | `*5678` | Can contain the full vehicle VIN or the last 4 digits of the VIN if prefixed with `*`. |
| `amountDue` | No | decimal | `10.00` | The unpaid amount due for the vehicle. |

### Example

```yaml
[{
    "location": "A007",
    "plate": "ABC-1234",
    "state": "WA",
    "make": "Ford"
},{
    "location": "A007",
    "plate": "BCE-1234",
    "state": "AZ",
    "make": "Toyota",
    "body": "Truck",
    "color": "Silver",
    "amountDue": 10.00
},{
    "location": "Q301",
    "plate": "CDF-1234",
    "state": "ID",
    "make": "GMC",
    "body": "SUV",
    "color": "Black",
    "amountDue": 12.50
}]
```


 
