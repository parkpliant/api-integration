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
https://push.parkpliant.com/api/swagger/ui

## API Endpoints

**Citations Post Service**

https://push.parkpliant.com/api/citations

**Status Post Service**

https://push.parkpliant.com/api/status

**Lots Post Service**

https://push.parkpliant.com/api/lots

**Callback Post Service**

https://push.parkpliant.com/api/callbacks


*The endpoints above accept only HTTPS POST requests with a content-type "application/json".  Attempting to visit them in your browser will usually result in a 404 or 405 error.*

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
    "make": "Ford",
    "amountDue": 10.00,
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
}]
```

In the above example:
- 2 citations were sent, sending 2 posts with one record each is also fine, though less efficient.  The post must always be an array ( [] in JSON ), even if the array only contains one element.

----

### Example Response
```yaml
HTTP/1.1 200 OK
Content-Length: 55
Content-Type: application/json; charset=utf-8

{
    "id": "c31deb20-1069-40c8-b218-e7f7e63ba56d",
    "count": 2,
    "errors": []
}
```
In the above example:
- The `id` is a unique identifier supplied for the transaction, that can be used for transaction tracing.  We recommend recording this value when possible, although is it not required.	




