---
source: LIN-GTB-2
url: https://linear.app/doodleag/issue/GTB-2
title: Let customers add a delivery note to their order
state: Ready
labels: [Feature, orders-service, web, Mobile]
updated: 2026-10-18T17:15:09.666Z
---

Support gets a lot of emails like "please leave it with the neighbour at no. 12" after
people have ordered. We want a free-text note on the order that the customer can fill
in when they place it, and that the warehouse sees.

What we need:

* An optional note when placing an order, in both the web order form and the mobile
  app's order screen. Max 280 characters (the warehouse label printer cuts off after
  that).
* The note is stored with the order and comes back when you fetch the order.
* The order list on web shows a small note icon for orders that have one. Mobile can
  show it on the order in My orders.
* The note can't be changed after the order is placed. If they want to change it they
  can email support like today.

Out of scope: showing the note in the confirmation email (notifications aren't wired up
yet anyway).

Backend first, then web and mobile can go in parallel once the spec is updated.
