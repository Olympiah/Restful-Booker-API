# Test Summary — Restful-Booker API

## Overview

This summary closes out testing of the Restful-Booker API as planned in the [Test Plan](./03-test-plan-booker.md). It covers what I ran, what passed, the defects I found, and what I deliberately left untested.

## What was tested

- **API:** Restful-Booker v1.0.0, hosted instance at `https://restful-booker.herokuapp.com`
- **Approach:** risk-based, as set out in the [Risk Map](./01-risk-map-booker.md) and [Test Strategy](./02-test-strategy-booker.md)
- **Tooling:** Postman for the collection, Newman to run it from the command line, and GitHub Actions for CI

## Results at a glance

| Measure | Count |
|---|---|
| Test cases designed | 54 |
| Test cases automated and run | 25 |
| Passed | 17 |
| Failed (confirmed defects) | 8 |
| Designed but not automated in this iteration | 29 |
| Defects logged | 4 |

## Results by scenario

| Scenario | Automated | Passed | Failed | Notes |
|---|---|---|---|---|
| S00 Health Check | 1 | 1 | 0 | |
| S01 Authentication | 2 | 1 | 1 | Wrong password returns `200` (BUG-004) |
| S02 Booking Lifecycle | 8 | 8 | 0 | Full create → read → update → patch → delete chain |
| S03 Input Validation | 9 | 2 | 7 | Validation is largely missing (BUG-001 to BUG-003) |
| S04 Authorization | 3 | 3 | 0 | Unauthenticated PUT, PATCH and DELETE all refused, data unchanged |
| S05 Search & Filters | 1 | 1 | 0 | |
| S06 Docs & Contract | 1 | 1 | 0 | Booking response matches the documented schema |
| **Total** | **25** | **17** | **8** | |

## Defects found

| ID | Title | Severity | Test cases |
|---|---|---|---|
| [BUG-001](../bugs/BUG-001-500-on-missing-required-fields.md) | API returns `500` instead of `400` when required booking fields are missing | Medium | TC-S03-01, TC-S03-02 |
| [BUG-002](../bugs/BUG-002-invalid-values-silently-corrupted.md) | API accepts invalid field values and silently stores corrupted data | High | TC-S03-03, TC-S03-04, TC-S03-06 |
| [BUG-003](../bugs/BUG-003-illogical-bookings-accepted.md) | API accepts logically impossible bookings (checkout before checkin, negative price) | Medium | TC-S03-05, TC-S03-07 |
| [BUG-004](../bugs/BUG-004-wrong-password-returns-200.md) | API returns `200` instead of `401` for an invalid password | Medium | TC-S01-02 |

The most serious finding is BUG-002. A price sent as text is stored as `null`, a deposit sent as `"yes"` is stored as `true`, and an impossible date is stored as `"0NaN-aN-aN"`, all with a `200 OK` response. The client is never told its data was changed.

All eight failing tests sit in the non-blocking `99 Known Issues` suite. They still assert the correct behavior, so they'll start passing if the API is ever fixed.

## Risk coverage

| Risk | Covered by | Outcome |
|---|---|---|
| R1 Input validation | S03 | Defects found (BUG-001 to BUG-003) |
| R2 Authentication | S01 | Defect found (BUG-004) |
| R3 Authorization | S04 | No defects: write endpoints refuse unauthenticated requests |
| R4 CRUD data integrity | S02 | No defects in the happy path; BUG-002 shows integrity fails on invalid input |
| R5 Search and filtering | S05 | No defects in the case tested |
| R6 Status codes and errors | S00, S02, S03 | Non-standard codes observed (see below) |
| R7 Docs vs behavior | S06 | No defects in the case tested |
| R8 Method semantics | S02, S03 | No defects in the cases tested |
| R9 Content negotiation | S03 | Malformed JSON correctly rejected; other cases not automated |

## Observations not logged as defects

- `GET /ping` returns `201 Created` and `DELETE /booking/:id` returns `201 Created`. Both are unusual (`200` and `200`/`204` would be standard), but both match the API's own documentation. The smoke suite accepts any `2xx` for delete; the exact-code check (TC-S06-05) was not automated in this iteration.

## Not tested in this iteration

29 designed test cases were not automated. They remain in the [test case sheet](./Booker_API_Test_Cases.xlsx) with Automated = N and Status = Not Run:

- **S01 Authentication:** TC-S01-03, TC-S01-04
- **S03 Input Validation:** TC-S03-08, TC-S03-10 to TC-S03-14, TC-S03-16 to TC-S03-18
- **S04 Authorization:** TC-S04-02 to TC-S04-04, TC-S04-06, TC-S04-08 to TC-S04-10
- **S05 Search & Filters:** TC-S05-02 to TC-S05-07
- **S06 Docs & Contract:** TC-S06-02 to TC-S06-06

I prioritised at least one test per product risk, weighted towards the highest-scoring risks, over full coverage of every designed case. This is a deviation from the test plan's exit criterion that every test case is implemented, and it was a deliberate scope decision.

## CI

The collection runs in GitHub Actions through Newman:

- **Smoke** (health, auth and booking lifecycle) runs on every push to `main` and blocks on failure.
- **Regression** (every folder except Known Issues) runs on manual dispatch and blocks on failure.
- **Known Issues** runs alongside regression, non-blocking, so the eight known defects stay visible without turning the pipeline red.

Each run uploads an HTML report as a downloadable artifact. Credentials are injected at runtime from GitHub Actions secrets; none are committed.

## Observations

- **Don't trust a status code on its own.** Early on, a request sent with the wrong method still returned `200 OK`. Only the assertions on the response body caught it.
- **Debug with evidence, not guesses.** When a chained test kept failing, logging to the Postman console showed the ID was being saved but not kept between requests. That isolated it to a variable-scope issue in the tool rather than the test logic, and switching the value to environment scope fixed it.
- **Shared test environments need self-contained tests.** The public instance resets its data, so every flow creates and cleans up its own booking instead of relying on existing data.
