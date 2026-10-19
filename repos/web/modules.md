# web — modules

A single-page React app. Its parts are folders under `src/`, not packages.

## App shell

What: mounts the app and routes between the three screens. `/` redirects to
`/products`; unknown paths show "Page not found." (`src/App.tsx:18-24`).

Main files: `index.html`, `src/main.tsx`, `src/App.tsx`, `src/index.css`.

Depends on: `react`, `react-dom`, `react-router-dom`; the three pages.

## Products page (`/products`)

What: lists products with name, SKU, category and price. Text search
("Name or SKU", sent as `q` on submit) and a category filter (sent as
`categoryId` on change). Requests up to 100 products; no paging. A product
without a price shows `—`.

Main files: `src/pages/ProductListPage.tsx`, `src/pages/ProductListPage.test.tsx`,
`e2e/products.spec.ts`.

Depends on: `catalogApi` (`GET /api/categories`, `GET /api/products`),
`formatMoney`.

## Order form (`/orders/new`)

What: a quantity field per orderable product (products without a price are
hidden, `OrderFormPage.tsx:24-25`), a customer ID field, a running total, and
"Place order". Submits `POST /api/v1/orders` with `{ customerId, lines }`,
remembers the customer ID, then goes to `/orders?placed=<id>`. Shows the
server's error text when the order is refused.

Main files: `src/pages/OrderFormPage.tsx`, `src/pages/OrderFormPage.test.tsx`,
`e2e/orders.spec.ts`.

Depends on: `catalogApi` (`GET /api/products`), `ordersApi`
(`POST /api/v1/orders`), `src/lib/cart.ts`, `src/lib/customer.ts`,
`src/lib/money.ts`.

## Order list (`/orders`)

What: a customer's orders, newest first per the API, 10 per page, filtered by
customer ID and status (`PLACED`, `SHIPPED`, `CANCELLED`). A placed order can
be cancelled after a `window.confirm`; the list then reloads. Shows "Order #N
placed." when it arrives with `?placed=N`.

Main files: `src/pages/OrderListPage.tsx`, `src/pages/OrderListPage.test.tsx`,
`e2e/orders.spec.ts`.

Depends on: `ordersApi` (`GET /api/v1/orders`,
`POST /api/v1/orders/{id}/cancel` with a fixed reason "Cancelled from the web
shop"), `src/lib/customer.ts`, `src/lib/money.ts`.

## API client (`src/api/`)

What: two `openapi-fetch` clients, `catalogApi` and `ordersApi`, both on the
page's origin (`VITE_API_BASE_URL`, default `''`); type aliases for the
schemas the pages use; `errorMessage` to turn either backend's error body
into text.

Main files: `src/api/client.ts`; generated `src/api/generated/catalog.ts` and
`src/api/generated/orders.ts` (from `openapi/*.json` by `npm run
api:generate`).

Depends on: `openapi-fetch`; `openapi-typescript` (dev, generation only).

## Lib (`src/lib/`)

What: pure helpers with unit tests.

- `cart.ts`: `Cart` (product ID → quantity), `setQuantity` (clamps 0..999, 0
  removes), `toOrderLines` (one line per product), `cartTotal`.
- `money.ts`: `formatMoney(amount, currency)`, `en-IE` currency format,
  cached formatters, `—` for no amount.
- `customer.ts`: `getCustomerId` / `setCustomerId` in localStorage key
  `shopfront.customerId`, default `web-customer`; swallows storage errors.

Main files: the three above and `cart.test.ts`, `money.test.ts`.

Depends on: nothing outside the browser.

## Test support (`src/test/`)

What: Vitest setup (jest-dom matchers, `cleanup` after each test) and API
mock helpers (`catalogGet`, `ordersGet`, `ordersPost`, `ok`, `fail`,
`resetApiMocks`, `sampleProducts`).

Main files: `src/test/setup.ts`, `src/test/api-mock.ts`.

Depends on: `vitest`, `@testing-library/react`, `@testing-library/jest-dom`.

## E2E (`e2e/`)

What: Playwright tests of the products and orders screens against a running,
seeded stack.

Main files: `e2e/products.spec.ts`, `e2e/orders.spec.ts`,
`playwright.config.ts`.

Depends on: `@playwright/test` 1.63.0; the dev-env stack and its seed data.

## Packaging (`Dockerfile`, `nginx.conf.template`)

What: builds `dist/` with Node 22 and serves it from `nginx:1.29-alpine` on
port 80, proxying the APIs and falling back to `index.html` for client
routes.

Depends on: orders-service and catalog-service at `ORDERS_API_URL` and
`CATALOG_API_URL`.
