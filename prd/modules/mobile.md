# Mobile

## Purpose

The Shopfront app for Android and iOS (Flutter), where customers browse the
catalog, place orders and follow them. It talks to the services through clients
generated from their OpenAPI specs.

## Main documents and records

| Name (EN) | Naziv (HR) | Notes |
|---|---|---|
| Order | Narudžba | Read with `GET /api/v1/orders/{id}` for the order screen |
| Order line | Stavka narudžbe | Product name, quantity, line total |

## Processes

**Following my orders (LIN-GTB-3).** My orders lists the customer's orders as
cards with date, status chip and total, grouped by month under sticky headers
(FR-0001). Tapping a card opens the order screen, new in this issue, with the
order's lines (FR-0002). Cancelling moves from the list to the bottom of the
order screen, still behind the confirmation dialog (FR-0003). With no orders,
the list shows an empty state with the design's illustration (FR-0004). Spacing
and type follow the Figma design (FR-0005), whose frames haven't been taken in
yet (Q-0001). Dark mode is a later step (DEC-0005).

## Requirements

| ID | Title | Priority | Status |
|---|---|---|---|
| FR-0001 | My orders list as order cards grouped by month | must | ready |
| FR-0002 | Order screen with the order's lines | must | ready |
| FR-0003 | Cancel an order from the order screen | must | ready |
| FR-0004 | My orders empty state with the Figma illustration | should | clarifying |
| FR-0005 | My orders and order screen follow the Figma design's spacing and type | should | clarifying |

## Open questions

- Q-0001 — The Figma frames taken in with LIN-GTB-3 are of another product (maintainer; blocks FR-0004, FR-0005)
- Q-0002 — Which orders can a customer cancel? (client)
- Q-0003 — Should the order card still show the order number and item count? (client)
