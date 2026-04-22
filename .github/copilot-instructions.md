# PiliPala — Copilot Instructions

PiliPala is a third-party BiliBili client built with Flutter, targeting Android and iOS (desktop is not a current focus).

## Build & Commands

```bash
# Install dependencies
flutter pub get

# Run the app (connected device required)
flutter run

# Analyze (lint)
flutter analyze

# Run tests
flutter test

# Run a single test file
flutter test test/widget_test.dart

# Build Android APK (split by ABI)
flutter build apk --release --split-per-abi

# Build iOS (no codesign)
flutter build ios --release --no-codesign

# After building iOS, the .app file is at build/ios/iphoneos/Runner.app
cd build/ios/iphoneos && rm -rf Payload && mkdir Payload && cp -r Runner.app Payload/ && zip -r pilipala.ipa Payload && mv pilipala.ipa /Volumes/dotennin/app/ios/ && cd ../../..

# Regenerate Hive type adapters (run after changing @HiveType models)
flutter pub run build_runner build --delete-conflicting-outputs
```

The CI workflow (`.github/workflows/`) targets Flutter **3.19.6** (stable channel).

## Architecture

### State Management — GetX
The app uses [GetX](https://pub.dev/packages/get) for routing, state management, and dependency injection throughout. Every feature page follows this three-file pattern:

```
pages/<feature>/
  index.dart       # barrel file: exports controller and view
  controller.dart  # GetxController subclass — all business logic and state
  view.dart        # StatefulWidget or StatelessWidget — pure UI
  widgets/         # page-local widgets
```

Controllers are instantiated with `Get.put(SomeController())` in the corresponding view. Reactive state uses `.obs` (e.g., `RxBool`, `RxList`, `RxString`).

### Routing
All routes are registered in `lib/router/app_pages.dart` using a custom `CustomGetPage` wrapper (sets `Transition.native`, disables pop gesture). Navigation uses `Get.toNamed('/route', arguments: {...}, parameters: {...})`. Query params are passed as URL parameters (e.g., `/video?bvid=xx&cid=xx`).

### HTTP Layer
`lib/http/` contains:
- `init.dart` — Singleton `Request` class wrapping Dio; handles cookie injection, WBI signature, CSRF token, buvid, and proxy settings.
- `api.dart` — All API endpoint URL constants.
- `constants.dart` — Base URL constants (`HttpString`).
- `interceptor.dart` — `ApiInterceptor` for error toasts; suppresses heartbeat/analytics errors.
- Feature-specific files (e.g., `video.dart`, `search.dart`) each contain static HTTP methods returning `Map<String, dynamic>` with `{'status': bool, 'data': ..., 'msg': ...}`.

The Bilibili API reference used is [bilibili-API-collect](https://github.com/SocialSisterYi/bilibili-API-collect).

### Persistence — Hive
`lib/utils/storage.dart` defines `GStrorage` (note: typo is intentional — do not rename) with five Hive boxes:
- `setting` — user settings (keys in `SettingBoxKey`)
- `userInfo` — logged-in user data
- `localCache` — runtime caches (keys in `LocalCacheKey`)
- `historyWord` — search history
- `video` — video-specific settings

Access boxes via `GStrorage.setting`, `GStrorage.localCache`, etc. (always use the static references, never open boxes manually).

### Global State
`GlobalDataCache` (singleton, initialized at startup) holds frequently-accessed settings so pages don't each read from Hive on every build. Update it when settings change.

### Video Player
The custom player lives in `lib/plugin/pl_player/`. It wraps `media_kit` and exposes a `PlPlayerController`. The player controller is separate from page GetX controllers and is passed around explicitly.

### Services
`lib/services/service_locator.dart` initializes `videoPlayerServiceHandler` (audio_service background playback) and `audioSessionHandler`. Called once in `onReady` inside `BuildMainApp`.

### Plugins
`lib/plugin/` contains three inline plugins:
- `pl_player/` — custom video player UI and controller
- `pl_gallery/` — image gallery viewer
- `pl_popup/` — custom popup/bottom-sheet

## Key Conventions

- **Barrel files**: Each feature in `pages/` exports via `index.dart`. Always import the barrel, not individual files.
- **Settings keys**: Use constants from `SettingBoxKey` (in `storage.dart`) when reading/writing the `setting` box. Never use raw string keys.
- **Route navigation helpers**: Use `RoutePush` (`lib/utils/route_push.dart`) for common navigations (bangumi, login, video) instead of calling `Get.toNamed` directly.
- **Toasts/dialogs**: Use `SmartDialog` (`flutter_smart_dialog`) for all toasts and loading indicators — not `ScaffoldMessenger`.
- **Localization**: The app is Chinese-first (`zh_CN`). UI strings are written inline in Chinese.
- **Platform branching**: Android-specific code is guarded with `Platform.isAndroid`. The `MyApp` widget builds `AndroidApp` (supports dynamic color via `DynamicColorBuilder`) or `OtherApp` accordingly.
- **Models**: Data models live in `lib/models/` grouped by domain. Models with Hive persistence have a `.g.dart` generated file — regenerate with `build_runner` when the model changes.
- **HTTP response shape**: All HTTP methods return `{'status': true/false, 'data': <model>, 'msg': <string>}`. Always check `result['status']` before accessing `result['data']`.
