# BUG-004

## Title
[AUTH - CREATE TOKEN] API returns 200 OK instead of 401 Unauthorized when an invalid password is sent

## Severity/Priority
**Medium.** No token is issued, so a wrong password does not grant access. But the API reports the failed login as a success, so any client that relies on the status code (as most do) will treat the login as successful and only fail later, when it tries to use a token it never received.

## Description
When `POST /auth` is called with a valid username and a wrong password, the API returns `200 OK` instead of `401 Unauthorized`. The response does not contain a token, so the credentials are correctly refused, but the status code says the request succeeded.

A `200` on a failed login breaks the standard HTTP contract for authentication. Clients have to inspect the response body to tell success from failure, instead of relying on the status code, and monitoring tools will count failed logins as successful requests.

**Related test cases:** TC-S01-02
**Related risks:** R2 (authentication), R6 (status codes and error responses)

## Steps to Reproduce
1. Send a `POST` request to `https://restful-booker.herokuapp.com/auth` with the headers `Content-Type: application/json` and `Accept: application/json`, using the valid username and a wrong password:

```bash
curl -i -X POST https://restful-booker.herokuapp.com/auth \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -d '{"username": "admin", "password": "wrong-password"}'
```

2. Observe the status code of the response.
3. Check whether the response body contains a `token`.

## Actual Result
The API returns `200 OK`. No token is included in the response.

## Expected Result
The API returns `401 Unauthorized` with an error message indicating the credentials are invalid. No token is included in the response.

## Attachments
- Postman Test Results for TC-S01-02, showing the failed assertion `expected response to have status code 401 but got 200` and the passed assertion `No token in the response`

## Environment
1. **API:** Restful-Booker v1.0.0, hosted instance
2. **Base URL:** https://restful-booker.herokuapp.com
3. **Tool:** Postman desktop app
