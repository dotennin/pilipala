# Repository Guidelines

## Project Structure & Module Organization
PiliPala is a Flutter app. Application code lives in `lib/`, with feature screens under `lib/pages/<feature>/` and the common `controller.dart`, `view.dart`, and `index.dart` pattern. Shared UI and skeleton components are in `lib/common/`, API clients are in `lib/http/`, data models are in `lib/models/`, utilities are in `lib/utils/`, and app routes are in `lib/router/`. Custom media/player helpers are under `lib/plugin/`, while platform projects live in `android/`, `ios/`, `macos/`, `linux/`, `windows/`, and `web/`. Static assets are declared in `pubspec.yaml` and stored in `assets/`.

## Build, Test, and Development Commands
- `flutter pub get`: install Dart and Flutter dependencies.
- `flutter run`: launch the app on the selected emulator, device, or web target.
- `flutter analyze`: run static analysis using `analysis_options.yaml`.
- `flutter test`: run widget and unit tests in `test/`.
- `flutter build apk`: build an Android release APK; use other Flutter build targets as needed.
- `dart run build_runner build --delete-conflicting-outputs`: regenerate generated Dart files such as Hive/json model outputs when source annotations change.

## Coding Style & Naming Conventions
Follow `package:flutter_lints/flutter.yaml`. Format Dart before submitting with `dart format lib test`. Use two-space indentation, `snake_case.dart` filenames, `UpperCamelCase` classes and widgets, and `lowerCamelCase` members. Keep feature-local widgets close to their page under `lib/pages/<feature>/widgets/`; promote reusable pieces to `lib/common/widgets/`. Do not hand-edit generated files such as `*.g.dart` or protobuf outputs.

## Testing Guidelines
Place tests in `test/` and name them `*_test.dart`. Prefer focused widget tests for UI behavior and unit tests for utility or parsing logic. Run `flutter test` before opening a PR, and run `flutter analyze` when touching shared APIs, models, routing, or platform code.

## Commit & Pull Request Guidelines
Recent history uses concise subjects, often Conventional Commit prefixes such as `fix:` and `chore:`, alongside occasional `Update README.md`. Prefer `type: short description`, for example `fix: correct rank tab mapping`. PRs should describe the user-visible change, list verification commands run, link related issues, and include screenshots or screen recordings for UI changes.

## Security & Configuration Tips
Do not commit secrets, cookies, signing files, or local configuration. Keep API behavior changes isolated in `lib/http/` and related models. Respect the local agent setting `DISABLE_CONFIG_BACKUP: true`; do not add config-backup steps unless explicitly requested.
