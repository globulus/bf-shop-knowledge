# Glossary

Every domain term, in Croatian and English, with the name code must use. New
terms are added by the synthesizer and reviewed in the PR that adds them.

| Croatian | English | Code name | Definition | Source |
|---|---|---|---|---|
| Narudžba | Order | `order` | A customer's order of products from the shop; read through `GET /api/v1/orders/{id}`. | LIN-GTB-3 |
| Stavka narudžbe | Order line | `orderLine` | One product in an order, with its product name, quantity and line total. | LIN-GTB-3 |
| Status narudžbe | Order status | `status` | Where an order stands: placed, shipped or cancelled (the statuses the issue names). | LIN-GTB-3 |
| Zaprimljena | Placed | `placed` | Order status: the order has been placed and not yet shipped or cancelled. | LIN-GTB-3 |
| Poslana | Shipped | `shipped` | Order status: the order has been shipped. | LIN-GTB-3 |
| Otkazana | Cancelled | `cancelled` | Order status: the order has been cancelled. | LIN-GTB-3 |
| Moje narudžbe | My orders | `myOrders` | The app screen that lists the customer's orders. | LIN-GTB-3 |
| Zaslon narudžbe | Order screen | `orderDetail` | The app screen of one order with its lines; new with LIN-GTB-3. | LIN-GTB-3 |
| Oznaka statusa | Status chip | `statusChip` | The coloured label showing an order's status: placed amber, shipped green, cancelled grey. | LIN-GTB-3 |
