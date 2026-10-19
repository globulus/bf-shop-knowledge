# web — place in the system

## What it serves

The customer-facing web shop, Shopfront (`README.md:1-3`). Screens:

| Route | Screen |
|---|---|
| `/` | redirects to `/products` |
| `/products` | product list with search and category filter |
| `/orders/new` | order form |
| `/orders` | order list with status filter, paging and cancel |

It serves no API of its own. There is no login (`src/lib/customer.ts:1`); the
customer is identified by a typed-in customer ID kept in localStorage.

## What it calls

Both backends are reached from the browser on the page's own origin; a proxy
splits them by path (`src/api/client.ts:5-7`):

| Path prefix | Service | Spec in this repo |
|---|---|---|
| `/api/v1/orders` | orders-service | `openapi/orders.json` ("Orders API" 1.4.0, OpenAPI 3.1.0) |
| the rest of `/api` | catalog-service | `openapi/catalog.json` ("Catalog API" 2.1, OpenAPI 3.0.1) |

Endpoints the code uses:

- catalog-service: `GET /api/categories`, `GET /api/products` (`q`,
  `categoryId`, `pageSize`).
- orders-service: `GET /api/v1/orders` (`customerId`, `status`, `page`,
  `size`), `POST /api/v1/orders`, `POST /api/v1/orders/{id}/cancel`.

The specs also describe endpoints the web doesn't call (product and category
create/update, prices, `GET /api/v1/orders/{id}`, `POST
/api/v1/orders/{id}/ship`).

Error bodies differ: orders-service answers with problem details
(`detail`), catalog-service with `{error, message}`
(`src/api/client.ts:19-23`; catalog's `ErrorResponse` schema,
`openapi/catalog.json:507-527`). orders-service's error body is not in its
spec.

No database, queue or other service is called.

## Contracts shared with other repositories

- `openapi/orders.json` and `openapi/catalog.json` are copies of the specs
  that orders-service and catalog-service publish as CI artifacts
  `orders-openapi` and `catalog-openapi` (`README.md:32-33`). The types in
  `src/api/generated/` are generated from them and CI checks they match
  (`.github/workflows/ci.yml:18-21`). Not found: how the copies are kept up
  to date (no workflow in this repo fetches them).
- The dev-env repository runs this repo's e2e tests nightly
  (`ci.yml:30-31`) and provides the stack and seed data they need
  (`e2e/products.spec.ts:3`, `README.md:44`).
- CI publishes the built app as the artifact `web-dist` (`ci.yml:25-28`).
  Not found: what consumes it.

## Configuration

| Setting | Where | Default | Used for |
|---|---|---|---|
| `ORDERS_API_URL` | `.env.local` for `npm run dev` (`vite.config.ts:8-9`); container env (`Dockerfile:9`) | dev `http://localhost:7081`; Docker `http://orders-service:8080` | orders-service proxy target |
| `CATALOG_API_URL` | same (`vite.config.ts:10`, `Dockerfile:10`) | dev `http://localhost:7082`; Docker `http://catalog-service:8080` | catalog-service proxy target |
| `VITE_API_BASE_URL` | build-time env (`src/api/client.ts:7`) | `''` (same origin) | base URL of both API clients |
| `E2E_BASE_URL` | env for `npm run e2e` (`playwright.config.ts:5`) | `http://localhost:7080` | where the e2e tests point |

`.env` and `*.local` files are git-ignored (`.gitignore:9-10`). No secrets
are in the repository.

## Running and deployment

- Dev: `npm run dev`, Vite on port 5173, proxying `/api/v1/orders` and `/api`
  to the URLs above (`vite.config.ts:14-21`). dev-env puts the services on
  ports 7081 and 7082 (`README.md:14-17`), and the whole stack's web entry on
  7080 (`README.md:48`).
- Image: `Dockerfile` builds with `node:22-alpine` (`npm ci`, `npm run
  build`) and serves `dist/` from `nginx:1.29-alpine` on port 80. The nginx
  image renders `nginx.conf.template` with `envsubst` at start
  (`nginx.conf.template:1-2`): `/api/v1/orders` → `ORDERS_API_URL`, `/api/` →
  `CATALOG_API_URL`, `/assets/` cached for a year as immutable, everything
  else falls back to `/index.html`.
- Not found: where the image is built, pushed or deployed. CI builds `dist/`
  but no image, and has no deploy step.
