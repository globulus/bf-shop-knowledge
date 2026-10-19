# Glossary

Every domain term, in Croatian and English, with the name code must use. New
terms are added by the synthesizer and reviewed in the PR that adds them.

| Croatian | English | Code name | Definition | Source |
|---|---|---|---|---|
| — | Product | `ProductResponse`, `productId` | An item for sale: id, SKU, name, description, category, active flag and current price. | mobile `lib/api/generated/catalog/models/product_response.dart` |
| — | SKU | `sku` | The product's stock-keeping code, shown when a product has no name. | mobile `lib/screens/products_screen.dart:74` |
| — | Category | `CategoryDto`, `categoryId` | A group of products with a name and a slug. | mobile `lib/api/generated/catalog/models/category_dto.dart` |
| — | Price | `PriceResponse`, `Money` | An amount in a three-letter currency; a product's price has a validity period (`validFrom`, `validTo`) and a history. | mobile `openapi/catalog.json` `Money`, `PriceResponse` |
| — | Order | `OrderResponse`, `OrderSummaryDto` | What a customer placed: lines, status, total amount, currency, timestamps. | mobile `lib/api/generated/orders/models/order_response.dart` |
| — | Order line | `OrderLineRequest`, `OrderLineResponse`, `CartLine` | One product with a quantity (1–999) in an order; priced by the backend at order time. | mobile `openapi/orders.json` `OrderLineRequest`; `lib/state/cart.dart:6` |
| — | Order status | `Status`, `OrderSummaryDtoStatus` | `PLACED`, `SHIPPED` or `CANCELLED`; only a placed order can be cancelled. | mobile `openapi/orders.json:29`, `lib/screens/my_orders_screen.dart:87` |
| — | Cart | `Cart` | App-side list of order lines not yet placed; shown as the "Order" tab. | mobile `lib/state/cart.dart:15`, `lib/app.dart:73` |
| — | Customer | `customerId` | Who orders; a string id sent with every order call. No login: the app uses `demo-customer`. | mobile `lib/config.dart:24-29` |
| — | Cancellation reason | `CancelOrderRequest.reason`, `cancellationReason` | Free text (up to 500 chars) stored with a cancelled order; the app sends "Cancelled in the app". | mobile `lib/data/api_shop_repository.dart:51` |
| — | Item count | `itemCount` | Sum of the quantities of an order's lines, shown in "My orders". | mobile `lib/api/generated/orders/models/order_summary_dto.dart:30` |
| — | My orders | `MyOrdersScreen` | The customer's own orders, newest first, first 50 only. | mobile `lib/screens/my_orders_screen.dart`, `README.md:63` |
| — | Product | `Product` (`ProductResponse`) | An item in the catalog, with SKU, name, category and current price. | web: `src/api/client.ts:12` |
| — | Category | `Category` (`CategoryDto`) | A group of products, with name and slug. | web: `src/api/client.ts:13` |
| — | SKU | `sku` | A product's stock-keeping code, searchable on the product list. | web: `openapi/catalog.json:613` |
| — | Price | `price` (`Money`: `amount`, `currency`) | A product's current price; a product without one can't be ordered. | web: `src/pages/OrderFormPage.tsx:24` |
| — | Cart | `Cart` | The quantities chosen on the order form, one per product, 0..999. | web: `src/lib/cart.ts:1` |
| — | Order | `Order` (`OrderResponse`), `OrderSummary` | A customer's purchase of one or more products. | web: `src/api/client.ts:14-15` |
| — | Order line | `lines`, `CartLine` (`productId`, `quantity`) | One product and its quantity in an order. | web: `src/lib/cart.ts:3` |
| — | Order status | `OrderStatus`: `PLACED`, `SHIPPED`, `CANCELLED` | Where an order is; only a placed order can be cancelled. | web: `src/pages/OrderListPage.tsx:8` |
| — | Customer ID | `customerId` | The text that identifies a customer while there is no login. | web: `src/lib/customer.ts:1-3` |
| — | Product | `Product` | An item in the catalog: SKU, name, optional description and category, active flag. | catalog-service `model/Product.java` |
| — | SKU | `sku` | A product's unique stock-keeping code, stored trimmed and upper-case. | catalog-service `service/CatalogService.java:56` |
| — | Category | `Category` | A named group of products with a unique URL-safe slug. | catalog-service `model/Category.java` |
| — | Slug | `slug` | A category's unique key of lower-case letters, digits and hyphens. | catalog-service `web/dto/CategoryDto.java:10` |
| — | Active product | `active` | A product shown in search by default; inactive ones only with `activeOnly=false`. | catalog-service `repository/ProductRepository.java:17` |
| — | Price | `Price` | A product's amount and currency for a period `validFrom` (inclusive) to `validTo` (exclusive, open while null). | catalog-service `model/Price.java` |
| — | Current price | `price` (in `ProductResponse`) | The price whose period contains now, or null if there is none. | catalog-service `repository/PriceRepository.java:16-24` |
| — | Price history | `priceHistory` | All of a product's prices, newest first. | catalog-service `service/CatalogService.java:79` |
| — | Order | `Order` | A customer's purchase of one or more products, with a status, a currency and a total. | orders-service `domain/Order.java` |
| — | Order line | `OrderLine` | One product in an order: catalog product ID, product name and unit price copied when the order is placed, and a quantity. | orders-service `domain/OrderLine.java` |
| — | Order status | `OrderStatus` | Lifecycle state of an order: `PLACED`, `SHIPPED` or `CANCELLED`. | orders-service `domain/OrderStatus.java` |
| — | Placed | `OrderStatus.PLACED` | State of a new order; the only state that can be shipped or cancelled. | orders-service `domain/OrderStatus.java` |
| — | Shipped | `OrderStatus.SHIPPED` | State set by the warehouse integration when the parcel leaves. | orders-service `api/OrderController.java:71` |
| — | Cancelled | `OrderStatus.CANCELLED` | State of an order cancelled before shipping, with an optional reason. | orders-service `domain/Order.java:80` |
| — | Cancellation reason | `cancellationReason` | Free text (up to 500 characters) saying why an order was cancelled. | orders-service `V2__add_cancellation_reason.sql` |
| — | Customer ID | `customerId` | Identifier of the customer an order belongs to, up to 64 characters, taken from the request. | orders-service `api/dto/CreateOrderRequest.java` |
| — | Catalog product | `CatalogProduct` | A product as catalog-service returns it: ID, SKU, name, active flag, current price. | orders-service `catalog/CatalogProduct.java` |
| — | SKU | `sku` | The catalog's stock-keeping code for a product. | orders-service `catalog/CatalogProduct.java` |
| — | Order confirmation | `sendOrderConfirmation` | Message to the customer that an order was placed, sent after the order is saved. | orders-service `notification/NotificationClient.java` |
