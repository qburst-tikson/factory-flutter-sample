# Factory Flutter Sample

Flutter sample app built end-to-end by **Factory AI** inside the Factory platform:
**Jira ticket (project FF) → Factory Droid plans and implements → Factory review / QA → pull request → human merge.**

Companion to [`ai-flutter-sample`](https://github.com/qburst-tikson/ai-flutter-sample), which runs the same app
through a Jira + GitHub Actions + Claude pipeline. The two setups are fully separate.

Rules every Droid follows: [`AGENTS.md`](AGENTS.md).

## Architecture

Clean Architecture, feature-first, with BLoC (Cubit), get_it and Equatable sealed states:

```
lib/
  core/            di, error, usecase, widgets
  features/
    auth/          login, session, logout
    home/          reference feature
    main_shell/    bottom navigation (Home / Search / Settings)
    search/
    settings/      shows the logged-in email
  app.dart
  main.dart
```

Demo login: `tikson@qburst.com` / `Avenger123`.

## Run locally

```bash
flutter pub get
flutter run
```

## Checks (same as CI)

```bash
dart format .
flutter analyze
flutter test
```
