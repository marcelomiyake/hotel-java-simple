# Stays · Hotel Reservation System

A hotel room reservation demo based on the [ByteByteGo Hotel Reservation System](https://bytebytego.com/courses/system-design-interview/hotel-reservation-system), with the visual flow handed off from OpenDesign.

## Screenshot

The screenshot shows the production React build served from a local preview, not from the Kind deployment; the API-backed flows and the Lighthouse audit were not verified for it.

![Stays hotel reservation search page](docs/screenshots/hotel-reservation-home.png)

## Case study and known defects

This is the **Java simple** implementation in [One Hotel Design, Five AI Implementations](https://marcelomiyake.com.br/posts/five-ai-hotel-implementations/), which compares five AI-built versions of the same hotel design, and in its [Part II on the cost of a new feature](https://marcelomiyake.com.br/posts/five-ai-hotel-implementations-part-2/). Part I inspected commit [`2945cbb`](https://github.com/marcelomiyake/hotel-java-simple/commit/2945cbb7fdd929d29ec1671654d6dd09486d15a0).

The study found:

- **The packaged services do not start.** The service JARs have no `Main-Class` entry because the POM does not bind the Spring Boot `repackage` goal; launching the reservation JAR with `java -jar` failed with `no main manifest attribute`. The Docker images use the same `java -jar` entry point, so expect the backend pods in the Kind deployment below to fail at startup until this is fixed.
- Launched from the classpath instead, the reservation service failed because no `RestClient.Builder` bean was available. The integration tests replace those HTTP clients with mocks, so they pass anyway.
- From source review, not reproduced: cancellation reads the reservation without a row lock before releasing inventory; booking never checks the guest count against room capacity; reservation lookup and cancellation routes are unprotected; and the duplicate-key fallback returns an existing reservation without re-checking that the request matches.
- Part II: a journey stays `STARTED` when the browser never reports leaving, because there is no server-side expiry.

No later commit fixes these. The passing tests and SonarQube Cloud results below do not cover them. This is a local learning implementation, not a production system.

## What it does

- Search Lisbon hotels by destination, dates, and guest count; compare live room availability and daily rates.
- Reserve by room type. PostgreSQL stores inventory per date, permits up to 10% overbooking, and uses version-checked updates to protect concurrent bookings.
- Record reservation journeys in PostgreSQL and mark the last viewed booking screen when a guest abandons before payment.
- Reuse a reservation UUID as an idempotency key. Repeating the same request returns the existing reservation; changing its details returns a conflict.
- View reservation history by email and cancel a confirmed reservation. Cancellation releases inventory and records a simulated refund.
- Give staff a protected screen for editing room types, capacity, inventory, and nightly rates.

The seeded catalog has two Lisbon hotels. Payments are intentionally simulated: the demo does not collect card details or contact a hotel.

## Architecture

| Component | Responsibility |
| --- | --- |
| `hotel-service` | Hotel and room type catalog, including staff updates |
| `rate-service` | Per-night quotes, weekday/weekend rates, and staff rate schedules |
| `reservation-service` | Search, date inventory, bookings, idempotency, cancellation, reservation journey analytics |
| `payment-service` | Idempotent demo charges and refunds |
| `service-common` | Shared API errors, health endpoint, and staff-key filter |
| React web app | Search, stay details, checkout, history, and staff flow |
| PostgreSQL | Catalog, rates, payments, and reservation inventory schemas |

The browser talks to the React app through Nginx. Nginx routes API calls to the four services. Each microservice has two Kind replicas; the web deployment also has two replicas. PostgreSQL uses one StatefulSet replica and a persistent volume for this local demo.

## Run on local Kind

Requirements: Docker, Kind, kubectl, and `openssl`.

```bash
./scripts/kind-up.sh
```

Open [http://localhost:8080](http://localhost:8080). The script creates the `hotel-reservation` cluster, builds and loads local images, and deploys the app. It generates local database and staff keys in `.kind-db-password` and `.kind-admin-key`; the staff screen uses the value from `.kind-admin-key`. Those files are ignored by Git.

```bash
./scripts/kind-down.sh
```

To inspect the deployment:

```bash
kubectl -n hotel-reservation get deployments,pods,services
```

## Main API routes

| Route | Purpose |
| --- | --- |
| `GET /api/search?destination=Lisbon&checkIn=YYYY-MM-DD&checkOut=YYYY-MM-DD&guests=2` | Search availability and rates |
| `GET /api/hotels?destination=Lisbon` | Read hotel catalog |
| `GET /api/rates/quote?roomTypeId=…&checkIn=…&checkOut=…` | Quote every night in a stay |
| `POST /api/reservations` | Create or replay a reservation |
| `POST /api/reservation-journeys/events` | Record a started journey, viewed booking screen, or abandonment event |
| `GET /api/reservations?email=…` | Read a guest’s reservation history |
| `DELETE /api/reservations/{id}` | Cancel a reservation and refund the demo payment |
| `POST /api/admin/hotels/{hotelId}/room-types` | Add a room type (requires `X-Admin-Key`) |
| `PUT /api/admin/room-types/{id}` | Update a room type (requires `X-Admin-Key`) |
| `PUT /api/admin/inventory` | Update future inventory (requires `X-Admin-Key`) |
| `PUT /api/admin/rates` | Update one nightly rate (requires `X-Admin-Key`) |

The local API is for a design exercise, not a production payment or hotel booking service.

## Quality checks

```bash
# Frontend checks
cd frontend
npm ci
npm test
npm run build
npm run lint

# Java 25 + PostgreSQL integration tests (Docker socket required)
cd ..
docker run --rm --network=host \
  -v "$PWD":/workspace \
  -v "$HOME/.m2":/root/.m2 \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -w /workspace maven:3.9-eclipse-temurin-25 mvn -B verify
```

Coverage is reported by Vitest and JaCoCo. The latest verified frontend line coverage is **92.02%** and combined Java line coverage is **85.01%**. SonarCloud results and the local visual QA status are recorded in the analysis below.

SonarQube Cloud project: [Hotel Reservation System](https://sonarcloud.io/project/overview?id=marcelomiyake_hotel-dry-kiss-yagni). Project configuration is in `sonar-project.properties`.

## Reservation abandonment analytics

Reservation tracking starts when a guest opens hotel details, follows the guest details and checkout screens, and ends as `COMPLETED` after payment succeeds or `ABANDONED` when the guest leaves the flow or closes the page. PostgreSQL stores an opaque journey UUID, the last screen (`DETAILS` or `CHECKOUT`), status, and timestamps. No guest name, email, dates, hotel, or room data is copied into the analytics table. The existing page title, description, social metadata, and structured data remain in place; a public `robots.txt` is also served for preview and deployment.

## AI build record

This implementation was generated with Codex (GPT-6 Luna, max effort). The original prompts, token usage, cost estimates, and change counts for the initial build and the reservation-abandonment feature are in [docs/ai-build-record.md](docs/ai-build-record.md).
