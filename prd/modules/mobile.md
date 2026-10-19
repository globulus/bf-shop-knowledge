# Mobile

## Purpose

The app for Android and iOS (Flutter), where customers browse the catalog,
place orders and see their orders in My orders.

## Main documents and records

| Name (EN) | Naziv (HR) | Notes |
|---|---|---|
| Order screen | Zaslon narudžbe | Where the customer places an order; now with an optional delivery note (FR-0003). |
| My orders | Moje narudžbe | The customer's orders; shows an order's delivery note (FR-0003). |

## Processes

**Delivery note.** The order screen has an optional delivery note of at most
280 characters, sent with the order to orders-service (FR-0001). My orders
shows the note's full text on the order that has one (DEC-0006). The note can't
be edited after the order is placed (FR-0003).

## Requirements

| ID | Title | Priority | Status |
|---|---|---|---|
| FR-0003 | Delivery note in the mobile app | must | ready |

## Open questions

- None.
