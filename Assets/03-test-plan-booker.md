# Test Plan — Restful-Booker API

**Author:** Oly
**Status:** Draft v1
**Updated:** September  2026
**Supersedes:** my original 2024 test plan for this API

## Objective

Verify that the Restful-Booker API handles authentication, authorization and the full booking lifecycle correctly, rejects invalid input cleanly, and behaves as its documentation says. Deliver the result as a versioned Postman collection that runs automatically in CI through Newman.

## Scope

**Test items**

- **API:** Restful-Booker, version 1.0.0 as published in its [API docs](https://restful-booker.herokuapp.com/apidoc/index.html)
- **Base URL:** `https://restful-booker.herokuapp.com`
- **Endpoints:** `POST /auth`, `GET /booking`, `GET /booking/:id`, `POST /booking`, `PUT /booking/:id`, `PATCH /booking/:id`, `DELETE /booking/:id`, `GET /ping`

### Inclusions

The product risks R1–R9 in the [Risk Map](./01-risk-map-booker.md):

- Input validation on create and update (R1)
- Authentication (R2) and authorization on write endpoints (R3)
- CRUD data integrity across the booking lifecycle (R4)
- Search and filters on `GET /booking` (R5)
- Status codes and error responses (R6)
- Docs vs actual behavior, including JSON schema checks (R7)
- Method semantics (R8) and content negotiation (R9)

### Exclusions

- Load, performance, concurrency and rate-limit testing
- Security testing beyond authentication and authorization
- Browser, device and OS compatibility

The reasons are in the [Risk Map](./01-risk-map-booker.md#out-of-scope).

## Test Environments

| Item | Details |
|---|---|
| System under test | Hosted public instance at the base URL above |
| Local runs | Postman desktop app and Newman on my machine |
| CI runs | Newman in GitHub Actions |

The instance is shared and periodically reset, so no test depends on pre-existing data (see P1 in the risk map).

## Defect Reporting Procedure

- A defect is any behavior that deviates from correct HTTP semantics or the published docs, even when the deviation is deliberate in this practice API.
- Each defect is logged in `bugs/` as `BUG-<number>-<short-title>.md`, using my bug report template.
- Each report includes the request as a cURL command, the actual response, the expected behavior with the reason for it, severity, and links to the risk ID and test case ID.
- The failing test moves to the non-blocking Known Issues suite and keeps asserting the correct behavior.

Full details: [Test Strategy](./02-test-strategy-booker.md#handling-known-bugs-in-ci).

## Test Approach

Testing is risk-based. Each risk maps to a folder in a Postman collection, where requests carry scripted assertions on status codes, body values, headers and JSON schema. Every flow creates and cleans up its own test data, so the suite works on the shared public instance. Newman runs the collection in GitHub Actions: a smoke suite on every push, and the full regression suite nightly. Tests for confirmed bugs sit in a non-blocking Known Issues folder.

Full details: [Test Strategy](./02-test-strategy-booker.md).

## Test Schedule

| Phase | Work |
|---|---|
| 1. Test design | Risk map, strategy, this plan, test cases |
| 2. Collection build | Postman collection, environment, test scripts, schema checks |
| 3. Automation | Newman locally, then the GitHub Actions workflow |
| 4. Closure | Test summary, README, portfolio card |

No fixed dates, since this is a side project. Phases run in order, and each leaves the repo in a presentable state.

## Test Deliverables

| Deliverable | Location |
|---|---|
| Risk map | `Assets/01-risk-map-booker.md` |
| Test strategy | `Assets/02-test-strategy-booker.md` |
| Test plan | `Assets/03-test-plan-booker.md` |
| Test cases | `Assets/Booker_API_Test_Cases.xlsx` |
| Postman collection and environment | `postman/` |
| CI workflow | `.github/workflows/` |
| Bug reports | `bugs/` |
| Test summary | `Assets/04-test-summary-booker.md` |
| README | `README.md` |

## Entry and Exit Criteria

### Test Design

- **Entry:** the risk map and test strategy are complete.
- **Exit:** every product risk has at least one test case, and each test case traces to a risk ID.

### Test Execution

- **Entry:** test cases are complete.
- **Exit:** every test case is implemented in Postman and runs in Newman. Smoke and regression run green in CI, with confirmed bugs logged and isolated in the Known Issues suite.

### Test Closure

- **Entry:** execution is complete and all failures are triaged.
- **Exit:** the test summary reports what ran, what passed, the defects found and what's still untested.

### Suspension and Resumption

If the hosted instance is down or returning server errors across the board, I pause testing, since results would reflect the environment rather than the API's behavior. I resume once `GET /ping` responds normally, and re-run anything affected.

## Tools

- **Postman** for building the collection, environments and test scripts
- **Newman** to run the collection from the command line
- **newman-reporter-htmlextra** for HTML reports
- **GitHub Actions** for CI
- **Markdown bug reports** in `bugs/`, using my bug report template

## Risks and Mitigations

The environment risks (a shared, resetting instance; slow response times; deliberate bugs) are covered as P1–P3 in the [Risk Map](./01-risk-map-booker.md#project-risks). Two more apply to how I'm running this project:

| Risk | Mitigation |
|---|---|
| Newman and Postman scripting are new or rusty for me, which could stall execution | Get Newman running locally against the smoke folders first, then add CI. Don't build the full workflow in one go. |
| This is a side project, so progress is limited to the time I have around work | Phases are small and self-contained, and each leaves the repo in a presentable state. |
