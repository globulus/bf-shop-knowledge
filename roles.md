# Roles

User roles in the product. Requirements refer to roles by slug.

| Slug | Role | Description |
|---|---|---|
| customer | Customer | Buys from the shop on the web or in the mobile app. There is no login yet: the app acts as one fixed customer (`prd/overview.md`, Users). |

## Permissions by module

| Module | Role | May |
|---|---|---|
| mobile | customer | List their orders, open an order with its lines, cancel a placed order after confirming (FR-0001, FR-0002, FR-0003; LIN-GTB-3, DEC-0004) |
