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
