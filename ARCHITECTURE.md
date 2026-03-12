# Architecture Specification — flutter-foundation

> **Authoritative reference for structure, dependencies, conventions, and development workflow.**  
> This document is intended for both human developers and AI agents implementing tasks in this repository.

---

## Table of Contents

1. [Overview](#overview)
2. [Platform Targets](#platform-targets)
3. [Architectural Principles](#architectural-principles)
4. [Repository Layout](#repository-layout)
5. [Layer Definitions](#layer-definitions)
   - [Core Layer](#core-layer)
   - [Features Layer](#features-layer)
   - [Shared Layer](#shared-layer)
6. [Dependency Stack](#dependency-stack)
7. [Application Shell](#application-shell)
8. [Feature Module Registration](#feature-module-registration)
9. [State Management](#state-management)
10. [Navigation](#navigation)
11. [Networking](#networking)
12. [Dependency Injection](#dependency-injection)
13. [Persistence](#persistence)
14. [Configuration](#configuration)
15. [Logging](#logging)
16. [Responsive Layout](#responsive-layout)
17. [Dashboard System](#dashboard-system)
18. [Theming and UI Utilities](#theming-and-ui-utilities)
19. [CI/CD Integration](#cicd-integration)
20. [AI Workflow Integration](#ai-workflow-integration)
21. [Code Generation](#code-generation)
22. [Developer Workflow](#developer-workflow)
23. [Conventions](#conventions)

---

## Overview

`flutter-foundation` is the reusable Flutter project template for the Ego Hygiene ecosystem. It provides a clean, scalable, and cross-platform base that all product applications can build upon. The foundation defines:

- A consistent folder and module structure based on Clean Architecture
- A curated dependency stack for state management, routing, networking, persistence, and UI
- An application shell ready to host feature modules
- A reusable dashboard layout system
- Full integration with the repository's tooling (FVM, commitlint, semantic-release, GitHub Actions)
- Hooks for the AI workflow system at `ai/factory/`

---

## Platform Targets

The foundation must compile and run correctly on all six Flutter platforms:

| Platform | Target                       |
|----------|------------------------------|
| Android  | API 21+ (minSdkVersion 21)   |
| iOS      | iOS 13+                      |
| Web      | Modern browsers (Chromium, Firefox, Safari) |
| Linux    | x86_64, Ubuntu 20.04+        |
| Windows  | Windows 10+ (x86_64)         |
| macOS    | macOS 12+ (Apple Silicon + Intel) |

Platform-specific code lives under `lib/core/platform/` and is accessed via abstract interfaces. No platform-specific imports should appear outside of that directory or inside platform-specific `_web.dart` / `_io.dart` conditional files.

---

## Architectural Principles

The project follows **Clean Architecture** with three concentric layers: Core, Features, and Shared.

```
┌───────────────────────────────────┐
│            Features               │  ← business logic, UI, feature-scoped state
├───────────────────────────────────┤
│              Shared               │  ← reusable widgets, themes, utilities
├───────────────────────────────────┤
│               Core                │  ← infrastructure: DI, routing, networking, config
└───────────────────────────────────┘
```

Key rules:
- **Dependencies flow inward.** Features depend on Core and Shared; Core has no dependency on Features.
- **No business logic in widgets.** Views are thin; logic belongs in providers and services.
- **Feature isolation.** Each feature module is self-contained — its own providers, services, models, repositories, and views.
- **Testability first.** All services and repositories expose abstract interfaces so they can be faked or mocked in tests.

---

## Repository Layout

```
flutter-foundation/
├── ai/                          # Git submodule: egohygiene/ai (workflow & templates)
│   └── factory/
│       ├── workflow/            # AI agent workflow rules
│       └── templates/           # Issue and PR templates for AI agents
├── lib/
│   ├── main.dart                # Application entrypoint
│   ├── app.dart                 # Root widget (ProviderScope + MaterialApp.router)
│   ├── core/                    # Infrastructure layer
│   │   ├── config/              # Environment configuration (flutter_dotenv)
│   │   ├── di/                  # Dependency injection (get_it + injectable)
│   │   ├── logging/             # Logging setup (logger)
│   │   ├── networking/          # HTTP client (dio + pretty_dio_logger)
│   │   ├── platform/            # Platform-specific abstractions
│   │   ├── routing/             # Navigation (go_router)
│   │   └── theme/               # App-wide theme tokens (referenced from shared)
│   ├── features/                # Feature modules
│   │   └── <feature_name>/
│   │       ├── data/            # Repositories, data sources, DTOs
│   │       ├── domain/          # Entities, abstract repository interfaces, use-cases
│   │       ├── presentation/    # Providers, view models, pages, widgets
│   │       └── <feature>.dart   # Public barrel export for the feature
│   └── shared/                  # Reusable cross-feature code
│       ├── widgets/             # Generic UI components
│       ├── layouts/             # App shell, dashboard shell, scaffold wrappers
│       ├── theme/               # ThemeData, color scheme, text styles
│       └── utils/               # Date helpers (intl, timeago), string utils, etc.
├── test/                        # Unit and widget tests mirroring lib/ structure
├── integration_test/            # Integration tests
├── assets/                      # Static assets (images, fonts, .env files)
│   └── .env                     # Default environment file (excluded from VCS if sensitive)
├── .fvmrc                       # Pinned Flutter SDK version (FVM)
├── pubspec.yaml                 # Flutter dependencies
├── build_runner.yaml            # build_runner configuration
├── commitlint.config.js         # Commit message validation
├── .releaserc.json              # semantic-release configuration
└── package.json                 # Node.js tooling (commitlint, husky, semantic-release)
```

---

## Layer Definitions

### Core Layer

`lib/core/` contains infrastructure concerns that the rest of the application depends on. It has no knowledge of individual features.

#### `core/config/`
- Loads environment variables from `assets/.env` using `flutter_dotenv`.
- Exposes a `AppConfig` class (or a typed Riverpod provider) that surfaces values such as `apiBaseUrl`, `logLevel`, and feature flags.
- `.env` files for each environment (`assets/.env`, `assets/.env.staging`, `assets/.env.production`) should be loaded at startup based on a compile-time flag or build flavor.

#### `core/di/`
- Service locator configured with `get_it` and `injectable`.
- A single `configureDependencies()` function is called before `runApp()`.
- Annotate service implementations with `@injectable`, `@singleton`, or `@lazySingleton` as appropriate.
- The generated `injection.config.dart` file is committed (or regenerated via `build_runner`).

#### `core/logging/`
- A singleton `AppLogger` wraps the `logger` package.
- Log levels are configured from `AppConfig` (e.g., `verbose` in debug, `warning` in production).
- All services obtain a logger via DI rather than instantiating `Logger` directly.

#### `core/networking/`
- A pre-configured `Dio` instance is registered in the DI container.
- Interceptors applied by default:
  - `PrettyDioLogger` (debug builds only)
  - An `AuthInterceptor` that attaches bearer tokens (feature-specific tokens are injected)
  - A `RetryInterceptor` for idempotent requests
- Base URL is read from `AppConfig.apiBaseUrl`.

#### `core/routing/`
- `GoRouter` configuration defining all top-level routes.
- Routes are composed from each feature's route definitions via a `featureRoutes` list that features contribute.
- A `routerProvider` Riverpod provider exposes the router instance.
- The shell route wraps authenticated pages with the `AppShell` widget (see [Application Shell](#application-shell)).

#### `core/platform/`
- Abstract interfaces for platform capabilities (e.g., `IFilePicker`, `IShareService`).
- Concrete implementations registered per-platform via `injectable` environment annotations or conditional imports.

---

### Features Layer

`lib/features/` is where product functionality lives. Each feature is a self-contained vertical slice.

#### Feature Module Structure

```
features/
└── dashboard/
    ├── data/
    │   ├── sources/             # Remote and local data sources
    │   ├── models/              # DTOs annotated with @freezed + @JsonSerializable
    │   └── repositories/        # Concrete repository implementations
    ├── domain/
    │   ├── entities/            # Pure domain objects (Freezed)
    │   ├── repositories/        # Abstract repository interfaces
    │   └── use_cases/           # Single-responsibility use-case classes
    ├── presentation/
    │   ├── providers/           # Riverpod providers / AsyncNotifiers
    │   ├── pages/               # Full-screen route targets
    │   └── widgets/             # Feature-scoped widgets
    └── dashboard.dart           # Barrel export
```

#### Guidelines
- DTOs (data transfer objects) live in `data/models/` and are never exposed beyond the feature's `data/` boundary; they are mapped to domain entities before leaving the repository layer.
- Domain entities are plain Dart classes or Freezed value objects — they have no Flutter or package dependencies.
- Use-cases are optional for simple CRUD features but are mandatory when business rules span multiple repositories.
- Providers in `presentation/providers/` use `@riverpod` annotation (code-generated via `riverpod_annotation`).

---

### Shared Layer

`lib/shared/` contains code that is reused across multiple features but carries no feature-specific business logic.

#### `shared/widgets/`
- Generic, stateless or minimally stateful UI components: `AppButton`, `AppCard`, `AppTextField`, `LoadingIndicator`, `EmptyState`, etc.
- Components accept only primitive data and callbacks — no providers or services are consumed directly.

#### `shared/layouts/`
- `AppShell` — the authenticated application shell with navigation rail / drawer.
- `DashboardShell` — the responsive grid container used by dashboard pages (see [Dashboard System](#dashboard-system)).
- `UnauthenticatedShell` — layout wrapper for auth flows.

#### `shared/theme/`
- `AppTheme.light()` and `AppTheme.dark()` return `ThemeData` objects.
- Design tokens (colors, spacing, typography, border radii) are defined as constants in `AppColors`, `AppSpacing`, and `AppTextStyles`.

#### `shared/utils/`
- `DateUtils` — formatting helpers using `intl` and relative-time strings using `timeago`.
- `StringUtils`, `ValidationUtils`, and other pure utility functions.

---

## Dependency Stack

All packages below target the latest stable version compatible with Flutter 3.27.x. Versions are pinned in `pubspec.yaml`.

### State Management

| Package | Purpose |
|---|---|
| `flutter_riverpod` | Core Riverpod runtime for Flutter |
| `hooks_riverpod` | `HookConsumerWidget` for combining Riverpod + Flutter Hooks |
| `flutter_hooks` | React-style hooks for Flutter widgets |
| `riverpod_annotation` | `@riverpod` annotation for code generation |

All providers are generated via `riverpod_annotation` + `build_runner`. This produces type-safe providers in `*.g.dart` files. Prefer `AsyncNotifierProvider` for async state and `NotifierProvider` for synchronous state. Use `@riverpod` on functions for simple read-only providers.

### Navigation

| Package | Purpose |
|---|---|
| `go_router` | Declarative, URL-based routing |

- All routes are defined as `GoRoute` or `ShellRoute` entries.
- Route paths are defined as string constants in a `AppRoutes` class.
- Deep linking is enabled by default for Web, Android, and iOS.
- Authentication redirects are handled by a `GoRouter` redirect callback that reads auth state from a Riverpod provider.

### Networking

| Package | Purpose |
|---|---|
| `dio` | HTTP client with interceptor support |
| `pretty_dio_logger` | Human-readable request/response logging (debug only) |

### Modeling and Serialization

| Package | Purpose |
|---|---|
| `freezed` | Immutable value objects, union types, `copyWith` |
| `json_serializable` | `fromJson` / `toJson` code generation |
| `build_runner` | Build system for code generation |

All DTOs and domain entities use `@freezed`. JSON serialization is added to DTOs with `@JsonSerializable`. Generated files (`*.freezed.dart`, `*.g.dart`) are committed to source control for reproducibility.

### Dependency Injection

| Package | Purpose |
|---|---|
| `get_it` | Service locator |
| `injectable` | Annotation-driven `get_it` registration |

A single `@module` class registers third-party dependencies (e.g., `Dio`, `SharedPreferences`). Feature services use `@injectable` or `@singleton`.

### Persistence

| Package | Purpose |
|---|---|
| `drift` | Reactive SQLite ORM |
| `sqlite3_flutter_libs` | Native SQLite3 libraries for all platforms |
| `shared_preferences` | Key-value store for lightweight preferences |

- Drift databases are defined in `core/` (schema) and accessed from feature repositories.
- `shared_preferences` is used only for simple settings (theme preference, first-launch flag, etc.).

### Configuration

| Package | Purpose |
|---|---|
| `flutter_dotenv` | Load `.env` files at runtime |

### Logging

| Package | Purpose |
|---|---|
| `logger` | Structured, colourized console logging |

### Responsive Layout

| Package | Purpose |
|---|---|
| `responsive_framework` | Breakpoint-based responsive scaling |
| `adaptive_breakpoints` | Material adaptive breakpoint utilities |

`ResponsiveBreakpoints.builder` is applied at the root `MaterialApp.router` level, wrapping the entire widget tree. Breakpoint definitions:

| Name | Min Width |
|---|---|
| `MOBILE` | 0 |
| `TABLET` | 600 |
| `DESKTOP` | 1024 |
| `4K` | 2560 |

### Dashboard Components

| Package | Purpose |
|---|---|
| `fl_chart` | Line, bar, pie, and scatter charts |
| `data_table_2` | Improved Flutter `DataTable` with sorting and column sizing |
| `pluto_grid` | Feature-rich data grid for tabular data |
| `flutter_staggered_grid_view` | Masonry / staggered grid layouts |

### UI Utilities

| Package | Purpose |
|---|---|
| `flutter_animate` | Declarative animation chaining |
| `gap` | `Gap` widget for spacing in `Row`/`Column` |
| `lucide_icons_flutter` | Lucide icon set |

### Date Utilities

| Package | Purpose |
|---|---|
| `intl` | Locale-aware date and number formatting |
| `timeago` | Human-readable relative time strings |

### Optional Visualization

| Package | Purpose |
|---|---|
| `graphview` | Graph and tree visualizations |
| `flutter_map` | Interactive maps (OpenStreetMap / custom tile providers) |

These packages are included in `pubspec.yaml` but their integrations are gated behind feature flags or kept in dedicated feature modules to avoid increasing binary size when unused.

---

## Application Shell

### Entrypoint (`lib/main.dart`)

```
main()
  └── 1. Load .env              (flutter_dotenv)
  └── 2. configureDependencies() (injectable / get_it)
  └── 3. runApp(ProviderScope(child: App()))
```

### Root Widget (`lib/app.dart`)

- Wrapped in `ProviderScope` (Riverpod root).
- Uses `MaterialApp.router` with the `GoRouter` instance from `routerProvider`.
- `ResponsiveBreakpoints.builder` wraps `MaterialApp.router` for global responsive scaling.
- `ThemeData` is driven by a `themeProvider` that reads the user's saved preference (`shared_preferences`).

### App Shell (`shared/layouts/app_shell.dart`)

The `AppShell` is a `ShellRoute` child that renders the persistent navigation chrome:

- **Mobile:** `NavigationBar` (bottom).
- **Tablet / Desktop:** `NavigationRail` (left side) or `NavigationDrawer` (collapsible).

Adaptive behavior is determined by `ResponsiveBreakpoints.of(context)`. The shell hosts a `body` slot that `GoRouter` fills with the current page.

---

## Feature Module Registration

This section defines the canonical pattern for how a feature module registers itself with the core application. Every feature contributes four integration points: routes, Riverpod providers, dependency injection registrations, and a public barrel export.

### Route Registration

Each feature defines its routes as a typed `List<RouteBase>` constant and exposes that list from its barrel file. The core router then merges the lists from every registered feature.

**Step 1 — Define route constants.**

Add the feature's path strings to `core/routing/app_routes.dart`:

```dart
abstract final class AppRoutes {
  static const login     = '/login';
  static const dashboard = '/dashboard';
  // Add each new feature's paths here
}
```

**Step 2 — Declare the feature's routes list.**

Create `features/<feature>/routing/<feature>_routes.dart` and define a top-level `List<RouteBase>`:

```dart
// features/dashboard/routing/dashboard_routes.dart
import 'package:go_router/go_router.dart';
import 'package:flutter_foundation/core/routing/app_routes.dart';
import '../presentation/pages/dashboard_page.dart';
import '../presentation/pages/dashboard_detail_page.dart';

final List<RouteBase> dashboardRoutes = [
  GoRoute(
    path: AppRoutes.dashboard,
    builder: (context, state) => const DashboardPage(),
    routes: [
      GoRoute(
        path: ':widgetId',
        builder: (context, state) => DashboardDetailPage(
          widgetId: state.pathParameters['widgetId']!,
        ),
      ),
    ],
  ),
];
```

**Step 3 — Register the routes in the core router.**

Merge the feature's route list into the `_featureRoutes` collection in `core/routing/router.dart`:

```dart
// core/routing/router.dart
import 'package:flutter_foundation/features/dashboard/dashboard.dart';

final _featureRoutes = [
  ...dashboardRoutes,
  // Add each new feature's routes here
];

final GoRouter _router = GoRouter(
  routes: [
    GoRoute(path: '/', redirect: (_, __) => AppRoutes.dashboard),
    ShellRoute(
      builder: (context, state, child) => AppShell(child: child),
      routes: _featureRoutes,
    ),
    GoRoute(
      path: AppRoutes.login,
      builder: (context, state) => const LoginPage(),
    ),
  ],
  redirect: _authRedirect,
);
```

Every feature's routes are isolated inside `_featureRoutes`. The only file that imports feature route lists is `core/routing/router.dart`.

---

### Provider Registration

Feature providers are declared in `features/<feature>/presentation/providers/` using the `@riverpod` annotation. Code generation (via `riverpod_annotation` + `build_runner`) makes them available application-wide — no central registration file is required.

Providers that need to be consumed outside the feature boundary must be re-exported from the feature's barrel file:

```dart
// features/dashboard/dashboard.dart
export 'presentation/providers/dashboard_providers.dart';
```

Cross-feature provider access uses the barrel import:

```dart
import 'package:flutter_foundation/features/dashboard/dashboard.dart';

// Inside a Consumer, HookConsumerWidget, or another provider:
final metrics = ref.watch(dashboardMetricsProvider);
```

Global providers (auth state, theme mode, remote config) live in `core/` or `shared/` and are **not** owned by any single feature.

---

### Dependency Injection Registration

Feature repositories and services register themselves with the `injectable` / `get_it` container by annotating their implementation classes. Use `@LazySingleton(as: IAbstractInterface)` for repositories (created on first access, alive for the app lifetime) and `@injectable` for transient (factory) services that require a new instance per call.

```dart
// features/dashboard/data/repositories/dashboard_repository_impl.dart
import 'package:injectable/injectable.dart';
import '../../domain/repositories/i_dashboard_repository.dart';

@LazySingleton(as: IDashboardRepository)
class DashboardRepositoryImpl implements IDashboardRepository {
  DashboardRepositoryImpl(this._apiClient, this._db);

  final DashboardApiClient _apiClient;
  final AppDatabase _db;

  // ...
}
```

After adding or modifying `@injectable` annotations, regenerate `injection.config.dart`:

```sh
fvm flutter pub run build_runner build --delete-conflicting-outputs
```

Feature Riverpod providers bridge the DI container and the reactive layer:

```dart
// features/dashboard/presentation/providers/dashboard_repository_provider.dart
import 'package:riverpod_annotation/riverpod_annotation.dart';
import 'package:get_it/get_it.dart';
import '../../domain/repositories/i_dashboard_repository.dart';

part 'dashboard_repository_provider.g.dart';

@riverpod
IDashboardRepository dashboardRepository(DashboardRepositoryRef ref) =>
    GetIt.I<IDashboardRepository>();
```

This pattern keeps infrastructure wiring (`get_it`) out of widgets entirely — widgets only interact with Riverpod providers.

---

### Feature Barrel Export

Every feature exposes a single public barrel file at `lib/features/<feature>/<feature>.dart`. This file re-exports only the parts of the feature that are intended to cross its boundary:

```dart
// features/dashboard/dashboard.dart
library dashboard;

// Routes
export 'routing/dashboard_routes.dart';

// Public providers (consumed by other features or the shell)
export 'presentation/providers/dashboard_providers.dart';

// Domain entities (when consumed by other features)
export 'domain/entities/dashboard_metric.dart';
```

The following are **not** re-exported from the barrel file:

- Data transfer objects (`data/models/`)
- Concrete repository implementations (`data/repositories/`)
- Internal data sources (`data/sources/`)
- Feature-private widgets (`presentation/widgets/`)

Code outside the feature imports only via the barrel:

```dart
import 'package:flutter_foundation/features/dashboard/dashboard.dart';
```

Deep imports into a feature's internal directories (e.g., `features/dashboard/data/models/dashboard_dto.dart`) are forbidden across feature boundaries.

---

### Registration Checklist (Adding a New Feature)

Complete the following steps in order when integrating a new feature module:

| # | Step | Location |
|---|------|----------|
| 1 | Add route path constants | `core/routing/app_routes.dart` |
| 2 | Create the feature's routes list | `features/<feature>/routing/<feature>_routes.dart` |
| 3 | Spread the routes list into `_featureRoutes` | `core/routing/router.dart` |
| 4 | Annotate repository implementations with `@LazySingleton(as: IRepo)` | `features/<feature>/data/repositories/` |
| 5 | Run `build_runner` to regenerate `injection.config.dart` | project root |
| 6 | Create a repository bridge provider using `GetIt.I<IRepo>()` | `features/<feature>/presentation/providers/` |
| 7 | Re-export routes, public providers, and domain entities | `features/<feature>/<feature>.dart` |
| 8 | Import the barrel file in `core/routing/router.dart` for route access | `core/routing/router.dart` |

---

## State Management

All application state is managed through Riverpod providers. Patterns:

| Pattern | When to use |
|---|---|
| `@riverpod` function | Simple, derived, read-only state |
| `NotifierProvider` | Synchronous mutable state |
| `AsyncNotifierProvider` | Async state (network, database) |
| `StreamProvider` | Real-time or stream-based state |
| `StateProvider` | Simple toggles and primitive values (use sparingly) |

Avoid global mutable state outside of Riverpod. Never use `setState` beyond leaf widget local state.

Providers are scoped to features in `features/<feature>/presentation/providers/`. Global providers (auth state, theme, config) live in `core/` or `shared/`.

### Provider Naming Convention

- `fooProvider` — exposes a value or service
- `fooNotifierProvider` — exposes a `Notifier` or `AsyncNotifier`
- Suffix `Family` for parameterized providers: `fooProvider(id)`

---

## Navigation

Routes are defined in `core/routing/router.dart`. Each feature contributes its routes via a list merged at the router level.

```
/                         → redirect to /dashboard (authenticated) or /login
/login                    → LoginPage
/dashboard                → DashboardPage        (ShellRoute: AppShell)
/dashboard/:widgetId      → DashboardDetailPage  (ShellRoute: AppShell)
```

Route names are string constants on `AppRoutes`:

```dart
abstract class AppRoutes {
  static const login    = '/login';
  static const dashboard = '/dashboard';
}
```

Navigation is performed via `context.go()`, `context.push()`, and `context.pop()` (GoRouter extensions). Never use `Navigator.of(context)` directly.

---

## Networking

The `Dio` instance is created in `core/networking/dio_client.dart` and registered as a singleton in DI. It is configured with:

1. `BaseOptions` — `baseUrl` from `AppConfig`, timeouts (connect: 10 s, receive: 30 s).
2. `PrettyDioLogger` — enabled only when `kDebugMode` is `true`.
3. `AuthInterceptor` — injects `Authorization: Bearer <token>` from the auth state provider.
4. Error handling — a `DioException` is caught and mapped to a domain `Failure` type in the repository layer.

Feature-specific API clients extend or compose the base `Dio` instance but do not create new instances.

---

## Dependency Injection

`injectable` generates `configureDependencies()` in `core/di/injection.config.dart`. Call this function before `runApp()`.

```dart
// lib/main.dart
await configureDependencies();
runApp(ProviderScope(child: App()));
```

Annotations used:

| Annotation | Lifecycle |
|---|---|
| `@singleton` | One instance for the app lifetime |
| `@lazySingleton` | Created on first access |
| `@injectable` | New instance per request |
| `@module` | Registers third-party or async dependencies |

The DI container and Riverpod coexist: DI manages infrastructure services (repositories, API clients, database); Riverpod manages reactive UI state on top of those services. Feature providers receive services via `ref.watch` on a provider that wraps `getIt<MyService>()`.

---

## Persistence

### Drift (SQLite)

- The `AppDatabase` class is defined in `core/di/` (or a dedicated `core/database/` sub-folder).
- Table definitions are Dart classes annotated with `@DataClassName`.
- DAOs are injected into feature repositories.
- Migrations use Drift's schema versioning support (`MigrationStrategy`).

### Shared Preferences

A `SharedPreferences` instance is registered in DI via an `@module` that exposes the async singleton:

```dart
@module
abstract class SharedPreferencesModule {
  @preResolve
  Future<SharedPreferences> get prefs => SharedPreferences.getInstance();
}
```

---

## Configuration

`flutter_dotenv` loads a `.env` file from `assets/.env` at startup. The file is declared in `pubspec.yaml` under `flutter.assets`. A typed `AppConfig` wrapper reads individual keys with fallback defaults:

```dart
class AppConfig {
  static String get apiBaseUrl =>
      dotenv.env['API_BASE_URL'] ?? 'https://api.example.com';
  static bool get debugMode =>
      dotenv.env['DEBUG'] == 'true';
}
```

Secrets must never be committed to version control. Provide an `assets/.env.example` with placeholder values. In CI, inject real values as environment secrets before the build step.

---

## Logging

`AppLogger` is a singleton wrapper around `logger`'s `Logger`. Severity levels map to:

| Build mode | Log level |
|---|---|
| Debug | `Level.verbose` |
| Profile | `Level.info` |
| Release | `Level.warning` |

Usage:

```dart
AppLogger.instance.d('Widget built');   // debug
AppLogger.instance.i('User logged in'); // info
AppLogger.instance.e('API error', err, stackTrace); // error
```

---

## Responsive Layout

`responsive_framework` is applied at the `MaterialApp.router` `builder` slot:

```dart
builder: (context, child) => ResponsiveBreakpoints.builder(
  child: child!,
  breakpoints: [
    const Breakpoint(start: 0,    end: 599,  name: MOBILE),
    const Breakpoint(start: 600,  end: 1023, name: TABLET),
    const Breakpoint(start: 1024, end: 2559, name: DESKTOP),
    const Breakpoint(start: 2560, end: double.infinity, name: '4K'),
  ],
),
```

Within widgets, use `ResponsiveBreakpoints.of(context).isMobile`, `.isTablet`, `.isDesktop` to drive adaptive layouts. `adaptive_breakpoints` is used for Material 3 canonical layouts (list-detail, supporting pane).

---

## Dashboard System

The dashboard is a first-class feature (`lib/features/dashboard/`) and also provides reusable layout components in `lib/shared/layouts/`.

### DashboardShell

`DashboardShell` is a `Widget` that wraps page content in a `StaggeredGrid` (via `flutter_staggered_grid_view`). It handles:

- Responsive column counts (1 on mobile, 2 on tablet, 3–4 on desktop).
- `DashboardCard` wrapper with consistent padding, elevation, and border radius.
- Resize-aware re-layout when window dimensions change.

### Metric Cards

`MetricCard` is a shared widget that displays a single KPI:

```
┌──────────────────────────┐
│  Icon    Label            │
│          Value (large)    │
│          Trend indicator  │
└──────────────────────────┘
```

### Chart Components

Charts are built with `fl_chart` and wrapped in feature-agnostic containers:

| Component | Chart type |
|---|---|
| `LineChartCard` | Time-series line chart |
| `BarChartCard` | Categorical bar chart |
| `PieChartCard` | Distribution pie/donut chart |

All chart wrappers accept a `title` and generic `data` list; rendering details are encapsulated inside the wrapper.

### Data Table Components

- `AppDataTable` — wraps `data_table_2` for simple tabular data with sorting.
- `AppDataGrid` — wraps `pluto_grid` for complex grids with inline editing, filtering, and column reordering.

### Dashboard Layout Example

```
┌─────────────────────────────────────────────────────┐
│  NavigationRail  │  DashboardShell                  │
│                  │  ┌──────────┐ ┌──────────┐       │
│                  │  │ Metric 1 │ │ Metric 2 │       │
│                  │  └──────────┘ └──────────┘       │
│                  │  ┌──────────────────────────────┐ │
│                  │  │      LineChartCard            │ │
│                  │  └──────────────────────────────┘ │
│                  │  ┌──────────────────────────────┐ │
│                  │  │        AppDataTable           │ │
│                  │  └──────────────────────────────┘ │
└─────────────────────────────────────────────────────┘
```

---

## Theming and UI Utilities

### Theming

`AppTheme.light()` and `AppTheme.dark()` produce `ThemeData` objects rooted in Material 3 (`useMaterial3: true`). Design tokens are centralised:

| Constant class | Contents |
|---|---|
| `AppColors` | Seed colors, semantic colors (error, success, warning) |
| `AppSpacing` | Spacing scale (4, 8, 12, 16, 24, 32, 48 dp) |
| `AppTextStyles` | `TextStyle` definitions aligned to the type scale |

The active theme is toggled by a `themeProvider` (`StateNotifierProvider<ThemeModeNotifier, ThemeMode>`) and persisted via `shared_preferences`.

### `flutter_animate`

Declarative animation chaining is applied to widgets with the `.animate()` extension. Common presets (fade-in, slide-up, shimmer loading) are defined as extension methods in `shared/utils/animate_utils.dart` to ensure consistency.

### `gap`

Use `Gap(AppSpacing.md)` inside `Row` and `Column` instead of `SizedBox` for whitespace. This avoids magic numbers and keeps spacing consistent with the design token scale.

### `lucide_icons_flutter`

All icons in the application use the Lucide icon set (`LucideIcons.*`). Do not use `Icons.*` from the Flutter SDK material library to maintain visual consistency.

---

## CI/CD Integration

### GitHub Actions Workflows

| Workflow | Trigger | Purpose |
|---|---|---|
| `flutter.yml` | push/PR to `main` | Install Flutter (via FVM version), run `flutter --version`, and will run `flutter test` and `flutter build` as they are added |
| `commitlint.yml` | push/PR to `main` | Validate all commit messages against `commitlint.config.js` |
| `release.yml` | push to `main` | Run `semantic-release` to tag, publish, and update `CHANGELOG.md` |
| `copilot-setup-steps.yml` | push/PR to `main`, `workflow_dispatch` | Bootstrap the full CI environment for GitHub Copilot agents; validates repo configuration, documentation, toolchain, and AI submodule structure |

All workflows check out the repository with `submodules: recursive` and use the `AI_DEPLOY_KEY` deploy key to access the private `egohygiene/ai` submodule.

### Shared Composite Action

`.github/actions/setup-environment/` is a composite action used by `flutter.yml`, `commitlint.yml`, and `release.yml`. It handles:

- Node.js setup (`actions/setup-node`) with version sourced from `.nvmrc`.
- pnpm installation via Corepack (`corepack enable pnpm && corepack prepare pnpm@latest --activate`), avoiding any `node20` JS action dependency.
- Node dependency installation (frozen lockfile when `pnpm-lock.yaml` exists).
- Flutter SDK setup via `subosito/flutter-action` (version pinned to `.fvmrc`).
- `git safe.directory` configuration for the Flutter SDK cache path.
- Flutter dependency installation via `flutter pub get`.

The action auto-detects which toolchains are needed (`hashFiles` guards on `package.json` and `pubspec.yaml`), making it reusable across Node-only, Flutter-only, and hybrid repositories.

### Node.js 24 Compatibility

GitHub Actions JavaScript runners are transitioning from Node.js 20 to Node.js 24. All workflows in this repository are fully compatible with Node.js 24:

- `actions/checkout@v6` and `actions/setup-node@v6` declare `using: node24` natively.
- `subosito/flutter-action@v2` is a composite (bash-only) action — no Node.js runtime dependency.
- `wagoid/commitlint-github-action@v6` runs in Docker — no Node.js runtime dependency.
- pnpm is installed via **Corepack** (`corepack enable pnpm && corepack prepare pnpm@latest --activate`), a Node.js built-in tool, eliminating the need for the `pnpm/action-setup` JavaScript action entirely.

All workflows also set `FORCE_JAVASCRIPT_ACTIONS_TO_NODE24: true` at the workflow `env:` level as a defensive measure, ensuring any transitively-invoked JavaScript action (e.g., `actions/cache` inside `subosito/flutter-action`) also runs under Node.js 24.

### Flutter Build Artifacts

As the project grows, `flutter.yml` should be extended with platform-specific build jobs:

```yaml
# Suggested extensions (to be added during implementation)
- run: flutter test
- run: flutter build apk --release
- run: flutter build ios --release --no-codesign
- run: flutter build web --release
- run: flutter build linux --release
- run: flutter build windows --release
- run: flutter build macos --release
```

Build artifacts (`.apk`, `.ipa`, web bundle) should be uploaded via `actions/upload-artifact` and attached to GitHub Releases via `.releaserc.json`'s `assets` list.

### Versioning and Changelog

- `semantic-release` reads commits since the last Git tag on `main`.
- Version bumps follow the rules in [CONTRIBUTING.md](CONTRIBUTING.md).
- `CHANGELOG.md` is automatically updated and committed back to `main` with `chore(release): v<version> [skip ci]`.
- The `pubspec.yaml` `version` field should be updated by an `@semantic-release/exec` plugin step in `.releaserc.json` during implementation.

### Commit Validation

`commitlint` (configured in `commitlint.config.js`) enforces Conventional Commits on every local `git commit` (via Husky) and again in CI. Only the types `feat`, `fix`, `docs`, `refactor`, `chore`, `style`, `test`, `perf`, `ci`, `build`, and `revert` are accepted.

---

## AI Workflow Integration

The `ai/` directory is a Git submodule pointing to [`egohygiene/ai`](https://github.com/egohygiene/ai). It is initialised with:

```sh
git submodule update --init --recursive
```

### Directory Structure

```
ai/
└── factory/
    ├── workflow/        # Agent workflow rules — read by AI agents before executing tasks
    └── templates/       # Issue and PR body templates for AI-generated tasks
```

### Rules for AI Agents

All AI agents operating on this repository **must**:

1. Read and follow the rules defined in `ai/factory/workflow/` before beginning any task.
2. Follow the issue contract when creating or closing issues.
3. Use only the commit types defined in `commitlint.config.js`.
4. Not introduce commits that violate commitlint rules.
5. Ensure commits are compatible with `semantic-release` version inference.
6. Not implement application code during architecture-only tasks (such as generating this document).

Refer to `ai/factory/templates/` for the canonical format of implementation issues, PR descriptions, and task handoffs between agents.

---

## Code Generation

All generated files (`*.g.dart`, `*.freezed.dart`, `injection.config.dart`) are produced by `build_runner`.

### Running the Generator

```sh
# One-time build
fvm flutter pub run build_runner build --delete-conflicting-outputs

# Watch mode (development)
fvm flutter pub run build_runner watch --delete-conflicting-outputs
```

A `build_runner.yaml` at the project root configures output directories and filters to keep generated files adjacent to their sources.

Generated files **are committed** to version control. This ensures that CI can run `flutter test` and `flutter build` without a generation step and that reviewers can inspect generated output in PRs.

---

## Developer Workflow

### Initial Setup

```sh
# 1. Clone with submodules
git clone --recurse-submodules <repo-url>
cd flutter-foundation

# 2. Install Flutter SDK (pinned via FVM)
fvm install

# 3. Install Node.js dev tooling (commitlint, husky, semantic-release)
npm install

# 4. Get Flutter dependencies
fvm flutter pub get

# 5. Run code generation
fvm flutter pub run build_runner build --delete-conflicting-outputs
```

### Daily Development Loop

```sh
# Start the app (choose a platform)
fvm flutter run -d chrome           # Web
fvm flutter run -d linux            # Linux desktop
fvm flutter run                     # Default connected device

# Run tests
fvm flutter test

# Re-generate code after model changes
fvm flutter pub run build_runner build --delete-conflicting-outputs
```

### Adding a New Feature

1. Create the directory structure under `lib/features/<feature_name>/`.
2. Define domain entities (Freezed) and the abstract repository interface.
3. Implement the data layer (DTO, remote/local data sources, concrete repository).
4. Follow the [Feature Module Registration](#feature-module-registration) pattern to wire up routes, providers, and DI.
5. Build pages and widgets in `presentation/pages/` and `presentation/widgets/`.
6. Run `build_runner` to generate updated files.
7. Write unit tests for use-cases and providers, widget tests for pages.
8. Commit following the Conventional Commits format.

---

## Conventions

### Package Name and Namespace

#### Flutter Project Name

The canonical Flutter project name for this template is **`flutter_foundation`**. This value:

- Is set as the `name` field in `pubspec.yaml`.
- Must follow Dart's `snake_case` package naming rules: all lowercase, underscores as word separators, no hyphens or uppercase letters.
- Becomes the Dart import prefix: `package:flutter_foundation/...`.

#### Dart Package Namespace

All cross-boundary Dart imports use the package import form:

```dart
import 'package:flutter_foundation/core/routing/router.dart';
import 'package:flutter_foundation/shared/theme/app_theme.dart';
```

Relative imports are used only within the same feature module (see [Imports](#imports)).

#### Application IDs and Bundle Identifiers

The Ego Hygiene reverse-domain prefix is **`com.egohygiene`**. Platform-specific application identifiers follow the pattern `com.egohygiene.<app_name>`:

| Platform | Identifier | Template value |
|---|---|---|
| Android (`applicationId` in `build.gradle`) | `com.egohygiene.<app_name>` | `com.egohygiene.flutter_foundation` |
| iOS / macOS (`CFBundleIdentifier`) | `com.egohygiene.<productName>` (camelCase, no underscores) | `com.egohygiene.flutterFoundation` |
| Web (PWA `id` / `<title>`) | `com.egohygiene.<app_name>` | `com.egohygiene.flutter_foundation` |
| Linux (`APPLICATION_ID` in `CMakeLists.txt`) | `com.egohygiene.<app_name>` | `com.egohygiene.flutter_foundation` |
| Windows (app ID in `CMakeLists.txt`) | `com.egohygiene.<app_name>` | `com.egohygiene.flutter_foundation` |

> **iOS / macOS note:** Apple's `CFBundleIdentifier` permits underscores, but camelCase is the prevailing Apple-ecosystem convention and avoids edge cases in Xcode provisioning and App Store Connect.

#### Naming Rules for Apps Derived from This Template

When a new application is created from `flutter-foundation`:

1. Choose a concise, descriptive `snake_case` product name (e.g., `habit_tracker`).
2. Set `name: <product_name>` in `pubspec.yaml`.
3. Set `applicationId "com.egohygiene.<product_name>"` in `android/app/build.gradle`.
4. Set the Bundle Identifier to `com.egohygiene.<productName>` (camelCase, no underscores — e.g., `com.egohygiene.habitTracker`) in `ios/Runner.xcodeproj` and `macos/Runner.xcodeproj`.
5. Update `APPLICATION_ID` in `linux/CMakeLists.txt` and the app identifier in `windows/CMakeLists.txt` to `com.egohygiene.<product_name>`.
6. Update all `package:flutter_foundation/...` imports to `package:<product_name>/...` throughout the codebase.

---

### File and Directory Naming

- All Dart files use `snake_case`.
- Directories use `snake_case`.
- Classes use `PascalCase`.
- Private members use a leading underscore `_`.

### Barrel Exports

Each feature exposes a single barrel file `<feature>.dart` that re-exports only its public API. Internal implementation files should not be imported directly from outside the feature boundary.

### Imports

Use relative imports within a feature. Use package imports (`package:flutter_foundation/...`) when crossing feature or layer boundaries. Avoid deep relative paths (`../../..`).

### Tests

- Unit tests mirror the `lib/` structure under `test/`.
- Widget tests live in `test/shared/widgets/` or `test/features/<feature>/presentation/`.
- Integration tests live in `integration_test/`.
- Mock/fake implementations of abstract interfaces live in `test/helpers/`.

### Documentation

- Public classes and methods must have a doc comment (`///`).
- Architecture decisions that deviate from this document should be recorded in an `ADR/` directory at the repository root.
