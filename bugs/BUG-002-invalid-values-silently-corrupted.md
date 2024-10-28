# BUG-002

## Title
[BOOKING - CREATE] API accepts invalid field values and silently stores corrupted data instead of returning 400 Bad Request

## Severity/Priority
**High.** Bookings are saved with missing or corrupted data (no price, an unreadable check-in date, a deposit status the client never sent), and the API reports success. The client has no way of knowing the stored booking differs from what it sent, short of reading it back and comparing every field.

## Description
When `POST /booking` receives a field with the wrong type or an invalid value, the API does not reject the request. Instead, it converts the value into something else, stores the booking, and returns `200 OK` with a `bookingid`:

- A text `totalprice` (`"abc"`) is stored as `null`.
- A text `depositpaid` (`"yes"`) is stored as `true`.
- An impossible `checkin` date (`"2026-13-45"`) is stored as the corrupted string `"0NaN-aN-aN"`.

This means the API does not validate field types or date formats before saving. Every downstream consumer (billing, availability, reporting) then has to deal with records containing a missing price, a guessed deposit status, or a date that can't be parsed.

**Related test cases:** TC-S03-03, TC-S03-04, TC-S03-06
**Related risks:** R1 (input validation on create/update), R4 (CRUD data integrity)

## Steps to Reproduce
1. Send a `POST` request to `https://restful-booker.herokuapp.com/booking` with the headers `Content-Type: application/json` and `Accept: application/json`, using a valid booking body with `totalprice` set to text:

```bash
curl -i -X POST https://restful-booker.herokuapp.com/booking \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -d '{"firstname": "Jim", "lastname": "Brown", "totalprice": "abc", "depositpaid": true, "bookingdates": {"checkin": "2026-11-01", "checkout": "2026-11-05"}, "additionalneeds": "Breakfast"}'
```

2. Observe the status code and the `totalprice` value in the response.
3. Repeat with `depositpaid` set to text:

```bash
curl -i -X POST https://restful-booker.herokuapp.com/booking \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -d '{"firstname": "Jim", "lastname": "Brown", "totalprice": 150, "depositpaid": "yes", "bookingdates": {"checkin": "2026-11-01", "checkout": "2026-11-05"}, "additionalneeds": "Breakfast"}'
```

4. Observe the status code and the `depositpaid` value in the response.
5. Repeat with an impossible `checkin` date:

```bash
curl -i -X POST https://restful-booker.herokuapp.com/booking \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -d '{"firstname": "Jim", "lastname": "Brown", "totalprice": 150, "depositpaid": true, "bookingdates": {"checkin": "2026-13-45", "checkout": "2026-11-05"}, "additionalneeds": "Breakfast"}'
```

6. Observe the status code and the `checkin` value in the response.

## Actual Result
In all three cases the API returns `200 OK`, creates the booking and returns a `bookingid`, with the invalid value silently changed:

| Test case | Field | Value sent | Value stored |
|---|---|---|---|
| TC-S03-03 | `totalprice` | `"abc"` | `null` |
| TC-S03-04 | `depositpaid` | `"yes"` | `true` |
| TC-S03-06 | `bookingdates.checkin` | `"2026-13-45"` | `"0NaN-aN-aN"` |

Example response from TC-S03-06:

```json
{
    "bookingid": 5229,
    "booking": {
        "firstname": "Garrick",
        "lastname": "Schaefer-1790581813879",
        "totalprice": 150,
        "depositpaid": true,
        "bookingdates": {
            "checkin": "0NaN-aN-aN",
            "checkout": "2026-11-05"
        },
        "additionalneeds": "Breakfast"
    }
}
```

## Expected Result
The API rejects each request with `400 Bad Request` and an error message naming the invalid field. No booking is created, and no value is converted or stored on the client's behalf.

## Attachments
- Postman Test Results for TC-S03-03, TC-S03-04 and TC-S03-06, showing the failed assertions: `expected response to have status code 400 but got 200` and `expected '{"bookingid":...' to not include 'bookingid'`
- Response bodies for each case, as shown in the Actual Result table above

## Environment
1. **API:** Restful-Booker v1.0.0, hosted instance
2. **Base URL:** https://restful-booker.herokuapp.com
3. **Tool:** Postman desktop app
