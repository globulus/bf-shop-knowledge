# orders-service — modules

One Gradle project (`settings.gradle`), one Spring Boot application. Its parts are
the Java packages under `src/main/java/com/shopfront/orders/`. Paths below are
relative to that folder unless they start with `src/`.

## Application (`com.shopfront.orders`)

**Does:** starts the Spring Boot app and switches on configuration-property
scanning and `@Async`.

**Main files:** `OrdersApplication.java`.

**Depends on:** every package below, through component scanning.

## api

**Does:** the REST API under `/api/v1/orders`: place, list, get, cancel and ship
orders. Maps service exceptions to RFC 7807 problem details.

**Main files:**
- `api/OrderController.java`: endpoints `POST /api/v1/orders`,
  `GET /api/v1/orders`, `GET /api/v1/orders/{id}`, `POST /api/v1/orders/{id}/cancel`,
  `POST /api/v1/orders/{id}/ship`.
- `api/ApiExceptionHandler.java`: 404, 409, 422 handlers; validation errors via
  `ResponseEntityExceptionHandler`.
- `api/dto/`: request and response records `CreateOrderRequest`,
  `OrderLineRequest`, `CancelOrderRequest`, `OrderResponse` (with nested
  `OrderLineResponse`), `OrderSummaryDto`, `PageResponse<T>`.

**Depends on:** `service` (`OrderService` and its exceptions), `domain`
(`OrderStatus` as a query parameter and response field), Spring Web, Bean
Validation, springdoc annotations, Spring Data's `Page` (in `PageResponse.of`).

**Tests:** `src/test/java/com/shopfront/orders/api/OrderControllerIT.java`.

## service

**Does:** the use cases, each in a transaction. Placing an order merges lines by
product, prices each line from the catalog, checks that the product exists, is
active, has a price and shares the order's currency, saves the order and
publishes `OrderPlacedEvent`. Also lists (paged, newest first, page size 1–100),
gets, cancels and marks orders shipped.

**Main files:**
- `service/OrderService.java`
- `service/OrderMapper.java`: entity → DTO mapping.
- `service/OrderNotFoundException.java`, `service/InvalidOrderStateException.java`,
  `service/ProductUnavailableException.java`.

**Depends on:** `repository`, `catalog` (`CatalogClient`, `CatalogProduct`),
`domain`, `notification` (`OrderPlacedEvent`), `api.dto` (it returns API DTOs),
Spring's `ApplicationEventPublisher`, the `Clock` bean from `config`.

**Tests:** `src/test/java/com/shopfront/orders/service/OrderServiceTest.java`
(includes the `known-bug` test).

## domain

**Does:** the JPA entities and the order state machine. `Order` holds its lines,
recalculates the total when a line is added, and allows `PLACED → SHIPPED` and
`PLACED → CANCELLED` only. `OrderLine` stores the catalog product ID and a copy of
the product name and unit price taken when the order was placed.

**Main files:** `domain/Order.java` (table `orders`), `domain/OrderLine.java`
(table `order_lines`), `domain/OrderStatus.java` (`PLACED`, `SHIPPED`, `CANCELLED`).

**Depends on:** Jakarta Persistence, Lombok, and `service.InvalidOrderStateException`
(the entity throws it).

**Tests:** `src/test/java/com/shopfront/orders/domain/OrderTest.java`.

## repository

**Does:** data access for orders. `search` filters by optional customer ID and
status, paged.

**Main files:** `repository/OrderRepository.java` (`JpaRepository<Order, Long>`).

**Depends on:** `domain`, Spring Data JPA.

## catalog

**Does:** the HTTP client for catalog-service. Reads one product by ID; a 404
gives an empty result, any other error propagates.

**Main files:**
- `catalog/CatalogClient.java`: `GET /api/products/{id}`.
- `catalog/CatalogClientConfig.java`: the `catalogRestClient` `RestClient` bean,
  with connect and read timeouts.
- `catalog/CatalogProperties.java`: `catalog.base-url`, `catalog.timeout`.
- `catalog/CatalogProduct.java`: the fields of catalog-service's product response
  that orders use (`id`, `sku`, `name`, `active`, `price.amount`, `price.currency`).

**Depends on:** Spring's `RestClient`; catalog-service at runtime.

**Tests:** `src/test/java/com/shopfront/orders/catalog/CatalogClientTest.java`.

## notification

**Does:** sends an order confirmation after the order's transaction commits, on
another thread. Today the only `NotificationClient` writes a log line
(`TODO(SHOP-412)`).

**Main files:**
- `notification/OrderPlacedEvent.java`: record with order ID, customer ID, total,
  currency.
- `notification/OrderConfirmationListener.java`: `@Async @TransactionalEventListener`;
  catches and logs failures.
- `notification/NotificationClient.java`: interface.
- `notification/LoggingNotificationClient.java`: the current implementation.

**Depends on:** Spring events, transactions and `@Async` (enabled in
`OrdersApplication`).

**Tests:** `src/test/java/com/shopfront/orders/notification/OrderConfirmationIT.java`
(flaky, see `constitution.md`).

## config

**Does:** application-wide beans: a UTC `Clock` and the OpenAPI document's info
(title "Orders API", version from `app.api-version`).

**Main files:** `config/AppConfig.java`.

**Depends on:** springdoc / swagger-models.

## Database schema (`src/main/resources/db/migration`)

**Does:** Flyway migrations for PostgreSQL; also run on H2 in tests.
- `V1__create_orders.sql`: tables `orders` and `order_lines`, indexes on
  `orders.customer_id` and `order_lines.order_id`.
- `V2__add_cancellation_reason.sql`: `orders.cancellation_reason`,
  `orders.cancelled_at`.

## OpenAPI export (test sources)

**Does:** writes the API spec to `build/openapi/orders.json` without a running
server.

**Main files:** `src/test/java/com/shopfront/orders/OpenApiSpecExportTest.java`
(tag `openapi`), the `generateOpenApi` task in `build.gradle:61-74`.

**Depends on:** the whole application context on the `test` profile.
