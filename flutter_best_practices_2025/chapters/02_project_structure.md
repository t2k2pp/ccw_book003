# 第2章: プロジェクト構造とアーキテクチャパターン

## この章で学べること

- 2025年推奨のフォルダ構成
- Feature-First vs Layer-First アプローチ
- Clean Architectureの実践的な適用
- パッケージ分割戦略

## フォルダ構成のベストプラクティス

### 小規模プロジェクト（~5画面）

```
lib/
├── main.dart
├── app.dart
├── models/
│   ├── user.dart
│   └── post.dart
├── screens/
│   ├── home_screen.dart
│   ├── profile_screen.dart
│   └── settings_screen.dart
├── widgets/
│   ├── custom_button.dart
│   └── user_card.dart
├── services/
│   ├── api_service.dart
│   └── auth_service.dart
└── utils/
    ├── constants.dart
    └── helpers.dart
```

**適用場面**: MVP、プロトタイプ、学習用プロジェクト

### 中規模プロジェクト（Feature-First）

```
lib/
├── main.dart
├── app.dart
├── core/
│   ├── router/
│   │   ├── app_router.dart
│   │   └── app_router.g.dart
│   ├── theme/
│   │   ├── app_theme.dart
│   │   └── colors.dart
│   ├── utils/
│   │   ├── extensions.dart
│   │   └── validators.dart
│   └── widgets/
│       ├── loading_indicator.dart
│       └── error_widget.dart
├── features/
│   ├── auth/
│   │   ├── data/
│   │   │   ├── models/
│   │   │   │   └── user_model.dart
│   │   │   ├── repositories/
│   │   │   │   └── auth_repository.dart
│   │   │   └── data_sources/
│   │   │       └── auth_remote_data_source.dart
│   │   ├── domain/
│   │   │   ├── entities/
│   │   │   │   └── user.dart
│   │   │   └── usecases/
│   │   │       └── login_usecase.dart
│   │   └── presentation/
│   │       ├── screens/
│   │       │   ├── login_screen.dart
│   │       │   └── signup_screen.dart
│   │       ├── widgets/
│   │       │   └── auth_form.dart
│   │       └── providers/
│   │           └── auth_provider.dart
│   ├── home/
│   │   └── ... (same structure)
│   └── profile/
│       └── ... (same structure)
└── shared/
    ├── models/
    ├── widgets/
    └── utils/
```

**適用場面**: 10-50画面、複数人開発、中期以上の開発期間

### 大規模プロジェクト（Layer-First with Features）

```
lib/
├── main.dart
├── app.dart
├── config/
│   ├── env/
│   │   ├── env.dart
│   │   └── env.g.dart
│   ├── router/
│   └── theme/
├── core/
│   ├── error/
│   │   ├── exceptions.dart
│   │   └── failures.dart
│   ├── network/
│   │   ├── dio_client.dart
│   │   └── network_info.dart
│   ├── usecases/
│   │   └── usecase.dart
│   └── utils/
├── features/
│   └── [feature_name]/
│       ├── data/
│       ├── domain/
│       └── presentation/
└── l10n/
    ├── app_en.arb
    └── app_ja.arb
```

**適用場面**: 大規模アプリ、長期運用、多国籍チーム

## Feature-First vs Layer-First

### Feature-First（推奨：中規模以上）

**メリット**:
- 機能ごとにコードが集約されている
- 新機能追加時の影響範囲が明確
- チーム分担がしやすい
- 機能削除時にフォルダごと削除可能

**デメリット**:
- 初期のフォルダ構造が複雑に見える
- 小規模プロジェクトでは過剰

**実装例**:

```dart
// features/auth/presentation/providers/auth_provider.dart
import 'package:riverpod_annotation/riverpod_annotation.dart';
import '../../domain/usecases/login_usecase.dart';

part 'auth_provider.g.dart';

@riverpod
class AuthNotifier extends _$AuthNotifier {
  @override
  AsyncValue<User?> build() {
    return const AsyncValue.data(null);
  }

  Future<void> login(String email, String password) async {
    state = const AsyncValue.loading();
    final usecase = ref.read(loginUsecaseProvider);
    final result = await usecase(LoginParams(email, password));

    state = result.fold(
      (failure) => AsyncValue.error(failure, StackTrace.current),
      (user) => AsyncValue.data(user),
    );
  }
}
```

### Layer-First

**メリット**:
- レイヤー間の責任が明確
- 小規模プロジェクトでシンプル
- 学習コストが低い

**デメリット**:
- 機能追加時に複数フォルダを横断
- 大規模プロジェクトで管理が困難

## Clean Architectureの実践

### 3層アーキテクチャ

```
Feature/
├── data/           # データ層（外部世界とのやり取り）
├── domain/         # ドメイン層（ビジネスロジック）
└── presentation/   # プレゼンテーション層（UI）
```

### 各層の責務

#### 1. Domain Layer（ドメイン層）

**責務**: ビジネスロジックを定義。他の層に依存しない。

```dart
// features/todo/domain/entities/todo.dart
class Todo {
  final String id;
  final String title;
  final bool isCompleted;

  const Todo({
    required this.id,
    required this.title,
    required this.isCompleted,
  });
}

// features/todo/domain/repositories/todo_repository.dart
abstract class TodoRepository {
  Future<List<Todo>> getTodos();
  Future<void> addTodo(Todo todo);
  Future<void> updateTodo(Todo todo);
  Future<void> deleteTodo(String id);
}

// features/todo/domain/usecases/get_todos_usecase.dart
class GetTodosUsecase {
  final TodoRepository repository;

  GetTodosUsecase(this.repository);

  Future<List<Todo>> call() {
    return repository.getTodos();
  }
}
```

#### 2. Data Layer（データ層）

**責務**: 外部データソース（API, DB）とのやり取り。Domainへの変換。

```dart
// features/todo/data/models/todo_model.dart
import 'package:freezed_annotation/freezed_annotation.dart';
import '../../domain/entities/todo.dart';

part 'todo_model.freezed.dart';
part 'todo_model.g.dart';

@freezed
class TodoModel with _$TodoModel {
  const factory TodoModel({
    required String id,
    required String title,
    @JsonKey(name: 'is_completed') required bool isCompleted,
  }) = _TodoModel;

  factory TodoModel.fromJson(Map<String, dynamic> json) =>
      _$TodoModelFromJson(json);
}

extension TodoModelX on TodoModel {
  Todo toEntity() => Todo(
        id: id,
        title: title,
        isCompleted: isCompleted,
      );
}

// features/todo/data/repositories/todo_repository_impl.dart
class TodoRepositoryImpl implements TodoRepository {
  final TodoRemoteDataSource remoteDataSource;
  final TodoLocalDataSource localDataSource;

  TodoRepositoryImpl({
    required this.remoteDataSource,
    required this.localDataSource,
  });

  @override
  Future<List<Todo>> getTodos() async {
    try {
      final models = await remoteDataSource.getTodos();
      // キャッシュに保存
      await localDataSource.cacheTodos(models);
      return models.map((m) => m.toEntity()).toList();
    } catch (e) {
      // オフライン時はキャッシュから取得
      final cached = await localDataSource.getCachedTodos();
      return cached.map((m) => m.toEntity()).toList();
    }
  }
}
```

#### 3. Presentation Layer（プレゼンテーション層）

**責務**: UIとユーザーインタラクション。状態管理。

```dart
// features/todo/presentation/providers/todo_provider.dart
import 'package:riverpod_annotation/riverpod_annotation.dart';

part 'todo_provider.g.dart';

@riverpod
class TodoList extends _$TodoList {
  @override
  Future<List<Todo>> build() async {
    final usecase = ref.read(getTodosUsecaseProvider);
    return await usecase();
  }

  Future<void> addTodo(String title) async {
    final usecase = ref.read(addTodoUsecaseProvider);
    await usecase(title);
    ref.invalidateSelf(); // 再読み込み
  }
}

// features/todo/presentation/screens/todo_screen.dart
class TodoScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final todosAsync = ref.watch(todoListProvider);

    return Scaffold(
      body: todosAsync.when(
        data: (todos) => ListView.builder(
          itemCount: todos.length,
          itemBuilder: (context, index) {
            return TodoItem(todo: todos[index]);
          },
        ),
        loading: () => const Center(child: CircularProgressIndicator()),
        error: (error, stack) => ErrorWidget(error: error),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () => _showAddTodoDialog(context, ref),
        child: const Icon(Icons.add),
      ),
    );
  }
}
```

## Dependency Injection（依存性注入）

### Riverpodを使ったDI

```dart
// features/todo/data/data_sources/todo_remote_data_source.dart
@riverpod
TodoRemoteDataSource todoRemoteDataSource(TodoRemoteDataSourceRef ref) {
  final dio = ref.watch(dioProvider);
  return TodoRemoteDataSourceImpl(dio);
}

// features/todo/data/repositories/todo_repository_impl.dart
@riverpod
TodoRepository todoRepository(TodoRepositoryRef ref) {
  return TodoRepositoryImpl(
    remoteDataSource: ref.watch(todoRemoteDataSourceProvider),
    localDataSource: ref.watch(todoLocalDataSourceProvider),
  );
}

// features/todo/domain/usecases/get_todos_usecase.dart
@riverpod
GetTodosUsecase getTodosUsecase(GetTodosUsecaseRef ref) {
  return GetTodosUsecase(ref.watch(todoRepositoryProvider));
}
```

**メリット**:
- テスト時にモックへの差し替えが簡単
- 依存関係が明示的
- コンパイル時の型チェック

## パッケージ分割戦略

### モノレポ vs マルチレポ

#### モノレポ（推奨）

```
my_app/
├── apps/
│   ├── mobile_app/
│   └── admin_app/
├── packages/
│   ├── core/
│   ├── ui_kit/
│   ├── api_client/
│   └── domain/
└── melos.yaml
```

**メリット**:
- コードの再利用が容易
- バージョン管理が統一
- リファクタリングが安全

**ツール**: Melos

```yaml
# melos.yaml
name: my_app
packages:
  - apps/*
  - packages/*

scripts:
  analyze:
    run: flutter analyze
    exec:
      concurrency: 1

  test:
    run: flutter test
    exec:
      concurrency: 1
```

#### マルチパッケージ構成例

```dart
// packages/core/lib/core.dart
export 'src/errors/failures.dart';
export 'src/network/dio_client.dart';
export 'src/utils/extensions.dart';

// apps/mobile_app/pubspec.yaml
dependencies:
  core:
    path: ../../packages/core
  ui_kit:
    path: ../../packages/ui_kit
```

## AI開発時の注意点

### AIが間違いやすいパターン

#### 1. 循環依存を作ってしまう

**NG例**:
```dart
// domain layer が presentation layer に依存
class UseCase {
  final SomeProvider provider; // NG!
}
```

**OK例**:
```dart
// domain layer は他に依存しない
class UseCase {
  final SomeRepository repository; // OK
}
```

#### 2. 過度にレイヤーを分けすぎる

小規模機能に Clean Architecture を厳密に適用すると逆効果。

**ガイドライン**:
- 1-2画面の簡単な機能: 簡易構成
- 複雑なビジネスロジックがある: Clean Architecture

#### 3. ファイル名の不統一

**統一ルール**:
```
// モデル
user_model.dart

// プロバイダー
user_provider.dart

// 画面
user_screen.dart

// ウィジェット
user_card.dart

// usecase
get_user_usecase.dart
```

## テンプレート例

### 新機能追加時のテンプレート

```bash
# Masonでテンプレート化
mason make feature --name todo

# 生成されるファイル
features/todo/
├── data/
│   ├── models/
│   │   └── todo_model.dart
│   ├── repositories/
│   │   └── todo_repository_impl.dart
│   └── data_sources/
│       └── todo_remote_data_source.dart
├── domain/
│   ├── entities/
│   │   └── todo.dart
│   ├── repositories/
│   │   └── todo_repository.dart
│   └── usecases/
│       └── get_todos_usecase.dart
└── presentation/
    ├── screens/
    │   └── todo_screen.dart
    ├── widgets/
    │   └── todo_item.dart
    └── providers/
        └── todo_provider.dart
```

## チェックリスト

### プロジェクト構造設計時

- [ ] プロジェクト規模に応じた構成を選択（小/中/大）
- [ ] Feature-First or Layer-First を決定
- [ ] Clean Architecture の適用範囲を定義
- [ ] DI（Riverpod）の方針を統一
- [ ] ファイル命名規則を文書化

### 新機能追加時

- [ ] 既存の構造に従っているか
- [ ] 循環依存が発生していないか
- [ ] レイヤー間の責務が正しいか
- [ ] テストしやすい構造になっているか

## 次のステップ

プロジェクト構造が決まったら、[第3章: 状態管理](03_state_management.md)で具体的な状態管理の実装を学びましょう。

---

**AI開発のヒント**:
生成AIに「TodoアプリのCRUD機能を追加して」と依頼する際は、この章の「Clean Architectureの実践」セクションを一緒に渡すと、適切なレイヤー分けを行ってくれます。
