# Test Strategy — Restful-Booker API

**Author:** Oly
**Status:** Draft v1
**Updated:** September  2026

## Purpose

This document explains *how* I test the Restful-Booker API. The [Risk Map](./01-risk-map.md) decides *where* the effort goes. The [Test Plan](./03-test-plan.md) covers scope, schedule and exit criteria for this project and summarizes this strategy in its Test Approach section.

## Approach in one paragraph

Testing is risk-based. I build a Postman collection where each risk maps to a folder of requests with scripted assertions. Newman runs the collection from the command line. GitHub Actions runs a fast smoke suite on every push and the full regression suite nightly. Expected results come from correct HTTP semantics and the published docs, never from whatever the API currently returns (see P3 in the risk map).

## Test types

| Type | What it means here | Risks |
|---|---|---|
| Functional (positive) | Each endpoint does what it should with valid input. | R2, R4, R5 |
| Negative / validation | Invalid, missing, malformed or illogical input gets a clear `4xx`, not a `500` or silent acceptance. | R1, R8, R9 |
| Authorization | Write endpoints are checked with no token, an invalid token, a valid cookie token and valid Basic auth. | R2, R3 |
| Schema checks | Responses are validated against a JSON schema with `pm.response.to.have.jsonSchema()`, so a wrong type or missing field fails the test even when the status code is fine. | R4, R7 |
| Regression | The full collection re-runs on a schedule to catch changes over time. | All |

## Test design techniques, mapped to risks

- **Equivalence partitioning and boundary value analysis (R1, R5).** Field types, empty vs missing values, zero and negative `totalprice`, and date boundaries such as checkout equal to checkin, checkout before checkin, and filter dates exactly on a booking's dates.
- **Decision table (R2, R3).** Auth method (none / cookie / Basic) × token state (missing / invalid / valid) × endpoint (`PUT` / `PATCH` / `DELETE`). Each combination has one expected outcome, so gaps are easy to see.
- **State transition (R4, R8).** The booking lifecycle: create → read → full update → partial update → delete → read again (expect `404`) → delete again. This covers data integrity and method semantics in one chained flow.
- **Error guessing (R6, R9).** Malformed JSON, wrong `Content-Type`, unsupported `Accept` values, non-numeric IDs and oversized strings. I'm also checking error bodies for leaked internal details.
- **Docs comparison (R7).** Every documented request and response example is exercised once and compared field by field.

## Tooling

- **Postman** for building the collection, environments, pre-request scripts and test scripts.
- **Newman** to run the collection headlessly, locally and in CI.
- **newman-reporter-htmlextra** for a readable HTML report, uploaded as a CI artifact.
- **GitHub Actions** for CI, following the same smoke/regression pattern as my OrangeHRM and Excalidraw projects.

The collection and environment are exported to the repo as JSON, so the tests are versioned with the docs rather than living only in my Postman workspace.

## Collection structure

| Folder | Contents | Suite |
|---|---|---|
| `00 Health` | `GET /ping` | Smoke |
| `01 Auth` | Token creation, bad credentials | Smoke |
| `02 Booking Lifecycle` | The chained create → read → update → patch → delete flow | Smoke |
| `03 Validation` | Negative and boundary cases on create and update | Regression |
| `04 Authorization` | The decision-table cases on write endpoints | Regression |
| `05 Search & Filters` | `GET /booking` filter cases | Regression |
| `06 Docs & Contract` | Doc examples and schema checks | Regression |
| `99 Known Issues` | Tests that fail because of a confirmed, logged bug | Non-blocking |

Smoke answers "is the API up, can I authenticate, and does the core booking flow work?" in under a minute. Regression runs everything.

## Test data

- **No hardcoded booking IDs.** Each flow creates its own booking and passes `bookingid` to later requests through collection variables. This handles the shared, periodically reset instance (P1).
- **Unique data per run**, using Postman dynamic variables such as `{{$randomFirstName}}` plus a run timestamp, so my bookings are identifiable and filter tests aren't polluted by other users' data.
- **Cleanup.** Every flow that creates a booking deletes it at the end.
- **Credentials** live in the environment file locally and in GitHub Actions secrets in CI. They're public demo credentials, but I'm keeping the same habit I'd use on a real project.

## Handling known bugs in CI

Restful-Booker has deliberate bugs, and my tests assert correct behavior. That creates a problem: a correct test for a known bug fails on every run, and a permanently red pipeline hides new failures.

My approach:

1. When a test fails and I confirm it's a real defect, I write a bug report in `bugs/` using my bug report template, linked to the risk ID and test case ID.
2. The test moves to the `99 Known Issues` folder. It keeps asserting the *correct* behavior. I never rewrite it to expect the buggy result.
3. CI runs `99 Known Issues` in a separate, non-blocking job. It stays visible in the report without failing the build.
4. If a known-issue test starts passing, that's a signal the behavior changed, and I review it.

This keeps the gating suites meaningful while documenting every defect honestly.

## CI pipeline

| Trigger | Runs | Blocking? |
|---|---|---|
| Push and pull request | Smoke folders | Yes |
| Nightly schedule and manual dispatch | All regression folders | Yes |
| Same as regression | `99 Known Issues` | No |

Each run uploads its HTML report as an artifact. Since the hosted instance is slow, I'm not gating on response times (P2). Timing is visible in the report but not asserted.

## Defect reporting

Defects go in `bugs/` as Markdown files named `BUG-<number>-<short-title>.md`, using my bug report template. Each report includes the request (as a cURL command), the actual response, the expected behavior with the reason for it, severity, and links to the risk ID and test case.

## Out of scope

As defined in the [Risk Map](./01-risk-map.md#out-of-scope): load, performance, concurrency and rate-limit testing; security testing beyond auth and authorization; and browser, device or OS compatibility.
