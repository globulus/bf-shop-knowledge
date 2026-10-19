# Roles

User roles in the product. Requirements refer to roles by slug.

| Slug | Role | Description |
|---|---|---|
| customer | Customer | Browses the catalog and places orders in the web shop or the app. There is no login yet (prd/overview.md). |

## Permissions by module

| Module | Role | May |
|---|---|---|
| orders | customer | Place an order with an optional delivery note; the note can't be changed afterwards (FR-0001, LIN-GTB-2). |
| web | customer | Write a delivery note in the order form; see which orders in the order list have one (FR-0002). |
| mobile | customer | Write a delivery note on the order screen; see it on the order in My orders (FR-0003). |
