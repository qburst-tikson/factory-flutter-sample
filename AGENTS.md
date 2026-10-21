# AGENTS.md — rules for Factory Droids on this Flutter project

Every Droid (planning, implementation, review, QA, docs) reads this file on every run and must follow it exactly.

## Project
- Flutter (stable channel), Dart 3, null-safe.
- Targets: Android + iOS.

## Architecture
- Clean Architecture, feature-first folders:
  ```
  lib/
    core/            # shared: di, network, error, theme, utils, widgets
    features/<name>/
      data/          # models (DTOs), datasources, repository implementations
      domain/        # entities, repository contracts, use cases
      presentation/  # bloc/, pages/, widgets/
  ```
- State management: **flutter_bloc** only. Cubit for simple flows, Bloc when there are many events.
  One per screen or flow. States are `sealed` classes extending `Equatable`
  (see `features/home/presentation/cubit/home_state.dart`; copy that pattern).
- Dependency injection: plain `get_it` in `lib/core/di/injection.dart` (no code generation).
  Repositories and use cases are `registerLazySingleton`; cubits/blocs are `registerFactory`.
  Add a commented block per feature. Never instantiate repositories inside widgets.
- Errors: data layer catches raw exceptions and throws a `Failure` subtype from
  `lib/core/error/failure.dart`. Cubits catch `Failure` and emit an error state.
  The UI never sees raw exceptions. Do not add dartz/fpdart.
- Use cases implement `UseCase<Output, Params>` from `lib/core/usecase/usecase.dart`.
- Pages: a `XxxPage` that provides the cubit via `sl<XxxCubit>()`, and a separate
  `XxxView` with the UI so widget tests can inject a mock cubit.
- Networking: no HTTP client yet. If a ticket needs one, add `dio` (listed in the plan)
  with a remote data source under `features/<name>/data/datasources/`.
- No business logic in widgets. Widgets render state and call cubit methods.
- The `home` feature is the reference implementation; follow its structure.

## Code style
- Must pass `dart format .` and `flutter analyze` with zero issues.
- Prefer `const` constructors. Keep widgets small; extract when a build method gets long.
- No new packages unless the approved plan lists them. Pin versions in pubspec.yaml.
- No hard-coded secrets, API keys or URLs — use the existing config/env mechanism.
- User-facing strings go through the existing localisation setup if the project has one.

## Tests
- Every new Bloc/Cubit gets `bloc_test` tests (success, failure, edge cases).
- Every new use case and repository gets unit tests (mock with `mocktail`).
- Widget tests for new screens covering the main states (loading / loaded / error).

## Git commit format (MANDATORY)
Conventional Commits, Jira key at the end:

```
<type>(<scope>): <short imperative summary, max 72 chars> [<JIRA-KEY>]

<body: what changed and WHY, wrapped at 72 chars>
```

- `type`: feat | fix | refactor | test | docs | chore | style | perf | build | ci
- `scope`: the feature folder or area (auth, profile, core, deps …)
- One logical change per commit. Tests may go with the code they test.
- Examples:
  - `feat(auth): add LoginBloc with email/password flow [FF-12]`
  - `test(auth): cover LoginBloc success and failure states [FF-12]`
  - `chore(deps): add flutter_bloc 9.x [FF-12]`

## Branches and pull requests
- Branch from `main` as `feature/<JIRA-KEY>-<short-slug>` (bugs: `fix/<JIRA-KEY>-<short-slug>`).
- Run `dart format .`, `flutter analyze` and `flutter test` before opening or updating a PR; all must pass.
- PR title: `[<JIRA-KEY>] <ticket summary>`.
- PR description: use `.github/pull_request_template.md` and fill in **every** section — no section
  left as a placeholder:
  - **Ticket**: the first line of the PR must be the clickable Jira link, e.g.
    `**Jira:** [FF-12](https://qburst-ai-sample-flutter.atlassian.net/browse/FF-12) — <ticket summary>`.
    Repeat the full link on the last line next to `Closes FF-12`. Replace every `FF-___` placeholder.
  - **Summary**.
  - **Acceptance criteria**: copy each criterion from the ticket, tick it, and name the file/widget/cubit
    that implements it. If one is not met, leave it unticked and explain why.
  - **What changed (by layer)**: domain, data, presentation, DI/core ("No change" where nothing changed).
  - **Files changed** table (file → purpose).
  - **Tests**: list every new or updated test file and what it covers, plus the real results
    (`flutter analyze` issue count, `flutter test` passed/failed counts from your last run).
  - **How to test manually**: numbered steps a tester can follow on the APK, including the demo login.
  - **Screens affected**, **Risks / reviewer focus**, **Plan & deviations** (link the `[AI-PLAN]` comment
    if there is one), **Commits** (one line each), and `Closes <JIRA-KEY>`.
- Add the labels `factory` and the ticket key (e.g. `FF-12`) to the PR.

## Jira updates
- When you start: comment "Factory Droid started work" and move the ticket to *In Development*.
- When the PR is open: comment a short summary on the ticket — what changed (2–3 bullets), tests added
  with results, how to test, and the PR link — then move it to *In Review*.
- When the PR is merged: move the ticket to *QA* and comment "Merged in PR #<n> — ready for QA", with the
  debug APK link from the PR's CI comment if available.

## Never
- Push to `main` directly, force-push, rebase shared branches, or touch other people's branches.
- Modify `.github/workflows/`, signing configs, `android/key.properties` or `ios/Runner.xcodeproj` signing settings.
- Delete or skip existing tests to make the suite pass.
- Commit secrets, tokens or credentials.
