# ParkPliant - Push API Integration

Our solutions support push of data via a flexible API.

- All data is pushed through HTTPS POST "application/json" to our API.
- All calls are authenticated via Basic authentication ("Authorization" header).
- Each push can contain from 1 to 1000 records of data.
- Our server will generally return one of the following responses
  - 200 – OK
    - This is a success response that means the transaction was accepted and queued for processing
    - It will include "application/json" content with a transaction id from our server and count of records received.
  - 204 - No Content
    - The request was properly formatted except for an empty payload, the request was discarded.
  - 400 – Bad Request
    - The request was improperly formatted, the payload was discarded
  - 401 – Unauthorized
    - The request came without authentication or with invalid credentials

----

### Open API (Swagger) Documentation
https://push.sandbox.parkpliant.com/api/swagger/ui (Sandbox)

https://push.parkpliant.com/api/swagger/ui (Production)


## API Endpoints

*Post endpoints accept only HTTPS POST requests with a content-type "application/json".  Attempting to visit them in your browser will usually result in a 404 or 405 error.*

**Citations Post Service**

https://push.parkpliant.com/api/citations

**Status Post Service**

https://push.parkpliant.com/api/status

**Lots Post Service**

https://push.parkpliant.com/api/lots

**Callback Post Service**

https://push.parkpliant.com/api/callbacks

**Images Post Service**

https://push.parkpliant.com/api/images

**Query Get Service**

https://push.parkpliant.com/api/query

----

### Example Request
```yaml
POST /api/citations HTTP/1.1
Host: push.parkpliant.com
Connection: keep-alive
Content-Length: <body-length>
Authorization: Basic <base64(username:password)>
Content-Type: application/json
Accept: application/json, text/json

[{
    "lotCode": "A007",
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

In the above example:
- 3 citations were sent, sending 3 posts with one record each is also fine, though less efficient.  The post must always be an array ( [] in JSON ), even if the array only contains one element.

----

### Example Response
```yaml
HTTP/1.1 200 OK
Content-Length: 55
Content-Type: application/json; charset=utf-8

{
    "id": "c31deb20-1069-40c8-b218-e7f7e63ba56d",
    "count": 3,
    "errors": []
}
```
In the above example:
- The `id` is a unique identifier supplied for the transaction, that can be used for transaction tracing.  We recommend recording this value when possible, although is it not required.	




