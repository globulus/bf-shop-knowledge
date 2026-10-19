# Decisions

Project decisions, permanent IDs. A reversed decision is marked
`superseded by DEC-NNNN`, never deleted.

| ID | Date | Decision | Decided by | Source |
|---|---|---|---|---|
| DEC-0001 | 2026-10-19 | The My orders card in the app shows the order's date, status chip and total only; the order number and item count leave the card, and the order number is shown on the order screen. The client may still change this. | kb-synthesizer (default) | LIN-GTB-3 (Q-0003) |
| DEC-0002 | 2026-10-19 | My orders lists month groups and the orders within them newest first; an order belongs to the calendar month of its order date in the device's time zone; dates and month headers (month and year) follow the app's locale. The client may still change this. | kb-synthesizer (default) | LIN-GTB-3 |
| DEC-0003 | 2026-10-19 | The app's order screen shows the order's number, date, status chip and total above its lines; lines appear in the order the API returns them; all amounts are shown as `GET /api/v1/orders/{id}` returns them, the app computes none; it shows a loading indicator while loading and a localized error with a retry when the order can't be loaded. The client may still change this. | kb-synthesizer (default) | LIN-GTB-3 |
| DEC-0004 | 2026-10-19 | The Cancel button on the app's order screen is offered for placed orders only, not for shipped or cancelled ones. The client may still change this. | kb-synthesizer (default) | LIN-GTB-3 (Q-0002) |
| DEC-0005 | 2026-10-19 | Dark mode of the new My orders and order screens is left out of LIN-GTB-3's requirements, as the issue says it "can be a follow-up"; it becomes a requirement when an issue asks for it. The client may still change this. | kb-synthesizer (default) | LIN-GTB-3 |
| DEC-0006 | 2026-10-19 | The My orders empty state shows, besides the Figma illustration, a short localized message that there are no orders yet. The client may still change this. | kb-synthesizer (default) | LIN-GTB-3 |
