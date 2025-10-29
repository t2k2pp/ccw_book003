# 第3章: 状態管理（Riverpod 3.x）

## この章で学べること

- Riverpod 3.x の基本と最新機能
- コード生成による型安全な状態管理
- AsyncNotifierProvider の活用法
- 他の状態管理ソリューションとの比較

## なぜRiverpod 3.xなのか

### 2025年の状態管理トレンド

| ソリューション | 状態 | 推奨度 |
|--------------|------|-------|
| Riverpod 3.x | 最新・推奨 | ⭐⭐⭐⭐⭐ |
| Bloc 8.x | 安定・企業向け | ⭐⭐⭐⭐ |
| Provider 6.x | メンテナンスモード | ⭐⭐ |
| GetX | 非推奨 | ⭐ |

### Riverpod 3.xの主な改善点

1. **完全なコード生成サポート**: `@riverpod`アノテーションで自動生成
2. **型安全性の向上**: コンパイル時エラー検出
3. **AsyncNotifier**: 非同期状態管理の標準化
4. **デバッグ機能の強化**: DevTools統合

## セットアップ

### pubspec.yaml

```yaml
dependencies:
  flutter:
    sdk: flutter
  flutter_riverpod: ^2.5.0
  riverpod_annotation: ^2.5.0

dev_dependencies:
  build_runner: ^2.4.0
  riverpod_generator: ^2.5.0
  riverpod_lint: ^2.3.0  # Linter
  custom_lint: ^0.6.0
```

### analysis_options.yaml

```yaml
analyzer:
  plugins:
    - custom_lint

# Riverpod Lintを有効化
custom_lint:
  rules:
    - provider_dependencies
    - scoped_providers_should_specify_dependencies
```

### main.dart

```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

void main() {
  runApp(
    // ProviderScopeで全体をラップ
    const ProviderScope(
      child: MyApp(),
    ),
  );
}
```

## 基本的な使い方

### 1. シンプルなProvider

```dart
import 'package:riverpod_annotation/riverpod_annotation.dart';

part 'counter_provider.g.dart';

// シンプルな値を提供
@riverpod
int counter(CounterRef ref) {
  return 0;
}

// 画面での使用
class CounterScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final counter = ref.watch(counterProvider);

    return Text('Count: $counter');
  }
}
```

### 2. Notifier（状態変更可能）

```dart
import 'package:riverpod_annotation/riverpod_annotation.dart';

part 'counter_notifier.g.dart';

@riverpod
class Counter extends _$Counter {
  @override
  int build() {
    return 0;  // 初期値
  }

  void increment() {
    state++;
  }

  void decrement() {
    state--;
  }

  void reset() {
    state = 0;
  }
}

// 使用例
class CounterScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final counter = ref.watch(counterProvider);

    return Column(
      children: [
        Text('Count: $counter'),
        ElevatedButton(
          onPressed: () => ref.read(counterProvider.notifier).increment(),
          child: const Text('Increment'),
        ),
      ],
    );
  }
}
```

### 3. AsyncNotifierProvider（非同期処理）

```dart
import 'package:riverpod_annotation/riverpod_annotation.dart';

part 'user_provider.g.dart';

@riverpod
class UserNotifier extends _$UserNotifier {
  @override
  Future<User?> build() async {
    // 初期データの取得
    return await _fetchUser();
  }

  Future<void> login(String email, String password) async {
    // ローディング状態にする
    state = const AsyncValue.loading();

    // 非同期処理を実行
    state = await AsyncValue.guard(() async {
      final user = await _authService.login(email, password);
      return user;
    });
  }

  Future<void> logout() async {
    state = const AsyncValue.loading();
    await _authService.logout();
    state = const AsyncValue.data(null);
  }

  Future<User> _fetchUser() async {
    // 実装
  }
}

// 使用例
class UserScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final userAsync = ref.watch(userNotifierProvider);

    return userAsync.when(
      data: (user) {
        if (user == null) {
          return const Text('Not logged in');
        }
        return Text('Welcome, ${user.name}');
      },
      loading: () => const CircularProgressIndicator(),
      error: (error, stack) => Text('Error: $error'),
    );
  }
}
```

## 依存関係の管理

### Providerの依存

```dart
// APIクライアントProvider
@riverpod
Dio dio(DioRef ref) {
  final dio = Dio(BaseOptions(
    baseUrl: 'https://api.example.com',
  ));

  // 認証トークンを追加
  dio.interceptors.add(
    InterceptorsWrapper(
      onRequest: (options, handler) {
        final token = ref.read(authTokenProvider);
        if (token != null) {
          options.headers['Authorization'] = 'Bearer $token';
        }
        handler.next(options);
      },
    ),
  );

  return dio;
}

// RepositoryがDioに依存
@riverpod
UserRepository userRepository(UserRepositoryRef ref) {
  final dio = ref.watch(dioProvider);
  return UserRepositoryImpl(dio);
}

// NotifierがRepositoryに依存
@riverpod
class UserList extends _$UserList {
  @override
  Future<List<User>> build() async {
    final repository = ref.watch(userRepositoryProvider);
    return await repository.getUsers();
  }

  Future<void> refresh() async {
    state = const AsyncValue.loading();
    final repository = ref.watch(userRepositoryProvider);
    state = await AsyncValue.guard(() => repository.getUsers());
  }
}
```

### Family（パラメータ付きProvider）

```dart
// ユーザーIDを受け取るProvider
@riverpod
Future<User> user(UserRef ref, String userId) async {
  final repository = ref.watch(userRepositoryProvider);
  return await repository.getUserById(userId);
}

// 使用例
class UserDetailScreen extends ConsumerWidget {
  final String userId;

  const UserDetailScreen({required this.userId});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final userAsync = ref.watch(userProvider(userId));

    return userAsync.when(
      data: (user) => Text(user.name),
      loading: () => const CircularProgressIndicator(),
      error: (error, stack) => Text('Error: $error'),
    );
  }
}
```

### AutoDispose（自動破棄）

```dart
// 画面を離れたら自動的に破棄される
@riverpod
Future<List<Todo>> todos(TodosRef ref) async {
  final repository = ref.watch(todoRepositoryProvider);

  // キャンセル処理の登録
  final cancelToken = CancelToken();
  ref.onDispose(() {
    cancelToken.cancel();
  });

  return await repository.getTodos(cancelToken: cancelToken);
}

// 自動破棄を無効にしてキャッシュを保持
@Riverpod(keepAlive: true)
Future<AppConfig> appConfig(AppConfigRef ref) async {
  // アプリ起動時に1度だけ読み込み、キャッシュし続ける
  return await ConfigService.load();
}
```

## 実践的なパターン

### パターン1: ページング（無限スクロール）

```dart
@riverpod
class PostList extends _$PostList {
  int _page = 1;
  bool _hasMore = true;

  @override
  Future<List<Post>> build() async {
    return await _fetchPosts();
  }

  Future<void> fetchMore() async {
    if (!_hasMore) return;

    _page++;
    final newPosts = await _fetchPosts();

    if (newPosts.isEmpty) {
      _hasMore = false;
    } else {
      state = AsyncValue.data([
        ...state.value ?? [],
        ...newPosts,
      ]);
    }
  }

  Future<void> refresh() async {
    _page = 1;
    _hasMore = true;
    state = const AsyncValue.loading();
    state = await AsyncValue.guard(() => _fetchPosts());
  }

  Future<List<Post>> _fetchPosts() async {
    final repository = ref.read(postRepositoryProvider);
    return await repository.getPosts(page: _page, limit: 20);
  }
}
```

### パターン2: リアルタイム更新（Stream）

```dart
@riverpod
Stream<List<Message>> messages(MessagesRef ref, String chatRoomId) {
  final repository = ref.watch(messageRepositoryProvider);
  return repository.watchMessages(chatRoomId);
}

// 使用例
class ChatScreen extends ConsumerWidget {
  final String chatRoomId;

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final messagesAsync = ref.watch(messagesProvider(chatRoomId));

    return messagesAsync.when(
      data: (messages) => ListView.builder(
        itemCount: messages.length,
        itemBuilder: (context, index) {
          return MessageBubble(message: messages[index]);
        },
      ),
      loading: () => const CircularProgressIndicator(),
      error: (error, stack) => Text('Error: $error'),
    );
  }
}
```

### パターン3: 複数Providerの組み合わせ

```dart
// フィルター設定
@riverpod
class TodoFilter extends _$TodoFilter {
  @override
  TodoFilterType build() => TodoFilterType.all;

  void setFilter(TodoFilterType filter) {
    state = filter;
  }
}

// フィルター済みTodoリスト
@riverpod
Future<List<Todo>> filteredTodos(FilteredTodosRef ref) async {
  final filter = ref.watch(todoFilterProvider);
  final allTodos = await ref.watch(todoListProvider.future);

  switch (filter) {
    case TodoFilterType.all:
      return allTodos;
    case TodoFilterType.completed:
      return allTodos.where((todo) => todo.isCompleted).toList();
    case TodoFilterType.active:
      return allTodos.where((todo) => !todo.isCompleted).toList();
  }
}
```

### パターン4: 楽観的UI更新

```dart
@riverpod
class TodoList extends _$TodoList {
  @override
  Future<List<Todo>> build() async {
    final repository = ref.watch(todoRepositoryProvider);
    return await repository.getTodos();
  }

  Future<void> toggleTodo(String todoId) async {
    final repository = ref.watch(todoRepositoryProvider);

    // 楽観的更新：即座にUIを更新
    state = AsyncValue.data(
      state.value!.map((todo) {
        if (todo.id == todoId) {
          return todo.copyWith(isCompleted: !todo.isCompleted);
        }
        return todo;
      }).toList(),
    );

    try {
      // サーバーに送信
      await repository.toggleTodo(todoId);
    } catch (e) {
      // 失敗時は元に戻す
      ref.invalidateSelf();
    }
  }
}
```

## テスト

### Providerのテスト

```dart
import 'package:flutter_test/flutter_test.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

void main() {
  test('Counter increments correctly', () {
    final container = ProviderContainer();
    addTearDown(container.dispose);

    // 初期値の確認
    expect(container.read(counterProvider), 0);

    // incrementを実行
    container.read(counterProvider.notifier).increment();

    // 値が増えたことを確認
    expect(container.read(counterProvider), 1);
  });

  test('UserNotifier fetches user data', () async {
    final container = ProviderContainer(
      overrides: [
        // Repositoryをモックに差し替え
        userRepositoryProvider.overrideWithValue(MockUserRepository()),
      ],
    );
    addTearDown(container.dispose);

    final userAsync = container.read(userNotifierProvider);

    expect(userAsync.isLoading, true);

    await container.read(userNotifierProvider.future);

    final user = container.read(userNotifierProvider).value;
    expect(user, isNotNull);
    expect(user!.name, 'Test User');
  });
}
```

## 他の状態管理ソリューションとの比較

### Riverpod vs Bloc

| 観点 | Riverpod 3.x | Bloc 8.x |
|------|-------------|----------|
| 学習コスト | 低 | 中 |
| ボイラープレート | 少ない（コード生成） | 多い |
| テストのしやすさ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| 非同期処理 | AsyncNotifier（シンプル） | Stream（柔軟） |
| 企業採用 | 増加中 | 広く採用されている |
| 適用場面 | 中小規模〜大規模 | 大規模・複雑なロジック |

**Blocを選ぶべき場合**:
- 既にBlocを使っているプロジェクト
- イベント駆動の複雑な状態遷移がある
- チームが既にBlocに精通している

**Riverpodを選ぶべき場合**:
- 新規プロジェクト
- 開発速度を重視
- シンプルな状態管理で済む

### Riverpod vs Provider

Providerは**Riverpodの前身**であり、2025年時点ではRiverpodへの移行が推奨されています。

**主な違い**:
- Riverpod: コンパイル時の型安全性
- Provider: `BuildContext`に依存
- Riverpod: テストがより簡単

## AI開発時の注意点

### AIが間違いやすいパターン

#### 1. 古いProvider記法を生成する

**NG（古い記法）**:
```dart
final counterProvider = StateProvider<int>((ref) => 0);
```

**OK（2025年推奨）**:
```dart
@riverpod
class Counter extends _$Counter {
  @override
  int build() => 0;

  void increment() => state++;
}
```

#### 2. AsyncValueの扱いを忘れる

**NG**:
```dart
final user = ref.watch(userProvider).value!; // Null安全でない
```

**OK**:
```dart
final userAsync = ref.watch(userProvider);
return userAsync.when(
  data: (user) => ...,
  loading: () => ...,
  error: (error, stack) => ...,
);
```

#### 3. ref.readとref.watchの混同

```dart
// build内では ref.watch を使う
@override
Widget build(BuildContext context, WidgetRef ref) {
  final counter = ref.watch(counterProvider); // OK

  return ElevatedButton(
    // イベントハンドラ内では ref.read を使う
    onPressed: () => ref.read(counterProvider.notifier).increment(), // OK
    child: Text('$counter'),
  );
}
```

## コード生成の実行

```bash
# 1回だけ実行
dart run build_runner build --delete-conflicting-outputs

# ファイル変更を監視して自動生成
dart run build_runner watch --delete-conflicting-outputs
```

## チェックリスト

### Riverpodセットアップ時

- [ ] `flutter_riverpod`, `riverpod_annotation`, `riverpod_generator`をインストール
- [ ] `ProviderScope`でアプリをラップ
- [ ] `build_runner`を実行して`.g.dart`ファイルを生成
- [ ] `riverpod_lint`を有効化

### Provider実装時

- [ ] `@riverpod`アノテーションを使用
- [ ] `part 'xxx.g.dart';`を記載
- [ ] AsyncValueを適切に処理（when/whenData/maybeWhen）
- [ ] `ref.read`と`ref.watch`を正しく使い分け
- [ ] AutoDisposeの必要性を検討

## 次のステップ

状態管理の基礎を理解したら、[第4章: 主要ライブラリとパッケージ](04_essential_packages.md)で、Riverpodと組み合わせて使う推奨パッケージを学びましょう。

---

**AI開発のヒント**:
生成AIに「Riverpod 3.xでログイン機能を実装して」と依頼する際は、この章の「AsyncNotifierProvider」セクションを一緒に渡すと、最新の記法で実装してくれます。
