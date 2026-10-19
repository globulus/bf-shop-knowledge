# orders-service — constitution

Rules for an agent that changes `globulus/bf-orders-service`. Draft of the
repository's `AGENTS.md`; the maintainer reviews it.

Read at `main`, by `RUN-20261019T085344Z-onboarder`. Every statement cites the
file it comes from. Paths are from the repository root.

## Stack

- Java 17 (`build.gradle:10-12`; CI uses Temurin 17, `.github/workflows/ci.yml:19-22`).
- Spring Boot 3.5.16, `io.spring.dependency-management` 1.1.7 (`build.gradle:3-4`).
- Gradle wrapper 8.14.4 (`gradle/wrapper/gradle-wrapper.properties:3`). Single
  project, no subprojects (`settings.gradle`).
- Spring Web, Spring Data JPA, Bean Validation, Actuator, Flyway (with the
  PostgreSQL module), springdoc-openapi 2.8.17, Lombok (compile only)
  (`build.gradle:24-45`).
- PostgreSQL at runtime (`org.postgresql:postgresql`), H2 in tests
  (`build.gradle:40-44`).

## Commands

Run every command from the repository root.

| Job | Command | Source | Notes |
|---|---|---|---|
| Install (resolve dependencies) | Not found in the repo: no `globuliResolve` task | `build.gradle` | Globuli's configured `./gradlew globuliResolve` is not defined by `build.gradle` or `settings.gradle`. The repo's own way to pre-fetch dependencies is `./gradlew dependencies`, as the `Dockerfile:7` does. See finding F-01. |
| Build (no tests) | `./gradlew assemble` | Gradle `java` plugin | Builds the boot jar into `build/libs/`. The `Dockerfile:10` uses `./gradlew bootJar -x test`. |
| Build and test, as CI does | `./gradlew build -PexcludeTags=known-bug` | `.github/workflows/ci.yml:27-28` | |
| All tests, as CI runs them | `./gradlew test -PexcludeTags=known-bug` | `build.gradle:47-54`, `ci.yml:26-28` | Skips tests tagged `known-bug` (and always `openapi`). |
| All tests, including known bugs | `./gradlew test` | `README.md:32-36` | Runs the `known-bug` test, which fails on purpose (see Known problems). Globuli's configured test command is this one; see finding F-02. |
| One test class | `./gradlew test --tests com.shopfront.orders.service.OrderServiceTest` | Gradle test filtering | A simple class name (`--tests OrderServiceTest`) also matches. |
| One test method | `./gradlew test --tests 'com.shopfront.orders.service.OrderServiceTest.createOrder_trimsCustomerId'` | Gradle test filtering | Tests have no `@DisplayName`, so a test's title is its method name. The method name alone, without the class, matches no test. See finding F-03. |
| Lint | `./gradlew check -x test` | Gradle `java` plugin | No lint or static-analysis plugin is applied (`build.gradle:1-5`): no Checkstyle, PMD, SpotBugs, Error Prone or Spotless. This command only compiles main and test code. See finding F-04. |
| Format | Not found | `build.gradle` | No formatter is configured, and there is no `.editorconfig`. Follow the style of the surrounding code (4-space indents, see Conventions). |
| OpenAPI spec | `./gradlew generateOpenApi` | `build.gradle:61-74`, `README.md:46-54` | Runs `OpenApiSpecExportTest` (tag `openapi`), which boots the app on H2 and writes `build/openapi/orders.json`. |
| Run locally | `./gradlew bootRun` | `README.md:17-23` | Needs PostgreSQL and catalog-service; environment variables in `system.md`. |
| Docker image | `docker build -t shopfront/orders-service .` | `README.md:56-61`, `Dockerfile` | |

Tests need no database and no network: the `test` profile uses in-memory H2 in
PostgreSQL mode (`src/test/resources/application-test.yml:3`), and the catalog is
mocked (`README.md:38-39`).

## Layout

| What | Where |
|---|---|
| Application entry point | `src/main/java/com/shopfront/orders/OrdersApplication.java` (`@SpringBootApplication`, `@ConfigurationPropertiesScan`, `@EnableAsync`) |
| Production code | `src/main/java/com/shopfront/orders/` with packages `api`, `api/dto`, `service`, `domain`, `repository`, `catalog`, `notification`, `config` (see `modules.md`) |
| Configuration | `src/main/resources/application.yml`; test overrides in `src/test/resources/application-test.yml` (profile `test`) |
| Database migrations (Flyway) | `src/main/resources/db/migration/` (`V1__create_orders.sql`, `V2__add_cancellation_reason.sql`) |
| Tests | `src/test/java/com/shopfront/orders/`, mirroring the main packages |
| Generated output | `build/` only (ignored by `.gitignore:3`), including `build/openapi/orders.json` |
| Generated code checked in | None found |
| CI | `.github/workflows/ci.yml` |
| Container | `Dockerfile`, `.dockerignore` |
| Docs | `README.md`, `docs/architecture.md`, `docs/KNOWN_ISSUES.md` |

## Conventions

Only what the code shows.

### Layering

- The controller calls `OrderService` only. The service returns API DTOs
  (`OrderResponse`, `PageResponse<OrderSummaryDto>`), not entities
  (`src/main/java/com/shopfront/orders/service/OrderService.java:50,90`).
- Entity-to-DTO mapping is in the package-private `final class OrderMapper`
  with static methods (`service/OrderMapper.java`). There is no mapping library.
- State changes live on the entity: `Order.cancel(reason, now)` and
  `Order.markShipped()` check the current status and throw
  `InvalidOrderStateException` (`domain/Order.java:80-94`). The allowed
  transitions are in `OrderStatus.isCancellable()` (`domain/OrderStatus.java`).
- `Order.addLine` sets the back-reference and recalculates the total
  (`domain/Order.java:67-78`). Add lines through it, never to `getLines()` directly.

### Naming

- Request and response types are Java `record`s in `api/dto`, named
  `*Request` and `*Response`; the list row is `OrderSummaryDto`
  (`api/dto/`).
- Exceptions are named `*Exception`, extend `RuntimeException`, and live in
  `service` (`service/OrderNotFoundException.java`).
- Unit tests are named `*Test`; Spring Boot integration tests are named `*IT`
  (`api/OrderControllerIT.java`, `notification/OrderConfirmationIT.java`). Both
  run in the same `test` task (`build.gradle:51-59`).
- Test methods: `action_expectedOutcome` in unit tests
  (`OrderServiceTest.createOrder_rejectsUnknownProduct`) and plain camelCase
  sentences in others (`OrderControllerIT.cancelTwiceIs409`,
  `OrderTest.shippedOrderCannotBeCancelled`).
- Migrations: `V<n>__<snake_case_description>.sql`, lower-case SQL
  (`db/migration/`).

### Dependency injection and configuration

- Constructor injection with explicit constructors and `private final` fields;
  no `@Autowired` in production code (`service/OrderService.java:39-47`,
  `api/OrderController.java:34-36`).
- Lombok is used on JPA entities only (`@Getter @Setter @NoArgsConstructor`,
  `domain/Order.java:29-31`). Other classes write getters and constructors by hand
  (`service/ProductUnavailableException.java`).
- Typed configuration is a `@ConfigurationProperties` record with
  `@DefaultValue`s (`catalog/CatalogProperties.java`), picked up by
  `@ConfigurationPropertiesScan`.
- Time: a `Clock` bean (`config/AppConfig.java:16-19`) is injected where the
  service needs "now" (`OrderService.cancelOrder`). The JPA callbacks
  `@PrePersist`/`@PreUpdate` use `Instant.now()` instead
  (`domain/Order.java:96-108`).
- Money is `BigDecimal`, scale 2, `HALF_UP` on totals (`domain/Order.java:73-78`);
  columns are `numeric(12, 2)`. Times are `Instant`, stored as
  `timestamp with time zone`, Hibernate JDBC time zone UTC
  (`application.yml:12-15`).

### Errors

- The API answers errors as RFC 7807 problem details. `ApiExceptionHandler`
  extends `ResponseEntityExceptionHandler` (so validation errors become 400
  problem details) and maps `OrderNotFoundException` → 404,
  `InvalidOrderStateException` → 409, `ProductUnavailableException` → 422 with a
  `productId` property (`api/ApiExceptionHandler.java`).
- A new error case gets its own exception in `service` and its own handler
  method there.
- Catalog errors other than 404 propagate (no catch), so an outage gives a 500
  (`catalog/CatalogClient.java:24-42`, `docs/architecture.md:56-61`).
- Notification failures are caught and logged at WARN, never rethrown
  (`notification/OrderConfirmationListener.java:26-33`).

### Validation

- Bean Validation annotations on DTO record components (`@NotBlank`, `@Size`,
  `@NotEmpty`, `@Min`, `@Max`, `@NotNull`, `List<@Valid …>`)
  (`api/dto/CreateOrderRequest.java`, `api/dto/OrderLineRequest.java`).
  The controller is `@Validated` and takes `@Valid @RequestBody`
  (`api/OrderController.java:29,40`).

### Logging

- SLF4J, `private static final Logger log = LoggerFactory.getLogger(X.class)`,
  parameterized messages (`service/OrderService.java:31,82`).

### Persistence

- Spring Data JPA repository interfaces with JPQL `@Query` text blocks
  (`repository/OrderRepository.java`).
- Flyway owns the schema; Hibernate only validates it (`ddl-auto: validate`,
  `application.yml:10-11`). A schema change is a new `V<n>__*.sql` file plus the
  matching entity change; both must agree or the app fails to start.
- `open-in-view: false` (`application.yml:9`): load lazy associations inside the
  service's `@Transactional` method.
- Service methods are `@Transactional`; reads are `@Transactional(readOnly = true)`
  (`service/OrderService.java:49,89,98`).

### API documentation

- Controllers carry springdoc annotations `@Tag` and `@Operation`
  (`api/OrderController.java:28,39`). The OpenAPI document's title and version come
  from `config/AppConfig.java:21-29` and `app.api-version` in `application.yml:30-31`.

### Tests

- JUnit 5 (JUnit Platform), AssertJ (`assertThat`, `assertThatThrownBy`), Mockito,
  all from `spring-boot-starter-test` (`build.gradle:42-44`).
- Unit tests: `@ExtendWith(MockitoExtension.class)`, `@Mock` fields, the class
  under test built by hand in `@BeforeEach`, with a fixed `Clock`
  (`service/OrderServiceTest.java:35-51`). Small `given…` helper methods set up
  mocks (`OrderServiceTest.java:163-174`).
- Domain tests use plain `new Order()` with no Spring (`domain/OrderTest.java`).
- HTTP client tests bind `MockRestServiceServer` to a `RestClient.Builder`
  (`catalog/CatalogClientTest.java:22-27`).
- Integration tests: `@SpringBootTest`, `@AutoConfigureMockMvc`,
  `@ActiveProfiles("test")`, `MockMvc` with JSON in text blocks and `jsonPath`
  assertions; the catalog is replaced with `@MockitoBean CatalogClient`
  (`api/OrderControllerIT.java:30-51`).
- Integration tests share one in-memory H2 database across the Spring context
  (`DB_CLOSE_DELAY=-1`) and don't clean up. They isolate data with unique
  customer IDs (`OrderControllerIT.listIsPagedAndFilteredByCustomer` uses a
  random UUID). Do the same in new integration tests.
- Tags: `known-bug` marks a test for a documented, unfixed bug
  (`docs/KNOWN_ISSUES.md:3-5`); `openapi` marks the spec export, which the `test`
  task always excludes (`build.gradle:53`).

### Style

- 4-space indents, imports one per line, no wildcard imports, static imports
  after the others (`service/OrderServiceTest.java:1-33`). No formatter enforces this.

## Don't touch

| What | Why |
|---|---|
| `src/main/resources/db/migration/V1__create_orders.sql`, `V2__add_cancellation_reason.sql` | Flyway migrations. Whether they are released is not visible in the repo, but V2 alters V1's table, so treat both as applied. Flyway checks the checksum of applied migrations; editing one breaks startup on existing databases. Change the schema only with a new `V3__…` file, and only when the work item's plan allows a migration. |
| `gradle/wrapper/gradle-wrapper.jar`, `gradle/wrapper/gradle-wrapper.properties`, `gradlew`, `gradlew.bat` | Gradle wrapper; changing it changes the build toolchain. |
| `.github/workflows/ci.yml` | CI configuration. It also publishes the `orders-openapi` artifact (`ci.yml:30-38`) that the web and mobile repos use. |
| The `-PexcludeTags` mechanism in `build.gradle:47-59` and the `known-bug` tag | CI depends on it to stay green while known bugs stay unfixed (`docs/KNOWN_ISSUES.md`). Remove the tag from a test only together with the fix and the `KNOWN_ISSUES.md` entry, as that file says. |
| `generateOpenApi` task (`build.gradle:61-74`) and `src/test/java/com/shopfront/orders/OpenApiSpecExportTest.java` | CI runs `generateOpenApi` and uploads `build/openapi/orders.json`; it fails if the file is missing (`ci.yml:30-38`). |
| The API under `/api/v1/orders` (paths, request and response shapes, status codes) | Contract with the web and mobile apps, which keep a copy of the spec in their `specs/` folders (`README.md:52-54`), and with the warehouse integration (`POST /{id}/ship`, `api/OrderController.java:71-75`). |
| `catalog/CatalogProduct.java` fields | Hand-kept mirror of catalog-service's `GET /api/products/{id}` response; there is no shared client library (`catalog/CatalogProduct.java:7-10`). Change it only together with catalog-service. |
| `Dockerfile` | Deployment build. Its HEALTHCHECK calls `/actuator/health/readiness` (`Dockerfile:20-21`), which needs the probe settings in `application.yml:39-47`. |

## Known problems

- **Known bug, failing test.** Duplicate product lines in one create request keep
  the last quantity instead of the sum (`docs/KNOWN_ISSUES.md:7-21`). The cause is
  the merge function `(first, second) -> second` in
  `service/OrderService.java:52-57`. The test
  `OrderServiceTest.createOrder_sumsQuantitiesOfDuplicateProductLines`
  (`src/test/java/com/shopfront/orders/service/OrderServiceTest.java:94-107`) is
  tagged `known-bug` and fails under plain `./gradlew test`. CI skips it.
- **Flaky test.** `OrderConfirmationIT.confirmationIsSentOnceOrderIsPlaced`
  (`src/test/java/com/shopfront/orders/notification/OrderConfirmationIT.java:40-52`)
  waits a fixed 200 ms for an async listener, while its fake gateway sleeps a
  random 0–250 ms (`OrderConfirmationIT.java:48-49,72`). It fails whenever the fake
  sleeps for more than the wait. `README.md:44` says it "fails now and then on slow
  machines". It is not tagged, so CI and `./gradlew test` both run it.
- **Out-of-date doc.** `docs/architecture.md:50` lists cancel as
  `DELETE /api/v1/orders/{id}`. The code has `POST /api/v1/orders/{id}/cancel` with
  an optional `{"reason": …}` body (`api/OrderController.java:64-69`), and
  `V2__add_cancellation_reason.sql:1` records the move. The code wins.
  `docs/architecture.md` says "Last reviewed: March 2025".
- **No authentication or authorization.** `customerId` is trusted from the request
  (`docs/architecture.md:77`). No Spring Security dependency (`build.gradle:28-45`).
  `GET /api/v1/orders` without `customerId` lists every customer's orders
  (`service/OrderService.java:94`, `repository/OrderRepository.java:13-17`).
  `POST /{id}/ship` is "warehouse only" by comment, not by a check
  (`api/OrderController.java:71-72`). See finding F-05.
- **TODOs and planned work in the docs.**
  - `TODO(SHOP-412)`: replace `LoggingNotificationClient` with the
    notification-service client (`notification/LoggingNotificationClient.java:7`).
  - No idempotency key on `POST /api/v1/orders`; double clicks create two orders
    (`docs/architecture.md:75`).
  - No retry or circuit breaker around `CatalogClient` (`docs/architecture.md:76`).
- **Query per order in the list.** `OrderMapper.toSummary` reads each order's lazy
  `lines` to count items (`service/OrderMapper.java:27`), so one list page loads the
  lines of each order on it separately (at most 100 orders, `OrderService.MAX_PAGE_SIZE`).
- **Catalog calls one by one.** `createOrder` calls the catalog once per distinct
  product, in sequence, inside the database transaction, each with a 2 s timeout
  (`service/OrderService.java:62-79`, `application.yml:26-28`). An order can have up
  to 50 lines (`api/dto/CreateOrderRequest.java:12`).
- **Default credentials.** `src/main/resources/application.yml:5-7` and `README.md`
  hold default database credentials used when `DB_USER`/`DB_PASSWORD` are unset.
  They are not copied here.
