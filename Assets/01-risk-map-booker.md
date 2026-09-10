# Risk Map — Restful-Booker API

**Author:** Oly
**Status:** Draft v1
**Updated:** September  2026

## Purpose

This risk map decides where my testing effort goes. Every test case and every decision in the [Test Strategy](./02-test-strategy.md) traces back to a risk ID below, so it's clear why something is tested, and why some things aren't.

## Context

[Restful-Booker](https://restful-booker.herokuapp.com/apidoc/index.html) is a purpose-built practice API for a fictional hotel booking system. It isn't a production product, and it ships with deliberate bugs. I chose it to build an API test suite in Postman and automate it with Newman in CI. I'm scoring risk as if a real booking system depended on this API, because that's the mindset the exercise is meant to train.

Endpoints under test: `POST /auth`, `GET /booking` (with filters), `GET /booking/:id`, `POST /booking`, `PUT /booking/:id`, `PATCH /booking/:id`, `DELETE /booking/:id`, `GET /ping`.

## Scoring

- **Impact (1–5):** how bad it would be for a booking system and its consumers if this failed.
- **Likelihood (1–5):** how likely a defect is, based on the API's complexity, what the docs leave unspecified, and what I've already observed.
- **Score = Impact × Likelihood.** High is 15 and above. Medium is 9–14. Low is 8 and below.

## Product risks

| ID | Risk area | What could go wrong | I | L | Score | Priority |
|---|---|---|---|---|---|---|
| R1 | Input validation on create/update | Missing, wrong-type or illogical fields (checkout before checkin, negative price) are accepted, or crash the server with a `500` instead of returning a `400`. I already saw a `500` for incomplete bodies in my 2024 run. | 4 | 5 | **20** | High |
| R2 | Authentication | `POST /auth` handles bad credentials incorrectly (for example, a `200` with an error message). Invalid, tampered or reused tokens are accepted. Cookie-token and Basic-auth behave differently. | 5 | 3 | **15** | High |
| R3 | Authorization on write endpoints | `PUT`, `PATCH` or `DELETE` succeed with a missing or invalid token, or fail with the wrong status code (`403` vs `401`). | 5 | 3 | **15** | High |
| R4 | CRUD data integrity | A created booking isn't stored as sent. `PUT`/`PATCH` change fields they shouldn't. A deleted booking can still be retrieved. | 5 | 3 | **15** | High |
| R5 | Search and filtering | `GET /booking` filters (firstname, lastname, checkin, checkout) return wrong, partial or unfiltered results, especially around date boundaries. | 3 | 4 | **12** | Medium |
| R6 | Status codes and error responses | Non-standard codes (for example, `201` on delete or ping), inconsistent error bodies, or internal details leaked in error responses. | 3 | 4 | **12** | Medium |
| R7 | Docs vs actual behavior | The API behaves differently from its published documentation, so consumers build against a contract that isn't real. | 3 | 4 | **12** | Medium |
| R8 | Method semantics | `PUT` with a partial body behaves like `PATCH`. `PATCH` with an empty body. Deleting the same booking twice. Operations on non-existent IDs. | 3 | 3 | **9** | Medium |
| R9 | Content negotiation | `Accept` / `Content-Type` combinations (JSON, XML, form-encoded) are handled inconsistently or return unhelpful errors. | 3 | 3 | **9** | Medium |

## Project risks

These affect how reliably I can test rather than the product itself. They shape the test design in the strategy.

| ID | Risk | Mitigation |
|---|---|---|
| P1 | The public instance is shared. Other people create, change and delete bookings, and the data is periodically reset. | No hardcoded booking IDs. Every flow creates its own booking, then reads, updates and deletes it, passing the ID through collection variables. |
| P2 | The hosted instance is slow and response times vary, so timing assertions would be flaky. | No strict response-time assertions in CI. Timing is observed and noted, not gated. |
| P3 | Deliberate bugs mean "expected" behavior can't be read from what the API currently does. | Expected results are based on correct HTTP semantics and the docs. Deviations are logged as bugs, not written into tests as passing behavior. |

## Out of scope

- **Load, performance, concurrency and rate-limit testing.** The instance is a shared public service I don't own. Stress-testing it would be inappropriate, and the results would be meaningless.
- **Security testing beyond auth and authorization.** No injection or penetration testing against someone else's server.
- **Browser, device and OS compatibility.** Not applicable to an API.

## Next

The [Test Strategy](./02-test-strategy.md) turns these risks into an approach: which techniques apply to each risk, how the Postman collection is organized, and which risks the CI smoke suite covers versus the full regression suite.
