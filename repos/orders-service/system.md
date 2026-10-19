# orders-service — place in the system

## What it is

The backend service that places, lists and cancels customer orders for the
Shopfront web shop and mobile app (`README.md:3-4`). It owns orders and their
lines. It does not own products: a line keeps the catalog's product ID and a copy
of the product name and unit price taken when the order is placed
(`docs/architecture.md:15-18`, `domain/OrderLine.java:33-39`).

```
 web (React) ─┐                          ┌─> catalog-service (product names, prices)
              ├─> orders-service ────────┤
 mobile ──────┤        │                 └─> notification client (logs only, SHOP-412)
 warehouse ───┘        v
 integration       PostgreSQL (database "orders")
```

(After `docs/architecture.md:7-13`, plus the warehouse caller of
`POST /{id}/ship` from `api/OrderController.java:71-72` and `docs/architecture.md:40`.)

## What it serves

### REST API, `/api/v1/orders` (`api/OrderController.java`)

| Method | Path | Does | Success | Errors |
|---|---|---|---|---|
| `POST` | `/api/v1/orders` | Place an order. Body `{"customerId", "lines": [{"productId", "quantity"}]}`; `customerId` 1–64 characters, 1–50 lines, quantity 1–999 | 201, `Location` header, `OrderResponse` | 400 validation, 422 product unknown / inactive / unpriced / other currency, 500 catalog error |
| `GET` | `/api/v1/orders?customerId=&status=&page=&size=` | List, newest first; `page` default 0, `size` default 20, clamped to 1–100 | 200, `PageResponse<OrderSummaryDto>` (`content`, `page`, `size`, `totalElements`, `totalPages`) | 400 for an unknown `status` value (Spring's conversion; not covered by a test) |
| `GET` | `/api/v1/orders/{id}` | One order with its lines | 200, `OrderResponse` | 404 |
| `POST` | `/api/v1/orders/{id}/cancel` | Cancel a `PLACED` order; optional body `{"reason"}` up to 500 characters | 200, `OrderResponse` | 404, 409 if shipped or already cancelled |
| `POST` | `/api/v1/orders/{id}/ship` | Mark a `PLACED` order shipped; for the warehouse integration | 200, `OrderResponse` | 404, 409 |

`docs/architecture.md:50` still lists cancel as `DELETE /api/v1/orders/{id}`; the
code has `POST …/cancel`. The code wins.

Errors are RFC 7807 problem details (`api/ApiExceptionHandler.java`). A 422 has an
extra `productId` property.

`OrderResponse` fields: `id`, `customerId`, `status`, `currency`, `totalAmount`,
`lines[]` (`productId`, `productName`, `quantity`, `unitPrice`, `lineTotal`),
`cancellationReason`, `createdAt`, `updatedAt` (`api/dto/OrderResponse.java`).
Dates are ISO-8601 strings (`application.yml:18-20`).

Authentication: none. `customerId` is taken from the request as is
(`docs/architecture.md:77`); there is no Spring Security dependency
(`build.gradle:28-45`). Not found: whether a gateway in front of the service adds
authentication.

### Other endpoints

- Swagger UI at `/swagger-ui.html`, OpenAPI JSON at `/v3/api-docs`
  (`README.md:10`, `application.yml:33-37`).
- Actuator `health` and `info` only (`application.yml:39-47`), with liveness and
  readiness probes enabled: `/actuator/health/liveness`,
  `/actuator/health/readiness`. The Docker HEALTHCHECK uses the readiness probe
  (`Dockerfile:20-21`).

## What it calls

| Dependency | How | Where |
|---|---|---|
| PostgreSQL, database `orders` | JDBC via Spring Data JPA; schema by Flyway | `application.yml:4-17`, `src/main/resources/db/migration/` |
| catalog-service | HTTP `GET {CATALOG_BASE_URL}/api/products/{id}`, connect and read timeout 2 s, no retry | `catalog/CatalogClient.java`, `catalog/CatalogClientConfig.java`, `application.yml:26-28` |
| Notifications | None yet. `LoggingNotificationClient` only logs; a notification-service client is planned (`TODO(SHOP-412)`) | `notification/LoggingNotificationClient.java:7` |

No message queue, cache or other service is used (`build.gradle:28-45`).

## Configuration

From `src/main/resources/application.yml` and `README.md:25-30`:

| Environment variable | Property | Default |
|---|---|---|
| `DB_URL` | `spring.datasource.url` | `jdbc:postgresql://localhost:5432/orders` |
| `DB_USER` | `spring.datasource.username` | set in `application.yml:6` (not copied here) |
| `DB_PASSWORD` | `spring.datasource.password` | set in `application.yml:7` (not copied here) |
| `CATALOG_BASE_URL` | `catalog.base-url` | `http://localhost:8081` |
| `PORT` | `server.port` | `8080` |
| `JAVA_OPTS` (container only) | JVM options | `-XX:MaxRAMPercentage=75` (`Dockerfile:19`) |

Fixed settings: `catalog.timeout: 2s`, `app.api-version: 1.4.0`, graceful
shutdown, `open-in-view: false`, `ddl-auto: validate`, log level `com.shopfront: INFO`.

Tests use profile `test` (`src/test/resources/application-test.yml`): in-memory H2
in PostgreSQL mode and `catalog.base-url: http://catalog.invalid`.

No Spring profiles other than `test` were found.

## Build and deployment

- CI (`.github/workflows/ci.yml`): on push to `main` and on every pull request.
  Ubuntu, Temurin 17, `./gradlew build -PexcludeTags=known-bug`, then
  `./gradlew generateOpenApi`. Uploads `build/openapi/orders.json` as artifact
  `orders-openapi`, and test reports on failure.
- Container (`Dockerfile`): two-stage build. `eclipse-temurin:17-jdk` builds with
  `./gradlew bootJar -x test`; `eclipse-temurin:17-jre` runs `app.jar` as a non-root
  user (uid 10001) on port 8080.
- Local stack: the `dev-env` repo starts everything with Docker Compose
  (`README.md:14-15`). Not part of this repository.
- Not found: deployment manifests (Kubernetes, Compose, Terraform) or a deploy step.
  CI builds and tests but neither publishes the image nor deploys.

## Contracts shared with other repositories

| Contract | Direction | Kept in step how |
|---|---|---|
| Orders API OpenAPI spec (`build/openapi/orders.json`, generated by `./gradlew generateOpenApi`, CI artifact `orders-openapi`) | Served to the web and mobile apps | The web and mobile repos keep a copy in their `specs/` folder; copy the new file there by hand when the API changes (`README.md:52-54`). |
| `POST /api/v1/orders/{id}/ship` | Called by the warehouse integration | Not found: how that integration is versioned or where it lives. |
| catalog-service `GET /api/products/{id}` response | Consumed | `catalog/CatalogProduct.java` mirrors the fields by hand; no shared client library (`CatalogProduct.java:7-10`). Unknown fields are ignored. |
| Notification-service API | Planned (SHOP-412) | Not yet used. |
| Database `orders` | Owned by this service | Flyway migrations in this repo. Other services don't share it: product IDs are not foreign keys, as the catalog lives in another database (`domain/OrderLine.java:33`). |
