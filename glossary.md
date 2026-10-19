# Glossary

Every domain term, in Croatian and English, with the name code must use. New
terms are added by the synthesizer and reviewed in the PR that adds them.

| Croatian | English | Code name | Definition | Source |
|---|---|---|---|---|
| — | Product | `Product` (`ProductResponse`) | An item in the catalog, with SKU, name, category and current price. | web: `src/api/client.ts:12` |
| — | Category | `Category` (`CategoryDto`) | A group of products, with name and slug. | web: `src/api/client.ts:13` |
| — | SKU | `sku` | A product's stock-keeping code, searchable on the product list. | web: `openapi/catalog.json:613` |
| — | Price | `price` (`Money`: `amount`, `currency`) | A product's current price; a product without one can't be ordered. | web: `src/pages/OrderFormPage.tsx:24` |
| — | Cart | `Cart` | The quantities chosen on the order form, one per product, 0..999. | web: `src/lib/cart.ts:1` |
| — | Order | `Order` (`OrderResponse`), `OrderSummary` | A customer's purchase of one or more products. | web: `src/api/client.ts:14-15` |
| — | Order line | `lines`, `CartLine` (`productId`, `quantity`) | One product and its quantity in an order. | web: `src/lib/cart.ts:3` |
| — | Order status | `OrderStatus`: `PLACED`, `SHIPPED`, `CANCELLED` | Where an order is; only a placed order can be cancelled. | web: `src/pages/OrderListPage.tsx:8` |
| — | Customer ID | `customerId` | The text that identifies a customer while there is no login. | web: `src/lib/customer.ts:1-3` |
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
