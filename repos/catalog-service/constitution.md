# catalog-service — constitution

Rules for an agent that changes `catalog-service` (`globulus/bf-catalog-service`).
This is the draft of the repository's own `AGENTS.md`, for the maintainer to review.
Every path is relative to the repository root.

## The stack

- Spring Boot 3.4.13 (parent POM), Java 17, Maven (`pom.xml:6-11`, `pom.xml:20`).
- Maven Wrapper 3.3.4, `distributionType=only-script`, Maven 3.9.16
  (`.mvn/wrapper/maven-wrapper.properties`). There is no `maven-wrapper.jar`.
- PostgreSQL at runtime, Flyway for migrations, Spring Data JPA (Hibernate)
  (`pom.xml:32-62`).
- H2 in PostgreSQL mode for tests (`pom.xml:69-73`,
  `src/test/resources/application-test.properties:1`).
- springdoc-openapi 2.8.9 for the OpenAPI spec and Swagger UI (`pom.xml:21`, `pom.xml:52-56`).
- Spring Boot Actuator for health (`pom.xml:40-43`).

## Commands

All from the repository root. Tests need no database: they boot the app on H2
(`README.md:10`).

| Job | Command | Source |
|---|---|---|
| Install (fetch dependencies) | `./mvnw -B dependency:go-offline` | `Dockerfile:5` |
| Build the jar (no tests) | `./mvnw -B -DskipTests package` | `Dockerfile:7` (with `-q`) |
| All tests | `./mvnw -B test` | surefire config, `pom.xml:82-92` |
| Tests as CI runs them | `./mvnw -B verify` | `.github/workflows/ci.yml:21`, `README.md:10` |
| One test class | `./mvnw -B test -Dtest=CatalogServiceTest` | Maven Surefire |
| One test method | `./mvnw -B test -Dtest=CatalogServiceTest#duplicateSkuIsRejected` | Maven Surefire |
| Export the OpenAPI spec | `./mvnw -B -Popenapi test` | `pom.xml:96-105`, `ci.yml:23`, `README.md:25` |
| Run the app | `./mvnw spring-boot:run` (needs PostgreSQL) | `README.md:11` |
| Docker image | `docker build -t catalog-service .` | `README.md:43`, `Dockerfile` |
| Lint / format | none | No Checkstyle, Spotless, PMD or formatter plugin in `pom.xml`. |

### Globuli's configured commands, checked

- **install** (`./mvnw -B dependency:go-offline`, then
  `./mvnw -B test -Dtest=*#globuli_install -Dsurefire.failIfNoSpecifiedTests=false`):
  the first part matches `Dockerfile:5`. The second matches no test in the
  repository (no method `globuli_install` exists), so it only resolves the
  test-phase plugins and runs no test. That is consistent with its purpose.
- **build** (`./mvnw -B -DskipTests package`): matches `Dockerfile:7`.
- **test** (`./mvnw -B test`): runs the same unit tests as CI's
  `./mvnw -B verify` (there is no Failsafe plugin and no verify-phase check in
  `pom.xml`). It does **not** run `OpenApiSpecExportTest`, which CI runs in a
  second step (`ci.yml:22-23`) with `-Popenapi`. A change that breaks the
  OpenAPI spec generation passes `./mvnw -B test` and fails CI.
- **test_one** (`./mvnw -B test -Dtest={title}`): works for a class name or
  `Class#method`. `OpenApiSpecExportTest` is tagged `openapi`, and the
  default build excludes that tag (`pom.xml:24`, `pom.xml:86-87`); to run it
  alone use `./mvnw -B -Popenapi test -Dtest=OpenApiSpecExportTest`.
  Not verified: whether `-Dtest` alone overrides the tag exclusion.
- **lint** (none): correct; the repository has no lint or format tool.

## Layout

| What | Where |
|---|---|
| Application entry point | `src/main/java/com/shopfront/catalog/CatalogServiceApplication.java` |
| JPA entities | `src/main/java/com/shopfront/catalog/model/` |
| Spring Data repositories | `src/main/java/com/shopfront/catalog/repository/` |
| Business logic and domain exceptions | `src/main/java/com/shopfront/catalog/service/` |
| REST controllers, error handling | `src/main/java/com/shopfront/catalog/web/` |
| Request/response records | `src/main/java/com/shopfront/catalog/web/dto/` |
| OpenAPI metadata | `src/main/java/com/shopfront/catalog/config/OpenApiConfig.java` |
| Runtime configuration | `src/main/resources/application.properties` |
| Flyway migrations | `src/main/resources/db/migration/` (only `V1__init.sql`) |
| Tests | `src/test/java/com/shopfront/catalog/`, mirroring the main packages |
| Test configuration (profile `test`) | `src/test/resources/application-test.properties` |
| CI | `.github/workflows/ci.yml` |
| Container image | `Dockerfile`, `.dockerignore` |
| Generated output | `target/` (ignored, `.gitignore:1`); the spec lands in `target/openapi/catalog.json` |

There is no checked-in generated code.

## Conventions

- **Layering:** controller → service → repository. Controllers call
  `CatalogService`; the service owns transactions (`@Transactional` on the
  class, `@Transactional(readOnly = true)` on reads,
  `service/CatalogService.java:25-27`, `:39`). Exception: `CategoryController`
  uses `CategoryRepository` directly; its comment says categories "never got a
  service class" (`web/CategoryController.java:19`).
- **Dependency injection:** constructor injection with `private final` fields,
  no `@Autowired` in main code (`service/CatalogService.java:29-37`,
  `web/ProductController.java:31-35`).
- **DTOs:** Java records. Product DTOs are nested records in one holder class
  `ProductDtos` (`web/dto/ProductDtos.java`); categories use one record,
  `CategoryDto`, for both request and response (`web/dto/CategoryDto.java`).
  The service maps entities to DTOs in private methods (`toResponse`,
  `toPriceResponse`, `service/CatalogService.java:95-105`). No mapping library.
- **Validation:** Jakarta Bean Validation annotations on the DTO records
  (`@NotBlank`, `@Size`, `@Pattern`, `@Digits`, `@DecimalMin`), `@Valid` on
  `@RequestBody` parameters (`web/dto/ProductDtos.java:21-61`,
  `web/ProductController.java:54`). Column limits in the DTOs match the
  migration (for example SKU 64, name 255, description 2000).
- **Entities:** field access, `GenerationType.IDENTITY` ids, a `protected`
  no-arg constructor for JPA, a public constructor for the required fields,
  getters, and setters only for fields that may change
  (`model/Product.java`, `model/Price.java`). Many-to-one relations are
  `FetchType.LAZY`. Times are `java.time.Instant`, stored as
  `timestamp with time zone`, with Hibernate's JDBC time zone set to UTC
  (`application.properties:8`).
- **Schema ownership:** Flyway owns the schema; Hibernate only validates it
  (`spring.jpa.hibernate.ddl-auto=validate`, `application.properties:7`).
  Any entity change needs a new migration `V<n>__<description>.sql`.
- **Queries:** derived query methods where they fit
  (`repository/PriceRepository.java:14`, `:26`); JPQL in text blocks with
  `@Query` and named `@Param`s otherwise (`repository/ProductRepository.java:14-24`).
- **Normalization:** SKUs are trimmed and upper-cased on create; names are
  trimmed (`service/CatalogService.java:56`, `:71`).
- **Error handling:** the service throws unchecked domain exceptions,
  `NotFoundException(what, id)` and `DuplicateException(message)`
  (`service/`). `GlobalExceptionHandler` maps them to 404 `not_found` and
  409 `duplicate`, and validation failures to 400 `validation_failed` with
  `fieldErrors` as `"field: message"` strings (`web/GlobalExceptionHandler.java`).
  Every 4xx body is an `ErrorResponse(error, message, fieldErrors, timestamp)`
  (`web/ErrorResponse.java`). Error codes are lower snake case.
- **Paging:** `page` is clamped to ≥ 0 and `pageSize` to 1..200, sorted by
  name (`service/CatalogService.java:42`). Responses use the record
  `ProductPage(items, total, page, pageSize)`.
- **OpenAPI annotations:** `@Tag` on each controller, `@Operation` where a
  summary adds information (`web/ProductController.java:28`, `:38`, `:64`).
- **Tests:** JUnit 5, AssertJ and Mockito from `spring-boot-starter-test`.
  - Plain unit tests for entity logic, with JUnit assertions
    (`model/PriceTest.java`).
  - Service tests boot the full context on H2 with `@SpringBootTest`,
    `@ActiveProfiles("test")` and `@Transactional`, so each test rolls back
    (`service/CatalogServiceTest.java:26-29`). They create their data through
    the service and repositories; there are no SQL fixtures or seed files.
  - Controller slice tests use `@WebMvcTest(Controller.class)`, `MockMvc`,
    and `@MockBean` for the service, asserting with `jsonPath`
    (`web/ProductControllerTest.java`).
  - Full-stack API tests use `@SpringBootTest` + `@AutoConfigureMockMvc`
    without `@Transactional` (`web/CategoryApiTest.java`). Their data stays
    in the shared in-memory H2 database (`DB_CLOSE_DELAY=-1`) for the rest of
    the run, so they use unique values such as slug `brewing-api-test`.
  - Test method names are camelCase sentences (`duplicateSkuIsRejected`,
    `unknownProductIs404WithErrorBody`); test classes end in `Test`.
  - JSON request bodies are inline text blocks.
- **Configuration:** environment variables with defaults through
  `${VAR:default}` in `application.properties` (`DB_URL`, `DB_USER`,
  `DB_PASSWORD`, `PORT`).

## Don't touch

- **`src/main/resources/db/migration/V1__init.sql`**: a Flyway migration on
  `main`. Flyway checksums applied migrations; editing it breaks every
  database that already ran it. Add a new `V2__…` migration instead (and
  only when the plan includes a migration).
- **`springdoc.api-docs.version=openapi_3_0`** (`application.properties:11-13`):
  pinned on purpose. The comment says the clients were never checked against
  OpenAPI 3.1, "so leave it unless you're doing that".
- **The response shapes of the API**, in particular `ProductResponse.price`
  (`web/dto/ProductDtos.java:26-34`). The web and mobile clients copy the
  generated spec into their `specs/` folders (`README.md:31-32`), and
  orders-service relies on `price` in `GET /api/products/{id}` being the
  price valid now, or `null` (`README.md:36-38`). Changing them is an API
  contract change.
- **`ErrorResponse`** (`web/ErrorResponse.java`): the web app shows its
  `message` (`web/ErrorResponse.java:6`).
- **The spec output path `target/openapi/catalog.json`** and the `openapi`
  profile and tag (`pom.xml:22-24`, `:89`, `:96-105`): CI uploads that file
  as the artifact `catalog-openapi` (`ci.yml:24-27`).
- **`mvnw`, `mvnw.cmd`, `.mvn/wrapper/`**: the Maven Wrapper; CI and the
  Dockerfile call `./mvnw`.
- **`Dockerfile`'s port and health check** (`Dockerfile:12-15`): the image
  listens on 8080 and is health-checked at `/actuator/health`; the actuator
  exposure in `application.properties:17-18` must keep `health`.

## Known problems

- **N+1 price queries:** `search` runs one price query per product on the
  page. The TODO says it is "fine for a page of 20, not for the export job"
  (`service/CatalogService.java:45`). Not found: any export job in this
  repository.
- **Overlapping prices:** `addPrice` closes the open price only when the new
  `validFrom` is after the open price's start
  (`service/CatalogService.java:88-90`). A new price dated at or before the
  open price's start leaves two open prices; `findValidAt` then picks the one
  with the latest `validFrom` (`repository/PriceRepository.java:16-24`).
  No test covers that case.
- **`@MockBean`** in `web/ProductControllerTest.java:11`, `:33` is deprecated
  since Spring Boot 3.4 in favour of `@MockitoBean`.
- **`./mvnw -B test` skips the OpenAPI export test** that CI runs (see
  Commands above).
- **No image push:** CI builds the Docker image on `main` but does not push
  it; "TODO push to the registry once we have one for this team"
  (`.github/workflows/ci.yml:37`).
- Not found: tests tagged or documented as failing or flaky, `@Disabled`
  tests, or an issue list in the repository.
