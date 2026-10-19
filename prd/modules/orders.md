# Orders

## Purpose

Placing, listing and cancelling orders (orders-service). The web shop and the
app place and fetch orders through its API.

## Main documents and records

| Name (EN) | Naziv (HR) | Notes |
|---|---|---|
| Order | Narudžba | Placed by a customer in the web shop or the app. |
| Delivery note | Napomena za dostavu | Optional free text on an order, at most 280 characters, written when the order is placed and never changed afterwards (FR-0001). |

## Processes

**Placing an order with a delivery note.** Support gets many emails after
ordering with delivery instructions ("please leave it with the neighbour at
no. 12", LIN-GTB-2). A customer can now add an optional delivery note when
placing the order; orders-service stores it with the order and returns it when
the order is fetched (FR-0001). The note is limited to 280 characters because
the warehouse label printer cuts off after that. It can't be changed after the
order is placed; a customer who wants to change it emails support as before.
The warehouse sees the note through the order as orders-service returns it
(DEC-0007). The API change comes first; the web shop (FR-0002) and the app
(FR-0003) follow once the OpenAPI spec is updated.

## Requirements

| ID | Title | Priority | Status |
|---|---|---|---|
| FR-0001 | Delivery note on an order (orders-service) | must | ready |

## Open questions

- Q-0001 — How does the warehouse see an order's delivery note?
- Q-0002 — How should the 280-character limit of a delivery note count?
