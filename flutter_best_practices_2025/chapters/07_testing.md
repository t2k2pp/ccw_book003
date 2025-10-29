# 第7章: テスト戦略

## この章で学べること

- ユニットテスト・Widgetテスト・インテグレーションテストの書き方
- Riverpodのテスト方法
- モック・スタブの作成
- テストカバレッジの向上

## テストの種類

| テストタイプ | 実行速度 | 信頼性 | カバー範囲 | 推奨比率 |
|-----------|---------|--------|----------|----------|
| ユニットテスト | ⚡⚡⚡ | 低 | 狭い | 70% |
| Widgetテスト | ⚡⚡ | 中 | 中 | 20% |
| インテグレーションテスト | ⚡ | 高 | 広い | 10% |

**テストピラミッド**: ユニットテスト > Widgetテスト > インテグレーションテスト

## ユニットテスト

### セットアップ

```yaml
dev_dependencies:
  flutter_test:
    sdk: flutter
  mockito: ^5.4.0
  build_runner: ^2.4.0
```

### 基本的なテスト

```dart
// test/utils/validator_test.dart
import 'package:flutter_test/flutter_test.dart';
import 'package:my_app/utils/validator.dart';

void main() {
  group('Validator', () {
    group('isValidEmail', () {
      test('有効なメールアドレスの場合trueを返す', () {
        expect(Validator.isValidEmail('test@example.com'), true);
        expect(Validator.isValidEmail('user+tag@domain.co.jp'), true);
      });

      test('無効なメールアドレスの場合falseを返す', () {
        expect(Validator.isValidEmail('invalid'), false);
        expect(Validator.isValidEmail('@example.com'), false);
        expect(Validator.isValidEmail('test@'), false);
      });

      test('空文字の場合falseを返す', () {
        expect(Validator.isValidEmail(''), false);
      });
    });

    group('isValidPassword', () {
      test('8文字以上の場合trueを返す', () {
        expect(Validator.isValidPassword('password123'), true);
      });

      test('8文字未満の場合falseを返す', () {
        expect(Validator.isValidPassword('pass'), false);
      });
    });
  });
}
```

### Riverpod Providerのテスト

```dart
// test/providers/counter_provider_test.dart
import 'package:flutter_test/flutter_test.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:my_app/providers/counter_provider.dart';

void main() {
  group('CounterProvider', () {
    test('初期値は0', () {
      final container = ProviderContainer();
      addTearDown(container.dispose);

      expect(container.read(counterProvider), 0);
    });

    test('incrementで値が増える', () {
      final container = ProviderContainer();
      addTearDown(container.dispose);

      container.read(counterProvider.notifier).increment();

      expect(container.read(counterProvider), 1);
    });

    test('複数回incrementできる', () {
      final container = ProviderContainer();
      addTearDown(container.dispose);

      final notifier = container.read(counterProvider.notifier);
      notifier.increment();
      notifier.increment();
      notifier.increment();

      expect(container.read(counterProvider), 3);
    });
  });
}
```

### 非同期Providerのテスト

```dart
// test/providers/user_provider_test.dart
import 'package:flutter_test/flutter_test.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:mockito/mockito.dart';
import 'package:mockito/annotations.dart';

@GenerateMocks([UserRepository])
import 'user_provider_test.mocks.dart';

void main() {
  group('UserProvider', () {
    late MockUserRepository mockRepository;

    setUp(() {
      mockRepository = MockUserRepository();
    });

    test('ユーザー一覧を取得できる', () async {
      final users = [
        User(id: '1', name: 'Alice'),
        User(id: '2', name: 'Bob'),
      ];
      when(mockRepository.getUsers()).thenAnswer((_) async => users);

      final container = ProviderContainer(
        overrides: [
          userRepositoryProvider.overrideWithValue(mockRepository),
        ],
      );
      addTearDown(container.dispose);

      // Providerの初期化を待つ
      final userList = await container.read(userListProvider.future);

      expect(userList, users);
      verify(mockRepository.getUsers()).called(1);
    });

    test('エラー時はAsyncErrorになる', () async {
      when(mockRepository.getUsers()).thenThrow(Exception('Network error'));

      final container = ProviderContainer(
        overrides: [
          userRepositoryProvider.overrideWithValue(mockRepository),
        ],
      );
      addTearDown(container.dispose);

      final userListAsync = container.read(userListProvider);

      expect(userListAsync.hasError, true);
    });
  });
}
```

## Widgetテスト

### 基本的なWidgetテスト

```dart
// test/widgets/counter_widget_test.dart
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:my_app/widgets/counter_widget.dart';

void main() {
  testWidgets('Counterが表示される', (tester) async {
    await tester.pumpWidget(
      const MaterialApp(
        home: CounterWidget(),
      ),
    );

    // 初期値0が表示されている
    expect(find.text('0'), findsOneWidget);
    expect(find.text('1'), findsNothing);

    // +ボタンをタップ
    await tester.tap(find.byIcon(Icons.add));
    await tester.pump();

    // 値が1に増えている
    expect(find.text('0'), findsNothing);
    expect(find.text('1'), findsOneWidget);
  });

  testWidgets('複数回タップできる', (tester) async {
    await tester.pumpWidget(
      const MaterialApp(
        home: CounterWidget(),
      ),
    );

    // 3回タップ
    await tester.tap(find.byIcon(Icons.add));
    await tester.pump();
    await tester.tap(find.byIcon(Icons.add));
    await tester.pump();
    await tester.tap(find.byIcon(Icons.add));
    await tester.pump();

    expect(find.text('3'), findsOneWidget);
  });
}
```

### Riverpod統合のWidgetテスト

```dart
// test/screens/user_list_screen_test.dart
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:mockito/mockito.dart';
import 'package:my_app/screens/user_list_screen.dart';

@GenerateMocks([UserRepository])
import 'user_list_screen_test.mocks.dart';

void main() {
  late MockUserRepository mockRepository;

  setUp(() {
    mockRepository = MockUserRepository();
  });

  testWidgets('ユーザー一覧が表示される', (tester) async {
    final users = [
      User(id: '1', name: 'Alice'),
      User(id: '2', name: 'Bob'),
    ];
    when(mockRepository.getUsers()).thenAnswer((_) async => users);

    await tester.pumpWidget(
      ProviderScope(
        overrides: [
          userRepositoryProvider.overrideWithValue(mockRepository),
        ],
        child: const MaterialApp(
          home: UserListScreen(),
        ),
      ),
    );

    // ローディング表示
    expect(find.byType(CircularProgressIndicator), findsOneWidget);

    // データ読み込み完了を待つ
    await tester.pumpAndSettle();

    // ユーザー名が表示される
    expect(find.text('Alice'), findsOneWidget);
    expect(find.text('Bob'), findsOneWidget);
  });

  testWidgets('エラー時にエラーメッセージが表示される', (tester) async {
    when(mockRepository.getUsers()).thenThrow(Exception('Network error'));

    await tester.pumpWidget(
      ProviderScope(
        overrides: [
          userRepositoryProvider.overrideWithValue(mockRepository),
        ],
        child: const MaterialApp(
          home: UserListScreen(),
        ),
      ),
    );

    await tester.pumpAndSettle();

    expect(find.text('Error'), findsOneWidget);
  });
}
```

### スクロール・リストのテスト

```dart
testWidgets('リストをスクロールできる', (tester) async {
  await tester.pumpWidget(
    ProviderScope(
      overrides: [
        userRepositoryProvider.overrideWithValue(mockRepository),
      ],
      child: const MaterialApp(
        home: UserListScreen(),
      ),
    ),
  );

  await tester.pumpAndSettle();

  // 最初のアイテムは表示されている
  expect(find.text('User 1'), findsOneWidget);

  // 最後のアイテムはまだ表示されていない
  expect(find.text('User 100'), findsNothing);

  // リストを下までスクロール
  await tester.drag(find.byType(ListView), const Offset(0, -5000));
  await tester.pumpAndSettle();

  // 最後のアイテムが表示される
  expect(find.text('User 100'), findsOneWidget);
});
```

## インテグレーションテスト

### セットアップ

```yaml
dev_dependencies:
  integration_test:
    sdk: flutter
```

```dart
// integration_test/app_test.dart
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:integration_test/integration_test.dart';
import 'package:my_app/main.dart' as app;

void main() {
  IntegrationTestWidgetsFlutterBinding.ensureInitialized();

  group('E2Eテスト', () {
    testWidgets('ログインから投稿まで', (tester) async {
      app.main();
      await tester.pumpAndSettle();

      // ログイン画面が表示される
      expect(find.text('Login'), findsOneWidget);

      // メールアドレスを入力
      await tester.enterText(
        find.byType(TextField).first,
        'test@example.com',
      );

      // パスワードを入力
      await tester.enterText(
        find.byType(TextField).last,
        'password123',
      );

      // ログインボタンをタップ
      await tester.tap(find.text('Login'));
      await tester.pumpAndSettle();

      // ホーム画面に遷移
      expect(find.text('Home'), findsOneWidget);

      // 投稿画面へ
      await tester.tap(find.byIcon(Icons.add));
      await tester.pumpAndSettle();

      // 投稿内容を入力
      await tester.enterText(
        find.byType(TextField),
        'Test post',
      );

      // 投稿
      await tester.tap(find.text('Post'));
      await tester.pumpAndSettle();

      // 投稿が表示される
      expect(find.text('Test post'), findsOneWidget);
    });
  });
}
```

### 実行方法

```bash
# エミュレータ/シミュレータで実行
flutter test integration_test/app_test.dart

# 接続されたデバイスで実行
flutter test integration_test --dart-define=INTEGRATION_TEST=true
```

## モック・スタブ

### Mockitoでモックを作成

```dart
// test/mocks.dart
import 'package:mockito/annotations.dart';
import 'package:my_app/repositories/user_repository.dart';
import 'package:my_app/services/api_client.dart';
import 'package:dio/dio.dart';

@GenerateMocks([
  UserRepository,
  ApiClient,
  Dio,
])
void main() {}
```

```bash
# モックを生成
dart run build_runner build
```

### モックの使用例

```dart
test('ログイン成功時にユーザー情報を返す', () async {
  final mockRepository = MockUserRepository();

  when(mockRepository.login('test@example.com', 'password'))
      .thenAnswer((_) async => User(id: '1', name: 'Test User'));

  final result = await mockRepository.login('test@example.com', 'password');

  expect(result.name, 'Test User');
  verify(mockRepository.login('test@example.com', 'password')).called(1);
});
```

### HTTP通信のモック

```dart
test('API呼び出しが成功する', () async {
  final mockDio = MockDio();

  when(mockDio.get('/users')).thenAnswer(
    (_) async => Response(
      data: [
        {'id': '1', 'name': 'Alice'},
        {'id': '2', 'name': 'Bob'},
      ],
      statusCode: 200,
      requestOptions: RequestOptions(path: '/users'),
    ),
  );

  final response = await mockDio.get('/users');

  expect(response.statusCode, 200);
  expect(response.data.length, 2);
});
```

## テストカバレッジ

### カバレッジの取得

```bash
# カバレッジ付きでテスト実行
flutter test --coverage

# HTMLレポート生成（要：lcov）
genhtml coverage/lcov.info -o coverage/html

# ブラウザで確認
open coverage/html/index.html
```

### カバレッジ除外設定

```yaml
# analysis_options.yaml
analyzer:
  exclude:
    - "**/*.g.dart"
    - "**/*.freezed.dart"
    - "**/generated/**"
    - "test/**"
```

### カバレッジ目標

| カテゴリ | 目標カバレッジ |
|---------|--------------|
| ビジネスロジック（UseCases） | 80%以上 |
| Repository | 70%以上 |
| Provider | 60%以上 |
| UI（Widget） | 40%以上 |

## ベストプラクティス

### 1. AAA パターン（Arrange-Act-Assert）

```dart
test('ユーザーを追加できる', () {
  // Arrange: テストデータの準備
  final repository = UserRepository();
  final user = User(id: '1', name: 'Alice');

  // Act: テスト対象の実行
  final result = repository.addUser(user);

  // Assert: 結果の検証
  expect(result, true);
});
```

### 2. Given-When-Then

```dart
test('Given logged in, When logout, Then user becomes null', () async {
  // Given
  final container = ProviderContainer();
  await container.read(authNotifierProvider.notifier).login('test', 'pass');

  // When
  await container.read(authNotifierProvider.notifier).logout();

  // Then
  expect(container.read(authNotifierProvider).value, null);
});
```

### 3. テストの独立性

```dart
// NG: テスト間で状態を共有
late UserRepository repository;

setUp(() {
  repository = UserRepository();
});

// OK: 各テストで新しいインスタンス
test('test1', () {
  final repository = UserRepository();
  // ...
});

test('test2', () {
  final repository = UserRepository();
  // ...
});
```

### 4. 意味のあるテスト名

```dart
// NG
test('test1', () { ... });

// OK
test('無効なメールアドレスの場合falseを返す', () { ... });
```

## Golden Tests（ビジュアルリグレッションテスト）

```dart
testWidgets('UserCard golden test', (tester) async {
  await tester.pumpWidget(
    MaterialApp(
      home: UserCard(
        user: User(id: '1', name: 'Alice'),
      ),
    ),
  );

  await expectLater(
    find.byType(UserCard),
    matchesGoldenFile('goldens/user_card.png'),
  );
});
```

```bash
# Golden fileを更新
flutter test --update-goldens
```

## CI/CDでのテスト実行

### GitHub Actions例

```yaml
# .github/workflows/test.yml
name: Test

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.27.0'

      - name: Install dependencies
        run: flutter pub get

      - name: Run tests
        run: flutter test --coverage

      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage/lcov.info
```

## AI開発時の注意点

### AIが生成しがちな問題のあるテスト

#### 1. 実装に依存しすぎるテスト

**NG**:
```dart
test('_privateMethodが呼ばれる', () {
  // プライベートメソッドのテスト
});
```

**OK**:
```dart
test('公開APIが期待通りに動作する', () {
  // 公開インターフェースのテスト
});
```

#### 2. 不安定なテスト

**NG**:
```dart
test('時刻に依存', () {
  final now = DateTime.now();
  // テスト実行タイミングで結果が変わる
});
```

**OK**:
```dart
test('モックした時刻で検証', () {
  final fixedTime = DateTime(2025, 1, 1);
  // 時刻をモック
});
```

## チェックリスト

- [ ] ユニットテストを書く（ビジネスロジック）
- [ ] Widgetテストを書く（UI）
- [ ] インテグレーションテストを書く（主要フロー）
- [ ] モックを適切に使用
- [ ] テストカバレッジ70%以上を目指す
- [ ] CI/CDでテスト自動実行

## 次のステップ

テスト戦略を理解したら、[第8章: パフォーマンス最適化](08_performance.md)でアプリの高速化を学びましょう。

---

**AI開発のヒント**:
生成AIに「Riverpodのプロバイダーをテストするコードを書いて」と依頼する際は、この章の「Riverpod Providerのテスト」セクションを一緒に渡すと正確なテストコードを生成してくれます。
