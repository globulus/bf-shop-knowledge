# Decisions

Project decisions, permanent IDs. A reversed decision is marked
`superseded by DEC-NNNN`, never deleted.

| ID | Date | Decision | Decided by | Source |
|---|---|---|---|---|
| DEC-0001 | 2026-10-19 | The delivery note's 280-character limit counts Unicode characters (code points) of the note after trimming; a line break counts as one character. orders-service rejects an order whose note is longer with a validation error (HTTP 400) and creates no order; it never truncates the note. The client may still change this. | kb-synthesizer (default) | LIN-GTB-2 (Q-0002) |
| DEC-0002 | 2026-10-19 | orders-service trims leading and trailing whitespace from the delivery note; a note that is empty or only whitespace is stored as no note. Line breaks inside the note are kept. An order without a note returns `deliveryNote` as `null`, and orders placed before the note existed have no note. The client may still change this. | kb-synthesizer (default) | LIN-GTB-2 (Q-0002) |
| DEC-0003 | 2026-10-19 | The note field in the web order form and the app's order screen accepts at most 280 characters (further typing or pasted text beyond 280 is not taken in); no character counter is required. The client may still change this. | kb-synthesizer (default) | LIN-GTB-2 |
| DEC-0004 | 2026-10-19 | "Can't be changed after the order is placed" means: orders-service offers no way to set or change the note of an existing order, every other order operation (such as cancelling) leaves the note as it was, and neither the web shop nor the app offers a way to edit it. The client may still change this. | kb-synthesizer (default) | LIN-GTB-2 |
| DEC-0005 | 2026-10-19 | The web order list shows, for an order with a note, a note icon with an accessible, localized label saying the order has a delivery note, and nothing for an order without one; the list does not show the note's text. The client may still change this. | kb-synthesizer (default) | LIN-GTB-2 |
| DEC-0006 | 2026-10-19 | The app shows the full text of an order's delivery note on that order in My orders, and nothing for an order without one. The client may still change this. | kb-synthesizer (default) | LIN-GTB-2 |
| DEC-0007 | 2026-10-19 | "The warehouse sees" the note through the order as orders-service returns it; no warehouse screen, label printing or other warehouse integration is in scope for LIN-GTB-2. The client may still change this. | kb-synthesizer (default) | LIN-GTB-2 (Q-0001) |
