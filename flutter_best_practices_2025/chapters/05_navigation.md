# 第5章: ナビゲーションとルーティング

## この章で学べること

- go_router 15.x+ の使い方
- 型安全なルーティング
- ディープリンク対応
- Riverpodとの統合

## go_routerが推奨される理由

### 2025年のルーティングソリューション

| ソリューション | 推奨度 | 備考 |
|--------------|-------|------|
| go_router | ⭐⭐⭐⭐⭐ | デファクトスタンダード |
| auto_route | ⭐⭐⭐⭐ | より高機能 |
| Navigator 2.0（直接） | ⭐⭐ | 低レベルすぎる |
| Navigator 1.0 | ❌ | レガシー |

### go_routerの利点

1. **宣言的ルーティング**: URLパスベースで直感的
2. **ディープリンク対応**: Web/モバイルで統一
3. **型安全**: パラメータの型チェック
4. **状態管理統合**: Riverpodと相性が良い

## 基本的なセットアップ

### pubspec.yaml

```yaml
dependencies:
  go_router: ^15.0.0
  flutter_riverpod: ^2.5.0
```

### 基本的なルーター定義

```dart
import 'package:go_router/go_router.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

final routerProvider = Provider<GoRouter>((ref) {
  return GoRouter(
    initialLocation: '/',
    routes: [
      GoRoute(
        path: '/',
        name: 'home',
        builder: (context, state) => const HomeScreen(),
      ),
      GoRoute(
        path: '/profile',
        name: 'profile',
        builder: (context, state) => const ProfileScreen(),
      ),
    ],
  );
});

// main.dart
class MyApp extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final router = ref.watch(routerProvider);

    return MaterialApp.router(
      routerConfig: router,
      title: 'My App',
    );
  }
}
```

## パラメータ付きルーティング

### パスパラメータ

```dart
GoRoute(
  path: '/users/:userId',
  name: 'userDetail',
  builder: (context, state) {
    final userId = state.pathParameters['userId']!;
    return UserDetailScreen(userId: userId);
  },
),
```

**遷移方法**:
```dart
// パスで指定
context.go('/users/123');

// 名前付きルートで指定
context.goNamed('userDetail', pathParameters: {'userId': '123'});
```

### クエリパラメータ

```dart
GoRoute(
  path: '/search',
  name: 'search',
  builder: (context, state) {
    final query = state.uri.queryParameters['q'] ?? '';
    final filter = state.uri.queryParameters['filter'];
    return SearchScreen(query: query, filter: filter);
  },
),
```

**遷移方法**:
```dart
context.goNamed(
  'search',
  queryParameters: {
    'q': 'flutter',
    'filter': 'recent',
  },
);
```

### Extra（オブジェクト渡し）

```dart
GoRoute(
  path: '/edit-post',
  name: 'editPost',
  builder: (context, state) {
    final post = state.extra as Post;
    return EditPostScreen(post: post);
  },
),
```

**遷移方法**:
```dart
context.goNamed(
  'editPost',
  extra: Post(id: '1', title: 'My Post'),
);
```

## ネストされたルート

### 親子関係の定義

```dart
GoRoute(
  path: '/',
  name: 'home',
  builder: (context, state) => const HomeScreen(),
  routes: [
    GoRoute(
      path: 'settings',
      name: 'settings',
      builder: (context, state) => const SettingsScreen(),
      routes: [
        GoRoute(
          path: 'account',
          name: 'account',
          builder: (context, state) => const AccountScreen(),
        ),
      ],
    ),
  ],
),
```

**URL構造**:
- `/` → HomeScreen
- `/settings` → SettingsScreen
- `/settings/account` → AccountScreen

## ShellRoute（タブナビゲーション）

### ボトムナビゲーション実装

```dart
final routerProvider = Provider<GoRouter>((ref) {
  return GoRouter(
    initialLocation: '/home',
    routes: [
      ShellRoute(
        builder: (context, state, child) {
          return ScaffoldWithNavBar(child: child);
        },
        routes: [
          GoRoute(
            path: '/home',
            name: 'home',
            pageBuilder: (context, state) => const NoTransitionPage(
              child: HomeScreen(),
            ),
          ),
          GoRoute(
            path: '/search',
            name: 'search',
            pageBuilder: (context, state) => const NoTransitionPage(
              child: SearchScreen(),
            ),
          ),
          GoRoute(
            path: '/profile',
            name: 'profile',
            pageBuilder: (context, state) => const NoTransitionPage(
              child: ProfileScreen(),
            ),
          ),
        ],
      ),
    ],
  );
});

class ScaffoldWithNavBar extends StatelessWidget {
  final Widget child;

  const ScaffoldWithNavBar({required this.child});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: child,
      bottomNavigationBar: BottomNavigationBar(
        currentIndex: _calculateSelectedIndex(context),
        onTap: (index) => _onItemTapped(index, context),
        items: const [
          BottomNavigationBarItem(icon: Icon(Icons.home), label: 'Home'),
          BottomNavigationBarItem(icon: Icon(Icons.search), label: 'Search'),
          BottomNavigationBarItem(icon: Icon(Icons.person), label: 'Profile'),
        ],
      ),
    );
  }

  int _calculateSelectedIndex(BuildContext context) {
    final location = GoRouterState.of(context).uri.toString();
    if (location.startsWith('/home')) return 0;
    if (location.startsWith('/search')) return 1;
    if (location.startsWith('/profile')) return 2;
    return 0;
  }

  void _onItemTapped(int index, BuildContext context) {
    switch (index) {
      case 0:
        context.go('/home');
        break;
      case 1:
        context.go('/search');
        break;
      case 2:
        context.go('/profile');
        break;
    }
  }
}
```

## リダイレクトと認証ガード

### 認証状態によるリダイレクト

```dart
final routerProvider = Provider<GoRouter>((ref) {
  final authState = ref.watch(authNotifierProvider);

  return GoRouter(
    initialLocation: '/',
    redirect: (context, state) {
      final isLoggedIn = authState.value != null;
      final isLoggingIn = state.matchedLocation == '/login';

      // ログインしていない場合、ログインページへ
      if (!isLoggedIn && !isLoggingIn) {
        return '/login';
      }

      // ログイン済みでログインページにいる場合、ホームへ
      if (isLoggedIn && isLoggingIn) {
        return '/';
      }

      // それ以外は現在のページを維持
      return null;
    },
    refreshListenable: GoRouterRefreshStream(
      ref.read(authNotifierProvider.notifier).stream,
    ),
    routes: [
      GoRoute(
        path: '/login',
        builder: (context, state) => const LoginScreen(),
      ),
      GoRoute(
        path: '/',
        builder: (context, state) => const HomeScreen(),
      ),
    ],
  );
});

// リフレッシュ用のヘルパークラス
class GoRouterRefreshStream extends ChangeNotifier {
  GoRouterRefreshStream(Stream<dynamic> stream) {
    _subscription = stream.listen((_) {
      notifyListeners();
    });
  }

  late final StreamSubscription<dynamic> _subscription;

  @override
  void dispose() {
    _subscription.cancel();
    super.dispose();
  }
}
```

## ディープリンク対応

### Android設定

```xml
<!-- android/app/src/main/AndroidManifest.xml -->
<manifest>
  <application>
    <activity>
      <!-- 既存の設定... -->

      <!-- ディープリンク -->
      <intent-filter>
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <data
          android:scheme="myapp"
          android:host="example.com" />
      </intent-filter>
    </activity>
  </application>
</manifest>
```

### iOS設定

```xml
<!-- ios/Runner/Info.plist -->
<key>CFBundleURLTypes</key>
<array>
  <dict>
    <key>CFBundleTypeRole</key>
    <string>Editor</string>
    <key>CFBundleURLSchemes</key>
    <array>
      <string>myapp</string>
    </array>
  </dict>
</array>
```

### ディープリンクの処理

```dart
GoRoute(
  path: '/posts/:postId',
  name: 'postDetail',
  builder: (context, state) {
    final postId = state.pathParameters['postId']!;
    return PostDetailScreen(postId: postId);
  },
),
```

**動作**:
- `myapp://example.com/posts/123` → PostDetailScreen(postId: '123')
- Web: `https://example.com/posts/123`も同じように動作

## エラーハンドリング

### 404ページ

```dart
GoRouter(
  // ...
  errorBuilder: (context, state) => ErrorScreen(
    error: state.error,
  ),
  routes: [
    // ...
  ],
)

class ErrorScreen extends StatelessWidget {
  final Exception? error;

  const ErrorScreen({this.error});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Error')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Icon(Icons.error_outline, size: 48),
            const SizedBox(height: 16),
            Text(error?.toString() ?? 'Page not found'),
            const SizedBox(height: 16),
            ElevatedButton(
              onPressed: () => context.go('/'),
              child: const Text('Go Home'),
            ),
          ],
        ),
      ),
    );
  }
}
```

## アニメーション・トランジション

### カスタムトランジション

```dart
GoRoute(
  path: '/details',
  pageBuilder: (context, state) => CustomTransitionPage(
    key: state.pageKey,
    child: const DetailsScreen(),
    transitionsBuilder: (context, animation, secondaryAnimation, child) {
      return FadeTransition(
        opacity: animation,
        child: child,
      );
    },
  ),
),
```

### トランジションなし

```dart
GoRoute(
  path: '/tab',
  pageBuilder: (context, state) => NoTransitionPage(
    key: state.pageKey,
    child: const TabScreen(),
  ),
),
```

## 型安全なルーティング（コード生成）

### go_router_builder の使用

```yaml
dependencies:
  go_router: ^15.0.0

dev_dependencies:
  go_router_builder: ^2.7.0
  build_runner: ^2.4.0
```

```dart
import 'package:go_router/go_router.dart';

part 'routes.g.dart';

@TypedGoRoute<HomeRoute>(
  path: '/',
  routes: [
    TypedGoRoute<UserDetailRoute>(
      path: 'users/:userId',
    ),
  ],
)
class HomeRoute extends GoRouteData {
  @override
  Widget build(BuildContext context, GoRouterState state) {
    return const HomeScreen();
  }
}

class UserDetailRoute extends GoRouteData {
  final String userId;

  const UserDetailRoute({required this.userId});

  @override
  Widget build(BuildContext context, GoRouterState state) {
    return UserDetailScreen(userId: userId);
  }
}

// 使用
const UserDetailRoute(userId: '123').go(context);
```

## AI開発時の注意点

### AIが間違いやすいパターン

#### 1. Navigator 1.0の古い記法

**NG（古い記法）**:
```dart
Navigator.of(context).push(
  MaterialPageRoute(builder: (context) => DetailScreen()),
);
```

**OK（2025年推奨）**:
```dart
context.go('/details');
// または
context.goNamed('details');
```

#### 2. context.go vs context.push

```dart
// go: スタックをクリアして遷移（戻れない）
context.go('/home');

// push: スタックに追加（戻れる）
context.push('/details');
```

#### 3. パラメータの型安全性

**NG**:
```dart
final userId = state.pathParameters['userId']; // String?
return UserDetailScreen(userId: userId!); // null例外の可能性
```

**OK**:
```dart
final userId = state.pathParameters['userId'];
if (userId == null) {
  return const ErrorScreen();
}
return UserDetailScreen(userId: userId);
```

## ベストプラクティス

### ルート名を定数化

```dart
class AppRoutes {
  static const home = '/';
  static const profile = '/profile';
  static const userDetail = '/users/:userId';
}

// 使用
context.go(AppRoutes.home);
```

### ルート定義の分割

```dart
// routes/auth_routes.dart
final authRoutes = [
  GoRoute(path: '/login', ...),
  GoRoute(path: '/signup', ...),
];

// routes/main_routes.dart
final mainRoutes = [
  GoRoute(path: '/', ...),
  GoRoute(path: '/profile', ...),
];

// router.dart
final router = GoRouter(
  routes: [
    ...authRoutes,
    ...mainRoutes,
  ],
);
```

## チェックリスト

- [ ] `go_router`をインストール
- [ ] `MaterialApp.router`を使用
- [ ] すべてのルートを定義
- [ ] 認証ガードを実装（必要に応じて）
- [ ] ディープリンク設定（必要に応じて）
- [ ] エラーページを作成
- [ ] ルート名を定数化

## 次のステップ

ナビゲーションの実装ができたら、[第6章: データ永続化とAPI連携](06_data_persistence.md)でバックエンドとの通信を学びましょう。

---

**AI開発のヒント**:
生成AIに「go_router 15.xで認証ガード付きのルーティングを実装して」と依頼する際は、この章の「リダイレクトと認証ガード」セクションを一緒に渡すと正確な実装をしてくれます。
