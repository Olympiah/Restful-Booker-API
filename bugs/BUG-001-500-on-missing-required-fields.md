# BUG-001

## Title
[BOOKING - CREATE] API returns 500 Internal Server Error instead of 400 Bad Request when required booking fields are missing

## Severity/Priority
**Medium.** Creating a booking with incomplete data fails with a server error instead of a clear validation error. No invalid booking is stored, and clients can work around it by always sending a complete body, but they get no indication of what's wrong with their request.

## Description
When `POST /booking` receives a request body that is missing required fields (`totalprice`, `depositpaid`, `bookingdates`), or an empty JSON body, the API crashes with `500 Internal Server Error` instead of rejecting the request with `400 Bad Request`.

A `500` tells the client the server failed, not that the request was invalid. So consumers can't tell a bad request apart from a genuine outage, and they're given no information about which field needs fixing. This suggests the API doesn't validate the request body before processing it.

**Related test cases:** TC-S03-01, TC-S03-02
**Related risks:** R1 (input validation on create/update), R6 (status codes and error responses)

## Steps to Reproduce
1. Send a `POST` request to `https://restful-booker.herokuapp.com/booking` with the headers `Content-Type: application/json` and `Accept: application/json`.
2. Use a body that is missing `totalprice`, `depositpaid` and `bookingdates`:

```bash
curl -i -X POST https://restful-booker.herokuapp.com/booking \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -d '{"firstname": "Jim", "lastname": "Brown", "additionalneeds": "Breakfast"}'
```

3. Observe the status code of the response.
4. Repeat with an empty JSON body:

```bash
curl -i -X POST https://restful-booker.herokuapp.com/booking \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -d '{}'
```

5. Observe the status code of the response.

## Actual Result
- **Missing required fields (TC-S03-01):** the API returns `500 Internal Server Error`. No booking is created.
- **Empty JSON body (TC-S03-02):** the API returns `500 Internal Server Error`. No booking is created.

In both cases the response gives no indication of which fields are missing or invalid.

## Expected Result
The API rejects the request with `400 Bad Request` and an error message identifying the missing required fields. No booking is created.

## Attachments
- Postman Test Results for TC-S03-01 and TC-S03-02, showing the failed assertion: `expected response to have status code 400 but got 500`

## Environment
1. **API:** Restful-Booker v1.0.0, hosted instance
2. **Base URL:** https://restful-booker.herokuapp.com
3. **Tool:** Postman desktop app
