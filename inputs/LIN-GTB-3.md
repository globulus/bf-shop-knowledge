---
source: LIN-GTB-3
url: https://linear.app/doodleag/issue/GTB-3
title: New "My orders" screen in the app
state: Ready
labels: [design, Mobile]
updated: 2026-10-18T17:15:09.951Z
designs:
  - https://www.figma.com/design/V6zOfKbNI3wPTVJBs02Jwd/Mobile-App?node-id=4687-11685
---

Design is done: [https://www.figma.com/design/V6zOfKbNI3wPTVJBs02Jwd/Mobile-App?node-id=4687-11685](<https://www.figma.com/design/V6zOfKbNI3wPTVJBs02Jwd/Mobile-App?node-id=4687-11685>)

The current My orders list is just order number, item count and total. In the new
design:

* Each order is a card with the date, the status as a coloured chip (placed = amber,
  shipped = green, cancelled = grey) and the total.
* Orders are grouped by month with a sticky month header.
* Tapping a card opens the order with its lines (product name, quantity, line total).
  We don't have this screen at all today.
* Cancel moves from the list into the order screen, at the bottom, as a text button.
  Keep the confirmation dialog.
* Empty state uses the illustration from the Figma file (it's exported in the
  "Assets" page).

Check the Figma for spacing and type, I won't repeat it here. Dark mode is in the file
too but can be a follow-up.

The order detail needs `GET /api/v1/orders/{id}`, which the app's generated client
already has.

<!-- Globuli (D-407): the frames of the issue's Figma links, exported as images. Not part of the issue's text. -->
## Designs

- [i](attachments/LIN-GTB-3/6186-2215-i.png), Figma node 6186:2215
- [Local components](attachments/LIN-GTB-3/6186-2217-local-components.png), Figma node 6186:2217
- [i](attachments/LIN-GTB-3/7531-7432-i.png), Figma node 7531:7432
- [Active Collab Card](attachments/LIN-GTB-3/7892-44286-active-collab-card.png), Figma node 7892:44286
- [Active Collab Card](attachments/LIN-GTB-3/7998-9098-active-collab-card.png), Figma node 7998:9098
- [Virtual rooms](attachments/LIN-GTB-3/10018-39149-virtual-rooms.png), Figma node 10018:39149
- [Virtual rooms](attachments/LIN-GTB-3/10018-37155-virtual-rooms.png), Figma node 10018:37155
- [Virtual rooms](attachments/LIN-GTB-3/9309-38789-virtual-rooms.png), Figma node 9309:38789
- Only the first 8 frames are exported (`intake.figma.max_frames`); the links lead to more.
