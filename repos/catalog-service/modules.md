# catalog-service — modules

One Maven module (`pom.xml`, artifact `com.shopfront:catalog-service`,
version `2.1.0-SNAPSHOT`). Its Java code is split by layer into packages
under `src/main/java/com/shopfront/catalog/`. Paths below are relative to
that folder.

## Application (`com.shopfront.catalog`)

Starts the Spring Boot application.

- Main file: `CatalogServiceApplication.java`.
- Depends on: every package below, through component scanning.

## Products (`web/ProductController`, `service/CatalogService`)

Search, read, create and update products. A product has a SKU (unique,
stored upper-case), a name, an optional description, an optional category
and an `active` flag. Search filters by category, by text (case-insensitive
match on name or SKU) and by active status, sorted by name, paged
(page size 1..200, default 20).

- Main files: `web/ProductController.java` (`/api/products`),
  `service/CatalogService.java`, `model/Product.java`,
  `repository/ProductRepository.java`, `web/dto/ProductDtos.java`.
- Depends on: Categories (to resolve `categoryId`), Prices (to fill the
  current `price` of each product), Errors.
- Tests: `src/test/java/com/shopfront/catalog/service/CatalogServiceTest.java`,
  `src/test/java/com/shopfront/catalog/web/ProductControllerTest.java`.

## Prices (`model/Price`, `repository/PriceRepository`)

A product's prices over time. Each price has an amount (numeric 12,2), a
three-letter upper-case currency code and a validity period `validFrom`
(inclusive) to `validTo` (exclusive, `null` while open)
(`model/Price.java:16-19`, `:54-56`). Adding a price closes the open one at
the new price's `validFrom` (`service/CatalogService.java:84-93`). The
current price is the one valid now (`PriceRepository.findValidAt`).

- Main files: `model/Price.java`, `repository/PriceRepository.java`; the
  logic is in `service/CatalogService.java` (`addPrice`, `priceHistory`,
  `toResponse`); endpoints `GET` and `POST /api/products/{id}/prices` in
  `web/ProductController.java:63-73`.
- Depends on: Products.
- Tests: `src/test/java/com/shopfront/catalog/model/PriceTest.java`,
  `CatalogServiceTest.addingPriceClosesThePreviousOne`.

## Categories (`web/CategoryController`)

List categories (sorted by name) and create one. A category has a name and a
unique slug (`[a-z0-9-]+`). No update or delete. No service class: the
controller uses the repository directly (`web/CategoryController.java:19`).

- Main files: `web/CategoryController.java` (`/api/categories`),
  `model/Category.java`, `repository/CategoryRepository.java`,
  `web/dto/CategoryDto.java`.
- Depends on: Errors (`DuplicateException`).
- Tests: `src/test/java/com/shopfront/catalog/web/CategoryApiTest.java`.

## Errors (`web/GlobalExceptionHandler`, `service/*Exception`)

Turns domain exceptions and validation failures into the common error body.

- Main files: `service/NotFoundException.java`,
  `service/DuplicateException.java`, `web/GlobalExceptionHandler.java`,
  `web/ErrorResponse.java`.
- Mapping: `NotFoundException` → 404 `not_found`; `DuplicateException` →
  409 `duplicate`; `MethodArgumentNotValidException` → 400
  `validation_failed` with `fieldErrors`.
- Depends on: nothing in the repository.

## OpenAPI (`config/OpenApiConfig`)

Spec metadata (title "Catalog API", version "2.1", server `/`) and the spec
export. springdoc serves the spec at `/v3/api-docs` and Swagger UI at
`/swagger-ui/index.html` (`README.md:20`). The test
`src/test/java/com/shopfront/catalog/OpenApiSpecExportTest.java` (tag
`openapi`, run by `./mvnw -Popenapi test`) writes it to
`target/openapi/catalog.json`.

- Main files: `config/OpenApiConfig.java`, the `springdoc.*` properties in
  `src/main/resources/application.properties:11-15`, the `openapi` profile in
  `pom.xml:96-105`.
- Depends on: the controllers, which it documents.

## Database schema (`src/main/resources/db/migration/`)

Flyway migrations. `V1__init.sql` creates `categories`, `products` and
`prices` and the index `idx_prices_product (product_id, valid_from)`.

- Depends on: PostgreSQL at runtime; H2 in PostgreSQL mode in tests.
