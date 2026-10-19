# web — repository constitution (draft of its AGENTS.md)

Repository: `globulus/bf-web`. Package name `shopfront-web`, version 0.9.0
(`package.json:2-4`). The Shopfront web shop: product list, order form, order
list. React 19, TypeScript, Vite (`README.md:3`).

Onboarded by RUN-20261019T090336Z-onboarder from `main`. Not found: the code
repo's history (its `git log` was not readable in that run), so nothing here
is drawn from how the repo has been worked on.

## Toolchain

- Node: `.nvmrc` says `22`; `package.json:6-8` requires `node >=22.12`.
  The Docker build uses `node:22-alpine` (`Dockerfile:1`). CI uses
  `node-version-file: .nvmrc` (`.github/workflows/ci.yml:13-16`).
- Package manager: npm, with a committed `package-lock.json`.
- ES modules (`"type": "module"`, `package.json:5`).

## Commands

All from the repository root. Scripts are in `package.json:9-18`.

| Job | Command | Notes |
|---|---|---|
| Install | `npm ci` | As CI (`ci.yml:17`) and `Dockerfile:4`. |
| Build | `npm run build` | `tsc -b && vite build`: type-checks both TS projects (`tsconfig.app.json` for `src/`, `tsconfig.node.json` for configs and `e2e/`), then builds to `dist/`. |
| All unit tests | `npm test` | `vitest run`, jsdom, files `src/**/*.test.{ts,tsx}` (`vite.config.ts:22-27`). |
| Unit tests with JUnit (Globuli) | `npm test -- --reporter=default --reporter=junit --outputFile.junit=reports/junit.xml` | Arguments pass through to `vitest run`. `/reports/` is git-ignored for this (`.gitignore:26-27`). |
| One test file / one test | `npm test -- {file} -t "{title}"` | Not defined by the repo. Follows from `test` being `vitest run`, whose positional arguments filter files and `-t` filters test names. Globuli's config has no `test_one`. |
| Unit tests, watch | `npm run test:watch` | `vitest`. |
| Lint | `npm run lint` | `eslint .` with `eslint.config.js`. |
| Format | Not found: no Prettier or other formatter config, no format script. |
| Regenerate API types | `npm run api:generate` | `openapi-typescript` on `openapi/orders.json` and `openapi/catalog.json` into `src/api/generated/`. |
| Dev server | `npm run dev` | Vite on port 5173 (`vite.config.ts:15`). |
| E2E | `E2E_BASE_URL=http://localhost:7080 npm run e2e` | Playwright against a running dev-env stack, seeded; needs `npx playwright install chromium` once (`README.md:42-49`). Not run in this repo's CI. |

CI (`.github/workflows/ci.yml`) runs, in order: `npm ci`, `npm run
api:generate` then `git diff --exit-code -- src/api/generated`, `npm run
lint`, `npm test`, `npm run build`, and uploads `dist/` as artifact
`web-dist`. It runs on push to `main` and on every pull request.

Globuli's configured commands (install, build, test, lint) match the
repository. `test_one` is missing from the configuration; the command above
is the one to use.

## Layout

| Path | What |
|---|---|
| `src/main.tsx` | Entry point: React root, `BrowserRouter`, `StrictMode`. |
| `src/App.tsx` | Header, navigation and routes. |
| `src/pages/` | One component per screen, with its test next to it (`*.test.tsx`). |
| `src/lib/` | Plain functions (cart, money, customer ID), with tests next to them (`*.test.ts`). |
| `src/api/client.ts` | The two typed API clients, exported type aliases, `errorMessage`. |
| `src/api/generated/` | Generated types. Do not edit (see below). |
| `src/test/` | Vitest setup (`setup.ts`) and API mock helpers (`api-mock.ts`). |
| `src/index.css` | The only stylesheet; plain CSS with class names (`toolbar`, `error`, `notice`, `num`, `status-*`, `pager`). |
| `openapi/` | Copies of the backends' OpenAPI specs. |
| `e2e/` | Playwright tests. |
| `public/` | Static files (`favicon.svg`). |
| Config | `vite.config.ts` (also Vitest config), `playwright.config.ts`, `eslint.config.js`, `tsconfig*.json`, `Dockerfile`, `nginx.conf.template`. |
| Database migrations | None: the repo has no database. |

## Conventions

- **TypeScript is strict**, with `noUnusedLocals` and `noUnusedParameters`
  (`tsconfig.app.json:18-22`). `verbatimModuleSyntax` is on, so type-only
  imports use `type`: `import { catalogApi, errorMessage, type Product } from
  '../api/client'` (`src/pages/OrderFormPage.tsx:3`).
- **Components**: function components, one default export per page file,
  file named in PascalCase after the component (`src/pages/OrderListPage.tsx`).
  State with `useState`/`useEffect`; no state library.
- **Routing**: `react-router-dom` 7, routes declared in `src/App.tsx:18-24`.
  Navigation after an action passes state in the query string
  (`navigate('/orders?placed=…')`, `src/pages/OrderFormPage.tsx:55`).
- **API calls** go through `catalogApi` / `ordersApi` from `src/api/client.ts`
  (openapi-fetch), never `fetch` directly. Calls are written as
  `api.GET('/path', { params: { query: … } })` and the result is destructured
  into `{ data, error }`.
- **Types for API data** come from the generated files through the aliases in
  `src/api/client.ts:12-17` (`Product`, `Order`, `OrderSummary`, …). Add an
  alias there when a page needs a new schema type.
- **Error handling**: `if (error || !data)` → show
  `errorMessage(error, '<fallback>')` in a `<p className="error">`; a rejected
  promise (network failure) gets a fixed fallback string in `.catch`
  (`src/pages/ProductListPage.tsx:24-35`). `errorMessage` reads `detail`
  (orders-service, problem details) or `message` (catalog-service)
  (`src/api/client.ts:24-31`).
- **Effects that fetch** guard against stale responses with a `cancelled`
  flag set in the cleanup (`src/pages/OrderListPage.tsx:20-48`). Reloads are
  triggered by bumping a `reloadKey` state (`OrderListPage.tsx:18,59`).
- **Generated types are all optional** for most response fields, so code uses
  `?.` and `??` a lot and asserts ids with `!` (`OrderFormPage.tsx:95-96`).
  `README.md:39-40` says every field is optional; the catalog spec does mark
  some as required (`Money.amount`, `Money.currency`, `CategoryDto.name`,
  `CategoryDto.slug`, `openapi/catalog.json:439,529`), so this is true of
  most fields, not all.
- **Money**: amounts are JS numbers; shown with `formatMoney(amount,
  currency)` (`src/lib/money.ts`), which uses `Intl.NumberFormat('en-IE')` and
  shows `—` for a missing amount.
- **Cart**: a `Record<productId, quantity>`, changed only through the pure
  functions in `src/lib/cart.ts`; quantity clamped to 0..999, one order line
  per product.
- **Test hooks in markup**: `data-testid` on rows and totals (`product-row`,
  `order-row`, `order-total`); inputs get `aria-label`s that tests query by
  (`aria-label={`Quantity of ${p.name}`}`, `OrderFormPage.tsx:94`).
- **User-facing text** is hard-coded English in the JSX
  (`src/App.tsx:12-14`, all pages). There is no localization library or
  message catalog in the repo. How new text should be localized is for the
  project constitution to decide (see finding F-01 of the onboarding run).
- **Comments** are short and explain why, often naming the other service
  (`src/lib/cart.ts:19-22`, `src/pages/OrderFormPage.tsx:24`).
- **Lint**: ESLint flat config with `@eslint/js` recommended,
  `typescript-eslint` recommended, `react-hooks` recommended-latest and
  `react-refresh` vite (`eslint.config.js:8-29`). Ignores `dist`,
  `playwright-report`, `test-results`, `src/api/generated`.

### Tests

- **Unit/component tests**: Vitest 4 with jsdom, React Testing Library and
  `@testing-library/user-event`; `@testing-library/jest-dom` matchers are
  loaded in `src/test/setup.ts`, which also runs `cleanup()` after each test.
  Tests sit next to the code as `<name>.test.ts(x)`.
- Style: `describe('<Component or module>')`, `it('<behavior in plain
  words>')`, arrange / act / assert separated by blank lines
  (`src/lib/cart.test.ts`).
- **Mocking the API**: each page test mocks `../api/client` at the top of the
  file, keeping the real exports and replacing the clients
  (`src/pages/OrderFormPage.test.tsx:8-12`):

  ```ts
  vi.mock('../api/client', async (orig) => ({
    ...(await orig<typeof import('../api/client')>()),
    catalogApi: { GET: vi.fn() },
    ordersApi: { GET: vi.fn(), POST: vi.fn() },
  }))
  ```

  `vi.mock` is hoisted, so it can't move into a helper
  (`src/test/api-mock.ts:4-6`). The helpers `catalogGet`, `ordersGet`,
  `ordersPost`, `ok(data)`, `fail(status, error)`, `resetApiMocks()` and
  `sampleProducts` come from `src/test/api-mock.ts`. `beforeEach` calls
  `resetApiMocks()` and, where the customer ID matters, `localStorage.clear()`.
- Pages are rendered inside `MemoryRouter` (with `Routes` when the test
  checks navigation, `OrderFormPage.test.tsx:14-23`). Queries prefer roles,
  labels and text; `findBy…` waits for async data.
- **E2E**: Playwright, Chromium only, serial (`fullyParallel: false`),
  base URL from `E2E_BASE_URL` (default `http://localhost:7080`), one retry
  on CI (`playwright.config.ts`). Tests rely on dev-env seed data
  (`e2e/products.spec.ts:3`) and use a unique customer per run
  (`e2e/orders.spec.ts:3-4`).

## Don't touch

- `src/api/generated/catalog.ts`, `src/api/generated/orders.ts`: generated
  by `npm run api:generate` (file header: "auto-generated by
  openapi-typescript. Do not make direct changes"). CI fails if they differ
  from what `openapi/` generates (`ci.yml:18-21`). Change them only by
  regenerating.
- `openapi/orders.json`, `openapi/catalog.json`: copies of the backends'
  specs, taken from their CI artifacts `orders-openapi` and `catalog-openapi`
  (`README.md:32-33`). Don't edit by hand; replace with the backend's
  artifact, then regenerate. A change here is an API contract change.
- `package-lock.json`: changed only together with a planned dependency change.
- `.github/workflows/ci.yml`, `Dockerfile`, `nginx.conf.template`: CI and
  deployment. The nginx proxy paths must stay in step with
  `vite.config.ts:16-20` and `src/api/client.ts` (both send `/api/v1/orders`
  to orders-service and the rest of `/api` to catalog-service; the orders
  prefix must come first, `vite.config.ts:17`).
- `data-testid` values, `aria-label`s, the "Customer ID" label, and button
  names ("Place order", "Cancel", "Search"), the "Name or SKU" placeholder and
  the "Order #N placed." text: the e2e tests in `e2e/` find elements by them,
  and the e2e tests also run from the dev-env repo's nightly workflow
  (`ci.yml:30-31`).
- The localStorage key `shopfront.customerId` (`src/lib/customer.ts:2`):
  renaming it loses every user's remembered customer ID.

## Known problems

- No login: the customer ID is typed in and kept in localStorage; the comment
  refers to "TB-5" (`src/lib/customer.ts:1`). Any user can see and cancel any
  customer's orders by typing their ID (`src/pages/OrderListPage.tsx:70-80`).
- orders-service mishandles an order with the same product on two lines; the
  web avoids it by keying the cart by product (`src/lib/cart.ts:19-22`, which
  points to orders-service's KNOWN_ISSUES).
- Error bodies are not in either OpenAPI spec, so `errorMessage` takes
  `unknown` (`src/api/client.ts:19-23`).
- The category list request in `ProductListPage` has no error handling and no
  `.catch` (`src/pages/ProductListPage.tsx:14-16`); a failed request leaves
  the category filter empty without a message.
- The order form takes the display currency from the first product
  (`src/pages/OrderFormPage.tsx:39`) and lists at most 100 products
  (`pageSize: 100`, `OrderFormPage.tsx:19`; also `ProductListPage.tsx:22`).
  There is no paging on either product screen.
- E2E tests are not run in this repo's CI; they run nightly in the dev-env
  repo (`ci.yml:30-31`).
- No tests are marked skipped, flaky or failing, and no TODO or FIXME
  comments were found in the hand-written `.ts`/`.tsx` files (all read).
