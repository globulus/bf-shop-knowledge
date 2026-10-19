# Web

## Purpose

The web shop (React), where customers browse the catalog, place orders and see
their orders.

## Main documents and records

| Name (EN) | Naziv (HR) | Notes |
|---|---|---|
| Order form | Obrazac narudžbe | Where the customer places an order; now with an optional delivery note (FR-0002). |
| Order list | Popis narudžbi | The customer's orders; shows a note icon on orders with a delivery note (FR-0002). |

## Processes

**Delivery note.** The order form has an optional delivery note of at most 280
characters, sent with the order to orders-service (FR-0001). The order list
shows a small note icon for orders that have a note, without the note's text
(DEC-0005). The note can't be edited after the order is placed (FR-0002).

## Requirements

| ID | Title | Priority | Status |
|---|---|---|---|
| FR-0002 | Delivery note in the web shop | must | ready |

## Open questions

- None.
