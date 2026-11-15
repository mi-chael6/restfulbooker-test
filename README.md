# Restful Booker API Automation

This folder contains a Postman collection for the API-testing portion of the Junior Tech Associate (QA) assessment. It exercises an authenticated booking lifecycle against the live [Restful-Booker API](https://restful-booker.herokuapp.com/apidoc/index.html).

## Collection

- [Restful Booker API Automation.postman_collection.json](./Restful%20Booker%20API%20Automation.postman_collection.json) — Postman Collection v2.1.

The collection is stateful: requests share a token and booking ID through collection variables. Run the requests in order using the Collection Runner.

## Setup and execution

1. Import the collection JSON into Postman.
2. Set the collection variable `baseUrl` to `https://restful-booker.herokuapp.com`.
3. Run the collection from top to bottom. The Auth request obtains a token and stores it in `token`; Create Booking stores the returned numeric ID in `bookingId`.
4. Review the Collection Runner results for each request and its assertions.

The Auth request currently includes the Restful-Booker demo credentials in its JSON body (`admin` / `password123`). Do not reuse these credentials for any other service. The collection creates and deletes live test data; run it only against the intended assessment environment.

## Request flow and checks

| Request | Method and path | Purpose and assertions |
|---|---|---|
| Auth | `POST /auth` | Expects HTTP 200, checks that the response contains a string `token`, and saves it to `token`. |
| Create Booking | `POST /booking` | Creates a booking, expects HTTP 200, saves numeric `bookingid` as `bookingId`, and checks selected booking fields. |
| Unauthorized Update | `PUT /booking/{{bookingId}}` | Sends an update without the authentication cookie and expects HTTP 403 with `Forbidden`. Run after Create Booking. |
| Update Booking | `PUT /booking/{{bookingId}}` | Sends the authenticated update with `Cookie: token={{token}}`; checks HTTP 200, selected updated values, and the response's key fields. |
| Verify Booking | `GET /booking/{{bookingId}}` | Checks HTTP 200 and verifies the updated booking fields and dates. |
| Delete Booking | `DELETE /booking/{{bookingId}}` | Authenticates with the token and checks for HTTP 201 and the `Created` response. |

The create and update requests use the booking data in their request bodies. The update changes the total price and additional needs; the subsequent GET verifies those updated values. The negative update is deliberately placed between booking creation and the successful update so it can use the generated ID without affecting the authenticated update flow.

## Test strategy

The collection uses a chained happy path to validate token creation, booking creation, update authorization, persistence of changed values, and cleanup. Assertions check response status codes, important response properties and types, and selected business data. The unauthorized update is a negative scenario that checks the API rejects a write without a token.

For broader coverage, extend the collection with negative cases for malformed or missing authentication input, invalid booking payloads, nonexistent booking IDs, and unauthorized delete. Add schema-level assertions for the full documented response shapes, and verify a deleted booking cannot subsequently be retrieved. Since the service is live and stateful, keep test data distinct where practical and ensure cleanup runs even when an earlier assertion fails.

