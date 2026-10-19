# mobile — repository constitution (draft)

*Written by the onboarder from the repository at its `main`
(`globulus/bf-mobile`). Draft of the repository's `AGENTS.md`; the maintainer
reviews it. Every statement cites the file it comes from. "Not found" means
the repository does not say.*

The repository is the Shopfront mobile app: a Flutter app for Android and iOS
that lists products, places an order and lists and cancels the customer's
orders (`README.md:1-4`). Package name `shopfront_mobile`, version `1.0.0+1`
(`pubspec.yaml:1,19`).

## Toolchain

| What | Value | Source |
|---|---|---|
| Flutter | 3.44 stable (README); CI pins `3.44.2` | `README.md:8`, `.github/workflows/ci.yml:9` |
| Dart SDK | `^3.12.2` | `pubspec.yaml:22` |
| Lockfile SDK floor | dart `>=3.12.2 <4.0.0`, flutter `>=3.18.0-18.0.pre.54` | `pubspec.lock:642-644` |
| Android | JDK 17, Android Gradle Plugin 9.0.1, Kotlin 2.3.20, Gradle 9.1.0 | `README.md:9`, `android/settings.gradle.kts:22-23`, `android/gradle/wrapper/gradle-wrapper.properties:5` |
| iOS | Xcode, CocoaPods; deployment target iOS 15.0 | `README.md:10`, `ios/Runner.xcodeproj/project.pbxproj:363` |

The Android build reads the Flutter SDK path from `android/local.properties`,
which is not committed (`android/settings.gradle.kts:2-9`,
`android/.gitignore:6`). The Flutter tool writes it.

## Commands

All from the repository root.

| Job | Command | Source |
|---|---|---|
| Install | `flutter pub get` | `README.md:13`, `ci.yml:21` |
| Lint | `flutter analyze` (CI); Globuli runs `flutter analyze --no-pub` | `ci.yml:26` |
| All tests | `flutter test` (CI); Globuli runs `flutter test --no-pub --file-reporter json:build/test-results.json` | `ci.yml:27`, `README.md:45` |
| One test | `flutter test --no-pub <file> --plain-name "<title>"` (Globuli's form; `--plain-name` is the Flutter test runner's filter) | not in the repo's docs or CI |
| Regenerate API clients | `./tool/generate_api.sh` | `README.md:35`, `tool/generate_api.sh` |
| Format | Not found as a project command. The generator script formats only `lib/api/generated` with `dart format` | `tool/generate_api.sh:9` |
| Build Android | `flutter build apk --debug` (CI); artifact `build/app/outputs/flutter-apk/app-debug.apk` | `ci.yml:44-48`, `README.md:53` |
| Build iOS | `flutter build ios --debug --no-codesign` (CI) | `ci.yml:61`, `README.md:55` |
| Run locally | `flutter run --dart-define=ORDERS_API_URL=... --dart-define=CATALOG_API_URL=...` | `README.md:14-15` |
| Integration test | `flutter test integration_test -d <device>`; add `--dart-define=E2E_BACKEND=true` for the real backend | `README.md:46-47`, `integration_test/order_flow_test.dart:1-5` |

Checks of Globuli's configured commands against the repository:

- `install: flutter pub get` — matches CI (`ci.yml:21`).
- `build: none` — CI builds a debug APK and an unsigned iOS app
  (`ci.yml:29-61`). Both need a platform toolchain (JDK, Android SDK, Xcode);
  neither runs Dart tests. Not a wrong setting; it means an agent's change to
  `android/` or `ios/` is only checked by CI.
- `test: flutter test --no-pub --file-reporter json:build/test-results.json`
  — runs the suites under `test/` (`flutter test` defaults to `test/`);
  `integration_test/` is not run, as in CI, because it needs a device
  (`integration_test/order_flow_test.dart:1-2`). `build/` is ignored by git
  (`.gitignore:33`), so the report file never ends up in a commit.
- `test_one: flutter test --no-pub {file} --plain-name {title}` — the test
  titles are plain strings (`test/widget/screens_test.dart:12`), so
  `--plain-name` finds them.
- `lint: flutter analyze --no-pub` — matches CI's `flutter analyze`
  (`ci.yml:26`), with the pub step skipped.
- Missing from Globuli's commands: CI's check that the committed generated
  client is up to date (`./tool/generate_api.sh` then
  `git diff --exit-code -- lib/api/generated`, `ci.yml:22-25`). An agent that
  changes `openapi/*.json` must run `./tool/generate_api.sh` and commit the
  result, or CI fails.

## Layout

| Path | What | Source |
|---|---|---|
| `lib/main.dart` | Entry point: builds `AppConfig` and the API repository, runs the app | `lib/main.dart:7-10` |
| `lib/app.dart` | `ShopfrontApp` (providers, theme) and `HomeShell` (three-tab navigation) | `lib/app.dart:10,33` |
| `lib/config.dart` | `AppConfig`: build-time settings read with `String.fromEnvironment` | `lib/config.dart:8-35` |
| `lib/data/` | `ShopRepository` interface and its HTTP implementation `ApiShopRepository` | `lib/data/shop_repository.dart:6`, `lib/data/api_shop_repository.dart:8` |
| `lib/api/generated/orders/`, `lib/api/generated/catalog/` | Generated API clients and DTOs, one folder per service, each with an `export.dart` | `swagger_parser.yaml:10-16` |
| `lib/state/cart.dart` | `Cart`, a `ChangeNotifier` holding the lines of the order being built | `lib/state/cart.dart:15` |
| `lib/screens/` | One file per tab: `products_screen.dart`, `create_order_screen.dart`, `my_orders_screen.dart` | `lib/app.dart:52-56` |
| `lib/util/money.dart` | `formatMoney` | `lib/util/money.dart:4` |
| `openapi/orders.json`, `openapi/catalog.json` | Copies of the services' OpenAPI specs, the generator's input | `swagger_parser.yaml:11,14` |
| `swagger_parser.yaml` | Generator configuration | file |
| `tool/generate_api.sh` | Runs the generator, `build_runner` and `dart format` on the output | file |
| `test/state/`, `test/util/` | Unit tests | `test/state/cart_test.dart`, `test/util/money_test.dart` |
| `test/widget/screens_test.dart` | Widget tests of the whole app against the fake repository | file |
| `test/support/fake_shop_repository.dart` | `FakeShopRepository`, the in-memory backend for widget and integration tests, plus the `product(...)` helper | `test/support/fake_shop_repository.dart:5,15` |
| `integration_test/order_flow_test.dart` | One device test of the browse–order–list flow | file |
| `android/`, `ios/` | Platform projects from the Flutter template | `.metadata:18-23` |
| `ios/RunnerTests/RunnerTests.swift` | An empty XCTest placeholder | file, lines 7-10 |
| `.github/workflows/ci.yml` | CI | file |
| `analysis_options.yaml` | Lint config: `package:flutter_lints/flutter.yaml`, no rules added or removed | `analysis_options.yaml:10,23-25` |

Not found: database migrations, assets, fonts, localization files
(`pubspec.yaml:70-99` has only the template's commented examples).

## Conventions

Only what the code shows.

- **Dependency injection with `provider`.** `ShopfrontApp` publishes the
  `ShopRepository` with `Provider.value` and the `Cart` with
  `ChangeNotifierProvider`; screens read them with `context.read<...>()` and
  rebuild with `context.watch<...>()` (`lib/app.dart:17-21`,
  `lib/screens/create_order_screen.dart:23-24,54`).
- **One repository interface over the generated clients.** Screens never
  call `OrdersClient` or `ProductsClient`; they call `ShopRepository`, whose
  HTTP implementation wraps the generated clients. The screens use the
  generated DTOs directly; there is no separate domain model
  (`lib/data/shop_repository.dart:4-5`).
- **HTTP with `dio` and `retrofit`.** One `Dio` per service with a 5 s connect
  and 10 s receive timeout; the base URL comes from `AppConfig`
  (`lib/data/api_shop_repository.dart:12-20`).
- **Generated code is committed and never edited.** Every generated file
  starts with `// GENERATED CODE - DO NOT MODIFY BY HAND` and disables lints
  for itself (`lib/api/generated/orders/orders/orders_client.dart:1-3`).
  `swagger_parser` makes the retrofit interfaces and `json_serializable`
  models; `build_runner` makes the `.g.dart` parts; unknown enum values map
  to `$unknown` (`swagger_parser.yaml:6-9`,
  `lib/api/generated/orders/models/status.dart:16-17`).
- **State.** `Cart extends ChangeNotifier`; mutators call `notifyListeners()`
  and the public list is unmodifiable (`lib/state/cart.dart:18,31-48`).
  Screens that load data hold a `Future` in `State`, render it with
  `FutureBuilder`, and reassign it in `_reload()` for retry and
  pull-to-refresh (`lib/screens/products_screen.dart:17-31,35,51`).
- **Error handling.** Loading errors show an inline message with a retry
  button (`lib/screens/products_screen.dart:41-46,94-113`;
  `lib/screens/my_orders_screen.dart:69-76`). Placing an order catches
  `DioException`, shows the response body's `detail` string if there is one,
  else the status code, and a generic message for any other exception
  (`lib/screens/create_order_screen.dart:37-50`). Cancel failures show a
  snackbar and reload (`lib/screens/my_orders_screen.dart:50-58`).
  Not found: a `detail` field in either OpenAPI spec; the catalog's
  `ErrorResponse` has `error`, `message`, `fieldErrors`, `timestamp`
  (`openapi/catalog.json:507-527`), and `openapi/orders.json` has no error
  schema.
- **Widget keys and tooltips for tests.** List tiles get `ValueKey`s such as
  `product-<id>`, `order-<id>`, `qty-<i>`; buttons get tooltips such as
  `Add <name>`, `More`, `Cancel order <id>`, and the tests find them by
  tooltip and by text (`lib/screens/products_screen.dart:73,77`,
  `lib/screens/my_orders_screen.dart:89,100`,
  `test/widget/screens_test.dart:35,58,100`).
- **User-facing text is a string literal in the widget**, in English
  (`lib/app.dart:65,73,77`, `lib/screens/products_screen.dart:42,49`,
  `lib/screens/my_orders_screen.dart:114-125`). Not found: a localization
  mechanism (`flutter_localizations`, `intl`, an `l10n.yaml` or `.arb`
  files). Globuli's common constitution (§6) requires user-facing text to go
  through the project's localization mechanism; this repository has none,
  so the maintainer has to say how agents add text here.
- **Money.** Amounts are `num` from the DTOs; `formatMoney` prints two
  decimals with a symbol for EUR, USD and GBP and the code otherwise
  (`lib/util/money.dart:1-10`). Totals are summed in `num` in the app
  (`lib/state/cart.dart:12,24`).
- **Tests.** `flutter_test`; unit tests use `test(...)`, widget tests
  `testWidgets(...)` with `pumpWidget(ShopfrontApp(repository: fake))` and
  `pumpAndSettle()`; `setUp` makes a fresh `FakeShopRepository` per test
  (`test/widget/screens_test.dart:8-14`). No mocking library: the fake is a
  hand-written `ShopRepository` with switches such as `failProducts` and a
  record of `placed` lines (`test/support/fake_shop_repository.dart:25-27`).
  The integration test reuses the same fake from `test/support/` and switches
  to the real `ApiShopRepository` with `--dart-define=E2E_BACKEND=true`
  (`integration_test/order_flow_test.dart:14-24`).
- **Naming.** Files `snake_case.dart`; one public widget per screen file
  with private helpers prefixed `_` (`_ProductTile`, `_ErrorView`,
  `_ProductsScreenState`); generated DTOs keep the spec's names
  (`OrderSummaryDto`, `ProductResponse`).
- **Lints.** The default `flutter_lints` set, nothing customized
  (`analysis_options.yaml`).

## Don't touch

| What | Why | Source |
|---|---|---|
| `lib/api/generated/**` | Generated from `openapi/*.json`; CI regenerates and fails on any diff. Change the spec and run `./tool/generate_api.sh` instead. | `README.md:27-28`, `ci.yml:22-25` |
| `openapi/orders.json`, `openapi/catalog.json` | Copies of orders-service's and catalog-service's published specs. They change only when the service changes, by copying the new artifact in (`README.md:30-34`); the knowledge repo's overview says an API change ships with its clients in one release (`prd/overview.md:46-47`). | |
| `*.g.dart` next to generated models and clients | `build_runner` output; regenerated by the script. | `tool/generate_api.sh:8` |
| `.metadata` | Flutter tool bookkeeping: "should not be manually edited". | `.metadata:4` |
| `pubspec.lock` | Resolved versions; changes only with a dependency change, which needs an approved plan (common constitution §3). | |
| `android/`, `ios/` template files (`MainActivity.kt`, `AppDelegate.swift`, `SceneDelegate.swift`, storyboards, icons, `Runner.xcodeproj`) | Flutter template; `.metadata` tracks them for `flutter migrate`. `ios/Runner.xcodeproj/project.pbxproj` is listed as unmanaged. | `.metadata:31-33` |
| `ios/Flutter/Debug.xcconfig` | Pins simulator builds to arm64 to work around a Flutter 3.44 / Xcode 27 lipo problem. | `ios/Flutter/Debug.xcconfig:3-6` |
| `android/app/src/debug/AndroidManifest.xml` | Enables cleartext HTTP in debug builds so the app can reach the local stack. | file, lines 7-8 |
| `ios/Runner/Info.plist` `NSAllowsLocalNetworking` | Same purpose on iOS. | `ios/Runner/Info.plist:52-56` |
| `.github/workflows/ci.yml` | CI; a change needs an approved plan (common constitution §3). | |

## Configuration

Build-time only, with `--dart-define` (`lib/config.dart:1-35`):

| Define | Default | Meaning |
|---|---|---|
| `ORDERS_API_URL` | `http://localhost:7081` | orders-service base URL |
| `CATALOG_API_URL` | `http://localhost:7082` | catalog-service base URL |
| `CUSTOMER_ID` | `demo-customer` | the customer every install acts as; no login yet |
| `E2E_BACKEND` | `false` | integration test only: use the real services (`integration_test/order_flow_test.dart:16`) |

From the Android emulator the host is `10.0.2.2`; from the iOS simulator
`localhost` (`README.md:18-19`). Not found: any runtime configuration, `.env`
file or remote config.

## Known problems

- **Skipped test.** `shipped orders have no cancel button` is `skip: true`
  because the fake cannot ship orders (`test/widget/screens_test.dart:81-89`).
- **Duplicate product lines.** Adding the same product twice makes two cart
  lines; the app relies on orders-service to merge them, and the README says
  the backend currently does not add the quantities up (`README.md:60-62`,
  `lib/state/cart.dart:29-30`; the test
  `adding the same product twice keeps two lines` pins the app side,
  `test/state/cart_test.dart:26-33`).
- **My orders shows the first 50 only.** The list asks for page 0, size 50,
  and ignores `totalPages` (`lib/data/api_shop_repository.dart:41-45`,
  `README.md:63`). Products likewise: page 0, size 50
  (`lib/data/api_shop_repository.dart:28-31`).
- **No login.** The customer id is a build-time constant, `TODO(auth)`
  (`lib/config.dart:24-29`).
- **Release build signed with the debug key**, `TODO` in
  `android/app/build.gradle.kts:30-33`; application id still the template
  `TODO` (`android/app/build.gradle.kts:18-19`).
- **Three generated enums for one status**: `Status`, `OrderResponseStatus`
  and `OrderSummaryDtoStatus`, all `PLACED | SHIPPED | CANCELLED`
  (`lib/api/generated/orders/models/`). The screens use
  `OrderSummaryDtoStatus` (`lib/screens/my_orders_screen.dart:87`).
- **Generated names from the catalog spec**: `getApiProductsId` (the spec's
  `get` is a keyword) and `create1` (the spec's `create_1`)
  (`lib/api/generated/catalog/products/products_client.dart:36-40`,
  `lib/api/generated/catalog/categories/categories_client.dart:20`).
- **`ios/RunnerTests/RunnerTests.swift`** is the template's empty test.
- **Nullable DTO fields with `!`.** `product.id!` and `order.id!` assume the
  service always sends an id (`lib/state/cart.dart:51`,
  `lib/screens/my_orders_screen.dart:51`).
- Not found: a flaky-test marker, an issue tracker reference or a
  `CHANGELOG`.

## History

Not found: the run could not read the repository's git history (the shell
refused `git` against the code repository's worktree, and the knowledge
repo's own history is a single import commit). The maintainer's `AGENTS.md`
can add how the repository is worked on.
