# 第4章: 主要ライブラリとパッケージ（2025年推奨）

## この章で学べること

- 2025年に使うべき推奨パッケージ一覧
- 非推奨・メンテナンス停止パッケージ
- バージョン互換性の注意点
- パッケージ選定基準

## カテゴリ別推奨パッケージ

### 状態管理

| パッケージ | バージョン | 推奨度 | 備考 |
|-----------|----------|--------|------|
| flutter_riverpod | 2.5+ | ⭐⭐⭐⭐⭐ | 2025年の標準 |
| riverpod_annotation | 2.5+ | ⭐⭐⭐⭐⭐ | コード生成必須 |
| flutter_bloc | 8.1+ | ⭐⭐⭐⭐ | 大規模・複雑なアプリ向け |
| provider | 6.1+ | ⭐⭐ | メンテナンスモード |
| get | - | ❌ | 非推奨 |

```yaml
dependencies:
  flutter_riverpod: ^2.5.0
  riverpod_annotation: ^2.5.0

dev_dependencies:
  riverpod_generator: ^2.5.0
```

### HTTP通信

| パッケージ | バージョン | 推奨度 | 備考 |
|-----------|----------|--------|------|
| dio | 5.4+ | ⭐⭐⭐⭐⭐ | インターセプター、リトライなど機能豊富 |
| http | 1.2+ | ⭐⭐⭐ | シンプルな用途向け |
| retrofit | 4.1+ | ⭐⭐⭐⭐ | 型安全なAPI定義 |
| graphql_flutter | 5.1+ | ⭐⭐⭐⭐ | GraphQL専用 |

```yaml
dependencies:
  dio: ^5.4.0
  retrofit: ^4.1.0
  json_annotation: ^4.9.0

dev_dependencies:
  retrofit_generator: ^8.1.0
  json_serializable: ^6.8.0
```

**実装例（Retrofit + Dio）**:
```dart
import 'package:dio/dio.dart';
import 'package:retrofit/retrofit.dart';

part 'api_client.g.dart';

@RestApi(baseUrl: 'https://api.example.com')
abstract class ApiClient {
  factory ApiClient(Dio dio, {String baseUrl}) = _ApiClient;

  @GET('/users')
  Future<List<User>> getUsers();

  @POST('/users')
  Future<User> createUser(@Body() User user);
}
```

### ナビゲーション

| パッケージ | バージョン | 推奨度 | 備考 |
|-----------|----------|--------|------|
| go_router | 15.0+ | ⭐⭐⭐⭐⭐ | デファクトスタンダード |
| auto_route | 9.0+ | ⭐⭐⭐⭐ | より高機能な型安全ルーティング |

```yaml
dependencies:
  go_router: ^15.0.0

# または
dependencies:
  auto_route: ^9.0.0

dev_dependencies:
  auto_route_generator: ^9.0.0
```

**go_router基本例**:
```dart
final router = GoRouter(
  routes: [
    GoRoute(
      path: '/',
      builder: (context, state) => const HomeScreen(),
      routes: [
        GoRoute(
          path: 'details/:id',
          builder: (context, state) {
            final id = state.pathParameters['id']!;
            return DetailsScreen(id: id);
          },
        ),
      ],
    ),
  ],
);
```

### データ永続化

| パッケージ | バージョン | 推奨度 | 用途 |
|-----------|----------|--------|------|
| drift | 2.18+ | ⭐⭐⭐⭐⭐ | SQLite（型安全） |
| isar | 3.1+ | ⭐⭐⭐⭐⭐ | NoSQL（高速） |
| hive | 2.2+ | ⭐⭐⭐ | 軽量key-value |
| shared_preferences | 2.2+ | ⭐⭐⭐ | 設定値のみ |
| sqflite | - | ⭐⭐ | 低レベル（Drift推奨） |

**Drift（推奨）**:
```yaml
dependencies:
  drift: ^2.18.0
  sqlite3_flutter_libs: ^0.5.0
  path_provider: ^2.1.0
  path: ^1.9.0

dev_dependencies:
  drift_dev: ^2.18.0
  build_runner: ^2.4.0
```

**Isar（高速NoSQL）**:
```yaml
dependencies:
  isar: ^3.1.0
  isar_flutter_libs: ^3.1.0
  path_provider: ^2.1.0

dev_dependencies:
  isar_generator: ^3.1.0
  build_runner: ^2.4.0
```

### コード生成

| パッケージ | バージョン | 推奨度 | 用途 |
|-----------|----------|--------|------|
| freezed | 2.5+ | ⭐⭐⭐⭐⭐ | イミュータブルクラス |
| json_serializable | 6.8+ | ⭐⭐⭐⭐⭐ | JSON変換 |
| build_runner | 2.4+ | ⭐⭐⭐⭐⭐ | コード生成実行 |

```yaml
dependencies:
  freezed_annotation: ^2.5.0
  json_annotation: ^4.9.0

dev_dependencies:
  freezed: ^2.5.0
  json_serializable: ^6.8.0
  build_runner: ^2.4.0
```

**Freezedの使用例**:
```dart
import 'package:freezed_annotation/freezed_annotation.dart';

part 'user.freezed.dart';
part 'user.g.dart';

@freezed
class User with _$User {
  const factory User({
    required String id,
    required String name,
    required String email,
    DateTime? createdAt,
  }) = _User;

  factory User.fromJson(Map<String, dynamic> json) => _$UserFromJson(json);
}
```

### UI・デザイン

| パッケージ | バージョン | 推奨度 | 用途 |
|-----------|----------|--------|------|
| flutter_hooks | 0.20+ | ⭐⭐⭐⭐ | React Hooks風の状態管理 |
| gap | 3.0+ | ⭐⭐⭐⭐⭐ | スペース挿入（SizedBox代替） |
| flutter_svg | 2.0+ | ⭐⭐⭐⭐⭐ | SVG画像 |
| cached_network_image | 3.3+ | ⭐⭐⭐⭐⭐ | 画像キャッシュ |
| shimmer | 3.0+ | ⭐⭐⭐⭐ | スケルトンローディング |
| flutter_animate | 4.5+ | ⭐⭐⭐⭐ | アニメーション |

```yaml
dependencies:
  flutter_hooks: ^0.20.0
  gap: ^3.0.0
  flutter_svg: ^2.0.0
  cached_network_image: ^3.3.0
  shimmer: ^3.0.0
  flutter_animate: ^4.5.0
```

**Gap使用例**（SizedBoxより簡潔）:
```dart
Column(
  children: [
    Text('Title'),
    const Gap(16),  // SizedBox(height: 16) の代わり
    Text('Content'),
  ],
)
```

### ユーティリティ

| パッケージ | バージョン | 推奨度 | 用途 |
|-----------|----------|--------|------|
| intl | 0.19+ | ⭐⭐⭐⭐⭐ | 国際化・日付フォーマット |
| url_launcher | 6.3+ | ⭐⭐⭐⭐⭐ | URL・電話・メール起動 |
| package_info_plus | 8.0+ | ⭐⭐⭐⭐ | アプリ情報取得 |
| device_info_plus | 10.1+ | ⭐⭐⭐⭐ | デバイス情報 |
| permission_handler | 11.3+ | ⭐⭐⭐⭐⭐ | 権限管理 |
| connectivity_plus | 6.0+ | ⭐⭐⭐⭐ | ネットワーク状態 |

```yaml
dependencies:
  intl: ^0.19.0
  url_launcher: ^6.3.0
  package_info_plus: ^8.0.0
  permission_handler: ^11.3.0
```

### フォーム・バリデーション

| パッケージ | バージョン | 推奨度 | 用途 |
|-----------|----------|--------|------|
| flutter_form_builder | 9.4+ | ⭐⭐⭐⭐⭐ | フォーム構築 |
| form_builder_validators | 11.0+ | ⭐⭐⭐⭐⭐ | バリデーション |
| reactive_forms | 17.0+ | ⭐⭐⭐⭐ | リアクティブフォーム |

```yaml
dependencies:
  flutter_form_builder: ^9.4.0
  form_builder_validators: ^11.0.0
```

**使用例**:
```dart
FormBuilderTextField(
  name: 'email',
  decoration: const InputDecoration(labelText: 'Email'),
  validator: FormBuilderValidators.compose([
    FormBuilderValidators.required(),
    FormBuilderValidators.email(),
  ]),
)
```

### Firebase

| パッケージ | バージョン | 推奨度 | 用途 |
|-----------|----------|--------|------|
| firebase_core | 3.3+ | ⭐⭐⭐⭐⭐ | Firebase初期化 |
| firebase_auth | 5.1+ | ⭐⭐⭐⭐⭐ | 認証 |
| cloud_firestore | 5.2+ | ⭐⭐⭐⭐⭐ | NoSQLデータベース |
| firebase_storage | 12.1+ | ⭐⭐⭐⭐ | ファイルストレージ |
| firebase_messaging | 15.0+ | ⭐⭐⭐⭐⭐ | プッシュ通知 |
| firebase_crashlytics | 4.0+ | ⭐⭐⭐⭐⭐ | クラッシュレポート |

```yaml
dependencies:
  firebase_core: ^3.3.0
  firebase_auth: ^5.1.0
  cloud_firestore: ^5.2.0
  firebase_messaging: ^15.0.0
```

### テスト

| パッケージ | バージョン | 推奨度 | 用途 |
|-----------|----------|--------|------|
| mockito | 5.4+ | ⭐⭐⭐⭐⭐ | モック作成 |
| mocktail | 1.0+ | ⭐⭐⭐⭐ | null-safety対応モック |
| flutter_test | SDK | ⭐⭐⭐⭐⭐ | 標準テスト |
| integration_test | SDK | ⭐⭐⭐⭐⭐ | E2Eテスト |

```yaml
dev_dependencies:
  flutter_test:
    sdk: flutter
  mockito: ^5.4.0
  build_runner: ^2.4.0
```

### アニメーション・UI強化

| パッケージ | バージョン | 推奨度 | 用途 |
|-----------|----------|--------|------|
| lottie | 3.1+ | ⭐⭐⭐⭐⭐ | Lottieアニメーション |
| rive | 0.13+ | ⭐⭐⭐⭐ | Riveアニメーション |
| flutter_staggered_grid_view | 0.7+ | ⭐⭐⭐⭐ | グリッドレイアウト |

## 非推奨・避けるべきパッケージ

### 完全に避けるべき

| パッケージ | 理由 | 代替 |
|-----------|------|------|
| get (GetX) | アンチパターン、過度に魔法的 | Riverpod + go_router |
| sqflite（直接使用） | 低レベルすぎる | Drift or Isar |
| shared_preferences（大量データ） | パフォーマンス問題 | Drift, Isar, Hive |

### メンテナンスモード

| パッケージ | 状態 | 代替 |
|-----------|------|------|
| provider | メンテナンスモード | Riverpod |
| moor | 名称変更 | Drift（同じもの） |
| pedantic | 廃止 | flutter_lints |

## バージョン互換性マトリクス

### Flutter 3.27 対応状況（2025年10月）

| パッケージ | Flutter 3.24 | Flutter 3.27 | 備考 |
|-----------|-------------|-------------|------|
| riverpod | ✅ | ✅ | 完全対応 |
| dio | ✅ | ✅ | 完全対応 |
| go_router | ✅ | ✅ | 完全対応 |
| drift | ✅ | ✅ | 完全対応 |
| freezed | ✅ | ✅ | 完全対応 |
| firebase_* | ✅ | ✅ | 完全対応 |

**注意**: Flutter更新時は`flutter pub upgrade`を実行してパッケージも更新すること

## パッケージ選定基準

### チェックリスト

#### 1. メンテナンス状況
- [ ] 最終更新が6ヶ月以内
- [ ] GitHubのIssue対応が活発
- [ ] Null Safety対応済み

#### 2. 人気度
- [ ] pub.devのLikes > 500
- [ ] Pub Points > 120
- [ ] GitHub Stars > 1000（目安）

#### 3. 品質
- [ ] Example/サンプルコードが充実
- [ ] ドキュメントが整備されている
- [ ] CI/CDが設定されている
- [ ] テストカバレッジが高い

#### 4. エコシステム
- [ ] 有名企業・コミュニティが支持
- [ ] 関連パッケージが充実
- [ ] StackOverflowに情報が多い

### 判断フロー

```
パッケージが必要
    ↓
公式パッケージが存在？
    YES → 公式を使用
    NO  ↓
    ↓
複数の選択肢がある？
    YES → 上記選定基準で評価
    NO  ↓
    ↓
メンテナンス状況は良好？
    YES → 採用
    NO  → 自作を検討
```

## 推奨pubspec.yaml（2025年版）

### 中規模アプリ向けテンプレート

```yaml
name: my_app
description: A Flutter application
publish_to: 'none'
version: 1.0.0+1

environment:
  sdk: '>=3.6.0 <4.0.0'
  flutter: '>=3.27.0'

dependencies:
  flutter:
    sdk: flutter

  # 状態管理
  flutter_riverpod: ^2.5.0
  riverpod_annotation: ^2.5.0

  # ナビゲーション
  go_router: ^15.0.0

  # HTTP通信
  dio: ^5.4.0
  retrofit: ^4.1.0
  json_annotation: ^4.9.0

  # データ永続化
  drift: ^2.18.0
  sqlite3_flutter_libs: ^0.5.0
  path_provider: ^2.1.0

  # コード生成
  freezed_annotation: ^2.5.0

  # UI
  flutter_hooks: ^0.20.0
  gap: ^3.0.0
  cached_network_image: ^3.3.0
  flutter_svg: ^2.0.0

  # ユーティリティ
  intl: ^0.19.0
  url_launcher: ^6.3.0
  permission_handler: ^11.3.0

dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^4.0.0

  # コード生成
  build_runner: ^2.4.0
  riverpod_generator: ^2.5.0
  riverpod_lint: ^2.3.0
  custom_lint: ^0.6.0
  freezed: ^2.5.0
  json_serializable: ^6.8.0
  retrofit_generator: ^8.1.0
  drift_dev: ^2.18.0

  # テスト
  mockito: ^5.4.0

flutter:
  uses-material-design: true
  assets:
    - assets/images/
    - assets/icons/
```

## パッケージ更新戦略

### 定期更新

```bash
# 更新可能なパッケージを確認
flutter pub outdated

# マイナーバージョンまで更新
flutter pub upgrade --minor-versions

# メジャーバージョンも含めて更新
flutter pub upgrade --major-versions
```

### 慎重に更新すべきパッケージ

- **状態管理**（riverpod, bloc）: 破壊的変更の可能性
- **ナビゲーション**（go_router）: ルーティング定義の変更
- **データベース**（drift, isar）: マイグレーション必要

**推奨**: メジャーバージョンアップ時は専用ブランチで検証

## AI開発時の注意点

### AIが古いパッケージを提案する例

| AIの提案 | 問題 | 正しい選択 |
|---------|------|----------|
| `provider` | メンテナンスモード | `flutter_riverpod` |
| `sqflite`直接使用 | 低レベル | `drift` |
| `shared_preferences`（大規模データ） | 用途外 | `drift` or `isar` |

### プロンプト例

**NG**:
```
「Flutter でTodoアプリを作って」
→ AIが古いパッケージを使う可能性
```

**OK**:
```
「Flutter 3.27、Riverpod 3.x、Drift を使ってTodoアプリを作って。
第4章のパッケージ推奨リストを参照してください。」
```

## 次のステップ

パッケージの選定ができたら、[第5章: ナビゲーションとルーティング](05_navigation.md)で go_router の実装を学びましょう。

---

**この章の活用法**:
新規プロジェクト開始時や、パッケージ追加を検討する際に、このページを参照して最新の推奨パッケージを確認してください。
