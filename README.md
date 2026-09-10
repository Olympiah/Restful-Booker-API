# Restful-Booker API Testing — Postman + Newman

![API Tests](https://github.com/Olympiah/Restful-Booker-API/actions/workflows/api-tests.yml/badge.svg)

Risk-based API testing of [Restful-Booker](https://restful-booker.herokuapp.com/apidoc/index.html), a hotel booking API, using a Postman collection run by Newman in GitHub Actions.

**A note on the API:** Restful-Booker is a purpose-built practice API for testers, not a production system, and it ships with deliberate bugs. I chose it to build an API test suite in Postman and learn to automate it with Newman in CI. I tested it as if a real booking system depended on it.

## Highlights

- **25 automated tests** across authentication, the full booking lifecycle, input validation, authorization, search and schema checks
- **4 defects found and documented**, including one where invalid data is silently corrupted and saved with a `200 OK`
- **Chained requests:** every flow creates its own booking and passes IDs and tokens between requests, so tests run unattended on a shared public server
- **Known Issues suite:** tests for confirmed bugs keep asserting the correct behavior, but run in a non-blocking CI job so the pipeline stays meaningful
- **Schema check** on the booking response, validating field names, types and date format
- **No committed credentials:** they're injected at runtime from GitHub Actions secrets

## Defects found

| ID | Summary | Severity |
|---|---|---|
| [BUG-001](bugs/BUG-001-500-on-missing-required-fields.md) | `500` instead of `400` when required booking fields are missing | Medium |
| [BUG-002](bugs/BUG-002-invalid-values-silently-corrupted.md) | Invalid values silently changed and saved (e.g. price `"abc"` stored as `null`) | High |
| [BUG-003](bugs/BUG-003-illogical-bookings-accepted.md) | Checkout before checkin and negative prices accepted | Medium |
| [BUG-004](bugs/BUG-004-wrong-password-returns-200.md) | Wrong password returns `200` instead of `401` | Medium |

## Documentation

| Document | What it covers |
|---|---|
| [Risk Map](Assets/01-risk-map-booker.md) | Product and project risks, scored by impact × likelihood |
| [Test Strategy](Assets/02-test-strategy-booker.md) | How I test: techniques, tooling, test data, CI and handling known bugs |
| [Test Plan](Assets/03-test-plan-booker.md) | Scope, environments, schedule, deliverables and entry/exit criteria |
| [Test Cases](Assets/Booker_API_Test_Cases.xlsx) | 54 designed test cases, traced to risk IDs, with results |
| [Test Summary](Assets/04-test-summary-booker.md) | What ran, what passed, defects found and what's untested |

## Repo structure

```
Assets/       Risk map, test strategy, test plan, test cases and test summary
bugs/         One Markdown bug report per defect
postman/      Exported Postman collection and environment
.github/      GitHub Actions workflow
package.json  Newman run scripts
```

## Collection structure

| Folder | Contents | Suite |
|---|---|---|
| `00 Health` | `GET /ping` | Smoke |
| `01 Auth` | Token creation | Smoke |
| `02 Booking Lifecycle` | Chained create → read → update → patch → delete | Smoke |
| `03 Validation` | Malformed JSON and invalid booking ID cases | Regression |
| `04 Authorization` | Unauthenticated update, patch and delete | Regression |
| `05 Search & Filters` | Filter by firstname | Regression |
| `06 Docs & Contract` | Booking response schema check | Regression |
| `99 Known Issues` | Tests for confirmed defects | Non-blocking |

## Running locally

Requires [Node.js](https://nodejs.org/).

```bash
npm ci
npm run test:smoke -- --env-var username=admin --env-var password=password123
npm run test:regression -- --env-var username=admin --env-var password=password123
npm run test:known-issues -- --env-var username=admin --env-var password=password123
```

These are Restful-Booker's public demo credentials, published in its API docs. Each run writes an HTML report to `newman/`.

## CI

| Trigger | Jobs | Blocking? |
|---|---|---|
| Push to `main` | Smoke | Yes |
| Manual dispatch (Actions → API Tests → Run workflow) | Regression | Yes |
| Manual dispatch | Known Issues | No |

Each run uploads its HTML report as a downloadable artifact.

## Author

Oly · [LinkedIn](https://www.linkedin.com/in/olympiah-otieno) · [Blog](https://myblogsaboutsoftwarequalityassurance.blogspot.com/)
