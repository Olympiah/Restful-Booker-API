# BUG-003

## Title
[BOOKING - CREATE] API accepts logically impossible bookings (checkout before checkin, negative price) instead of returning 400 Bad Request

## Severity/Priority
**Medium.** Each field is the right type, but the booking as a whole makes no sense: a stay that ends before it starts, or a price below zero. The API saves these without complaint. Clients can work around it by validating on their side, but the API offers no protection against bad business data.

## Description
When `POST /booking` receives values that are valid on their own but impossible together, or impossible for a booking, the API creates the booking and returns `200 OK` with a `bookingid`:

- A `checkout` date five days **before** the `checkin` date is accepted.
- A negative `totalprice` (`-100`) is accepted.

This means the API has no business-rule validation on bookings. Any client (or mistake) can create stays with negative length and bookings that would pay the guest instead of charging them.

**Related test cases:** TC-S03-05, TC-S03-07
**Related risks:** R1 (input validation on create/update)

## Steps to Reproduce
1. Send a `POST` request to `https://restful-booker.herokuapp.com/booking` with the headers `Content-Type: application/json` and `Accept: application/json`, using a valid booking body where checkout is before checkin:

```bash
curl -i -X POST https://restful-booker.herokuapp.com/booking \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -d '{"firstname": "Jim", "lastname": "Brown", "totalprice": 150, "depositpaid": true, "bookingdates": {"checkin": "2026-10-10", "checkout": "2026-10-05"}, "additionalneeds": "Breakfast"}'
```

2. Observe the status code and whether a `bookingid` is returned.
3. Repeat with a negative `totalprice`:

```bash
curl -i -X POST https://restful-booker.herokuapp.com/booking \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -d '{"firstname": "Jim", "lastname": "Brown", "totalprice": -100, "depositpaid": true, "bookingdates": {"checkin": "2026-11-01", "checkout": "2026-11-05"}, "additionalneeds": "Breakfast"}'
```

4. Observe the status code and whether a `bookingid` is returned.

## Actual Result
- **Checkout before checkin (TC-S03-05):** the API returns `200 OK` and creates the booking (a `bookingid` is returned).
- **Negative totalprice (TC-S03-07):** the API returns `200 OK` and creates the booking (a `bookingid` is returned).

## Expected Result
The API rejects each request with `400 Bad Request` and an error message explaining the rule that was broken (checkout must be after checkin; totalprice cannot be negative). No booking is created.

## Attachments
- Postman Test Results for TC-S03-05 and TC-S03-07, showing the failed assertions: `expected response to have status code 400 but got 200` and `expected '{"bookingid":...' to not include 'bookingid'`

## Environment
1. **API:** Restful-Booker v1.0.0, hosted instance
2. **Base URL:** https://restful-booker.herokuapp.com
3. **Tool:** Postman desktop app
