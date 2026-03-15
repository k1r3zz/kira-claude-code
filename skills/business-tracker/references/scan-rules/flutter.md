# Scan Rules — Flutter/Dart

File priority rules for Flutter/Dart projects (tag: `flutter`).

| Priority | File patterns |
|----------|--------------|
| P0 | `*_store.dart`, `*_bloc.dart`, `*_cubit.dart`, `*_viewmodel.dart`, `*_service.dart` (exclude pure-technical services) |
| P1 | `*_model.dart`, `*_entity.dart`, `*_repository.dart`, `router.dart`, `routes.dart` |
| P2 | `*_utils.dart` / `*_helper.dart` with business vocabulary, `config.dart`, `constants.dart`, `*_api.dart` |
| Skip | `*.g.dart`, `*.freezed.dart`, `*.gen.dart`, `*_widget.dart` (pure UI), `test/`, `assets/`, `packages/`, `.pub-cache/`, `pubspec.yaml`, `build.gradle`, `Podfile` |
