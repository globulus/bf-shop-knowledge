# Glossary

Every domain term, in Croatian and English, with the name code must use. New
terms are added by the synthesizer and reviewed in the PR that adds them.

| Croatian | English | Code name | Definition | Source |
|---|---|---|---|---|
| — | Product | `Product` | An item in the catalog: SKU, name, optional description and category, active flag. | catalog-service `model/Product.java` |
| — | SKU | `sku` | A product's unique stock-keeping code, stored trimmed and upper-case. | catalog-service `service/CatalogService.java:56` |
| — | Category | `Category` | A named group of products with a unique URL-safe slug. | catalog-service `model/Category.java` |
| — | Slug | `slug` | A category's unique key of lower-case letters, digits and hyphens. | catalog-service `web/dto/CategoryDto.java:10` |
| — | Active product | `active` | A product shown in search by default; inactive ones only with `activeOnly=false`. | catalog-service `repository/ProductRepository.java:17` |
| — | Price | `Price` | A product's amount and currency for a period `validFrom` (inclusive) to `validTo` (exclusive, open while null). | catalog-service `model/Price.java` |
| — | Current price | `price` (in `ProductResponse`) | The price whose period contains now, or null if there is none. | catalog-service `repository/PriceRepository.java:16-24` |
| — | Price history | `priceHistory` | All of a product's prices, newest first. | catalog-service `service/CatalogService.java:79` |
