# Mail Data Callback

This callback is available exclusively to **print/mail service providers** and **government clients** who handle their own mailing. It is **not available** to private operators or lot owners.

## DPPA Restrictions

The Mail Data payload contains personal information obtained from Motor Vehicle Records (MVR) through DMV lookups, including the registered owner's name and mailing address. The federal Driver's Privacy Protection Act (DPPA), codified at [18 U.S.C. 2721-2725](https://www.law.cornell.edu/uscode/text/18/part-I/chapter-123), restricts the disclosure of personal information from motor vehicle records.

Only entities with a permissible use under the Act may receive this data. Permissible uses relevant to this callback include:

- **Government agencies** exercising official functions (e.g., municipal parking authorities, university police departments, government-operated transit authorities)
- **Contracted print/mail providers** acting as authorized agents of the data requester, under a written agreement that limits use of the data to the contracted purpose

Private parking operators and lot owners generally do not have a permissible use that would allow them to receive raw MVR data directly. Unauthorized disclosure of MVR data can result in civil and criminal penalties under federal law.

## Configuration

This callback is **not configurable through the public callback configuration API**. Configuration is managed internally. Contact your account representative to set up or modify the Mail Data callback URL.

The same `authorization` header configured for your other callbacks will be used for Mail Data callbacks. All standard [operational contract](README.md#operational-contract) behavior (batching, retry, frequency) applies.

## Reporting results back

After you generate and mail the letters described by this callback, you report the outcome back through the restricted [Letters](../letters) endpoint — either attaching the rendered PDF or reporting a status (e.g. undeliverable), keyed by the `letter_id` from this payload.  Like this callback, that endpoint is enabled per-account and is not part of the general integration flow.

## Payload

We will post a JSON Array of mail data records when mailing letters are prepared with complete violation and recipient data.

### Fields
| Field | Type | Description |
|-------|------|-------------|
| `letter_id` | integer | Mailing letter record ID. |
| `reference_id` | string | The internal reference identifier, unique to your source, that was supplied with the original citation post. |
| `letter_code` | string | Letter template code identifying the type of letter being sent. |
| `violation_number` | string | The violation notice number. |
| `violation_description` | string | Description of the violation type. |
| `issue_date` | string (date) | Date the violation was issued (`yyyy-MM-dd`). |
| `issue_time` | string (time) | Time the violation was issued (12-hour format, e.g. `2:30 PM`). |
| `license_plate` | string | Vehicle license plate number. |
| `license_state` | string | License plate state abbreviation. |
| `vehicle_make` | string | Vehicle manufacturer. |
| `vehicle_type` | string | Vehicle body type (e.g. `Sedan`, `SUV`, `Truck`). |
| `vehicle_color` | string | Vehicle color. |
| `parking_amount` | decimal | Parking fee amount. |
| `violation_amount` | decimal | Total violation amount. |
| `total_amount` | decimal | Total amount due. |
| `current_due` | decimal | Current balance due. |
| `location_name` | string | Parking lot name or lot code. |
| `location_address` | string | Lot street address. |
| `location_city` | string | Lot city. |
| `client_name` | string | Client display name for mailing purposes. |
| `client_city` | string | Client city. |
| `client_url` | string | Client website URL. |
| `recipient_name` | string | Letter recipient full name. *DPPA-protected MVR data.* |
| `recipient_address_1` | string | Recipient address line 1. *DPPA-protected MVR data.* |
| `recipient_address_2` | string | Recipient address line 2. *DPPA-protected MVR data.* |
| `recipient_city` | string | Recipient city. *DPPA-protected MVR data.* |
| `recipient_state` | string | Recipient state. *DPPA-protected MVR data.* |
| `recipient_zip` | string | Recipient ZIP code. *DPPA-protected MVR data.* |
| `impound_boot` | string | Impound/boot policy indicator. |
| `city_code` | string | Client city code. |
| `qr_code_url` | string | URL for the payment QR code. |
| `image_url_1` | string | First violation photo URL. |
| `image_url_2` | string | Second violation photo URL. |
| `dmv_addendum` | string | State-specific DMV addendum text required on certain notices. |
| `lot_entry_time` | string (timestamp) | Lot entry time ([ISO-8601](https://en.wikipedia.org/wiki/ISO_8601) with timezone, e.g. `2024-03-15T08:30:00-05:00`). |
| `lot_exit_time` | string (timestamp) | Lot exit time ([ISO-8601](https://en.wikipedia.org/wiki/ISO_8601) with timezone, e.g. `2024-03-15T14:45:00-05:00`). |

### Example

```json
[{
  "letter_id": 458201,
  "reference_id": "6B547-F4684",
  "letter_code": "NOTICE1",
  "violation_number": "V-2024-00158",
  "violation_description": "Overtime Parking",
  "issue_date": "2024-03-15",
  "issue_time": "2:30 PM",
  "license_plate": "ABC1234",
  "license_state": "FL",
  "vehicle_make": "Toyota",
  "vehicle_type": "Sedan",
  "vehicle_color": "Silver",
  "parking_amount": 0.00,
  "violation_amount": 45.00,
  "total_amount": 45.00,
  "current_due": 45.00,
  "location_name": "City Hall Garage",
  "location_address": "100 Main Street",
  "location_city": "Fort Myers",
  "client_name": "City of Fort Myers",
  "client_city": "Fort Myers",
  "client_url": "https://www.cityftmyers.com",
  "recipient_name": "Jane Smith",
  "recipient_address_1": "742 Evergreen Terrace",
  "recipient_address_2": null,
  "recipient_city": "Fort Myers",
  "recipient_state": "FL",
  "recipient_zip": "33901",
  "impound_boot": null,
  "city_code": "FTM",
  "qr_code_url": "https://pay.parkpliant.com/qr/6B547-F4684",
  "image_url_1": "https://storage.net/photos/img001.jpg",
  "image_url_2": "https://storage.net/photos/img002.jpg",
  "dmv_addendum": "Florida Statute 316.1967 authorizes...",
  "lot_entry_time": "2024-03-15T08:30:00-05:00",
  "lot_exit_time": "2024-03-15T14:45:00-05:00"
}]
```
