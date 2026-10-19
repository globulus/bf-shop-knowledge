# mobile — place in the system

*Written by the onboarder from the repository at its `main`.*

## What it is

The customer-facing Android and iOS app of Shopfront (`README.md:1-4`;
`prd/overview.md:39`). It is a client only: it has no server side, no
database and no queue. Everything it shows comes from two HTTP services.

## What it serves

Three screens behind a bottom navigation bar (`lib/app.dart:59-79`):

| Tab | Screen | Backend calls |
|---|---|---|
| Products | product list with add-to-order | `GET /api/products?activeOnly=true&page=0&pageSize=50` on catalog-service (`lib/data/api_shop_repository.dart:28-31`) |
| Order | the lines being ordered, place order | `POST /api/v1/orders` on orders-service with `customerId` and `lines[{productId, quantity}]` (`lib/data/api_shop_repository.dart:34-38`) |
| My orders | the customer's orders, cancel | `GET /api/v1/orders?customerId=...&page=0&size=50`; `POST /api/v1/orders/{id}/cancel` with `{reason: "Cancelled in the app"}` (`lib/data/api_shop_repository.dart:41-53`) |

The app exposes no API of its own, no deep links and no push notifications.
Not found: analytics, crash reporting, or any other outbound integration.

## What it calls

| Service | Base URL define | Default | Spec copy | Spec version |
|---|---|---|---|---|
| orders-service | `ORDERS_API_URL` | `http://localhost:7081` | `openapi/orders.json` | Orders API 1.4.0, OpenAPI 3.1.0 (`openapi/orders.json:2-7`) |
| catalog-service | `CATALOG_API_URL` | `http://localhost:7082` | `openapi/catalog.json` | Catalog API 2.1, OpenAPI 3.0.1 (`openapi/catalog.json:2-7`) |

The defaults are where `dev-env` runs the services (`README.md:19-20`). Both
calls are plain HTTP with `dio`, 5 s connect and 10 s receive timeouts, no
authentication header, no retry (`lib/data/api_shop_repository.dart:12-20`).
Debug builds allow cleartext traffic on Android and local networking on iOS
(`android/app/src/debug/AndroidManifest.xml:7-8`,
`ios/Runner/Info.plist:52-56`).

### Identity

There is no login. Every install is the customer `demo-customer`, overridable
with `--dart-define=CUSTOMER_ID=...` (`lib/config.dart:24-29`,
`README.md:21-23`). The id is sent as `customerId` when placing and listing
orders; the services do not authenticate it (nothing in either spec asks for
a credential).

## Shared contracts

- **The services' OpenAPI specs.** The services publish them; this repo and
  the web repo keep copies in `openapi/` and generate their clients
  (`prd/overview.md:46-47`; `README.md:27-36`). A backend change reaches the
  app by copying the new spec into `openapi/`, running
  `./tool/generate_api.sh` and fixing what no longer compiles
  (`README.md:30-36`). CI fails when the committed client differs from what
  the committed spec generates (`.github/workflows/ci.yml:22-25`).
- **What the app depends on in those specs** (anything else can change
  without touching the app):
  - catalog: `GET /api/products` with `q`, `activeOnly`, `page`, `pageSize`;
    `ProductPage.items[]` of `ProductResponse{id, sku, name, price{amount,
    currency}}` (`openapi/catalog.json:99-179,567-617`).
  - orders: `POST /api/v1/orders` with `CreateOrderRequest{customerId,
    lines[{productId, quantity}]}` (max 50 lines, quantity 1–999) returning
    `OrderResponse{id, status, totalAmount}`; `GET /api/v1/orders` with
    `customerId`, `page`, `size` returning
    `PageResponseOrderSummaryDto.content[]` of `OrderSummaryDto{id,
    itemCount, status, totalAmount, currency}`;
    `POST /api/v1/orders/{id}/cancel` with `CancelOrderRequest{reason}`
    (`openapi/orders.json:13-158,189-351`).
  - order status values `PLACED`, `SHIPPED`, `CANCELLED`; the app shows a
    cancel button only for `PLACED` (`lib/screens/my_orders_screen.dart:87`)
    and labels anything else `Unknown`
    (`lib/screens/my_orders_screen.dart:114-125`). A new value does not
    crash the app: the generated enums map it to `$unknown`
    (`lib/api/generated/orders/models/order_summary_dto_status.dart:16-22`).
  - error body: the order screen reads a `detail` string from a failed
    `POST /api/v1/orders` (`lib/screens/create_order_screen.dart:46-50`).
    Not found in `openapi/orders.json`: an error schema with `detail`.
- **Behaviour the app leaves to orders-service:** merging duplicate product
  lines into one (`lib/state/cart.dart:29-30`) and pricing each line at the
  catalog's current price (`openapi/orders.json:66`).
- Not found: a shared package, generated client published for others, or
  events the app produces or consumes.

## Configuration

Build-time only, via `--dart-define` (`lib/config.dart`). See the
constitution's Configuration table for the four defines. There is no runtime
configuration and no environment file.

## Build and deployment

- CI (`.github/workflows/ci.yml`) runs on push to `main` and on pull
  requests: analyze and test, then a debug APK uploaded as the artifact
  `app-debug-apk`, and an unsigned iOS debug build on macOS.
- Not found: a release pipeline, store upload, signing configuration
  (`android/app/build.gradle.kts:30-33` signs release with the debug key;
  `android/.gitignore:12-14` keeps `key.properties` and keystores out of git),
  versioning scheme beyond `version: 1.0.0+1` in `pubspec.yaml:19`, or tags.
- Not found: a Dockerfile or any server deployment; the app is installed on
  devices only.

## Local stack

`dev-env` (Docker Compose for the whole product, `prd/overview.md:43-45`)
runs orders-service on 7081 and catalog-service on 7082 (`README.md:19-20`).
From the Android emulator the app reaches them at `10.0.2.2`, from the iOS
simulator at `localhost` (`README.md:14-19`). The integration test can run
against that stack with `--dart-define=E2E_BACKEND=true`
(`integration_test/order_flow_test.dart:4-5,16`).
