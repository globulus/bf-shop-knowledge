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
