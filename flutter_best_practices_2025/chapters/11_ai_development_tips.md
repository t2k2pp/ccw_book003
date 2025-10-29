# 第11章: 生成AI開発時の注意点とベストプラクティス

## この章で学べること

- AIが間違いやすい具体的なポイント
- 効果的なプロンプトの書き方
- コードレビューのチェックリスト
- バージョン差異の確認方法

## AIが間違いやすい主要ポイント（2025年版）

### 1. 状態管理

#### 古いProvider記法を生成する

**AIの出力（❌ 非推奨）**:
```dart
final counterProvider = StateProvider<int>((ref) => 0);
```

**正しい記法（✅ 2025年推奨）**:
```dart
import 'package:riverpod_annotation/riverpod_annotation.dart';

part 'counter_provider.g.dart';

@riverpod
class Counter extends _$Counter {
  @override
  int build() => 0;

  void increment() => state++;
}
```

**プロンプト例**:
```
Riverpod 3.xのコード生成（@riverpod）を使って、
カウンターのプロバイダーを実装してください。
StateProviderは使わないでください。
```

### 2. ナビゲーション

#### Navigator 1.0の古い記法

**AIの出力（❌ レガシー）**:
```dart
Navigator.of(context).push(
  MaterialPageRoute(
    builder: (context) => DetailScreen(),
  ),
);
```

**正しい記法（✅ 2025年推奨）**:
```dart
// go_router 15.x
context.push('/detail');
// または
context.goNamed('detail', pathParameters: {'id': '123'});
```

**プロンプト例**:
```
go_router 15.xを使って、ホーム画面から詳細画面への
遷移を実装してください。Navigator.pushは使わないでください。
```

### 3. データベース

#### sqfliteを直接使う

**AIの出力（❌ 低レベル）**:
```dart
final db = await openDatabase('my_db.db');
await db.insert('users', user.toMap());
```

**正しい記法（✅ 2025年推奨）**:
```dart
// Drift 2.18+
@DriftDatabase(tables: [Users])
class AppDatabase extends _$AppDatabase {
  // ...
}

await database.into(users).insert(user);
```

**プロンプト例**:
```
Drift 2.18を使って、ユーザーテーブルを定義し、
CRUD操作を実装してください。sqfliteは使わないでください。
```

### 4. HTTP通信

#### http パッケージの非効率な使用

**AIの出力（❌ 機能不足）**:
```dart
import 'package:http/http.dart' as http;

final response = await http.get(Uri.parse('https://api.example.com/users'));
```

**正しい記法（✅ 2025年推奨）**:
```dart
// Dio 5.4+ with Retrofit 4.1+
@RestApi(baseUrl: 'https://api.example.com')
abstract class ApiClient {
  factory ApiClient(Dio dio) = _ApiClient;

  @GET('/users')
  Future<List<User>> getUsers();
}
```

**プロンプト例**:
```
Dio 5.4とRetrofit 4.1を使って、
ユーザー一覧を取得するAPIクライアントを実装してください。
インターセプターでエラーハンドリングも行ってください。
```

### 5. パッケージバージョン

#### 古いパッケージバージョン

**AIの出力（❌ 古い）**:
```yaml
dependencies:
  provider: ^6.0.0
  shared_preferences: ^2.0.0
```

**正しい記法（✅ 2025年推奨）**:
```yaml
dependencies:
  flutter_riverpod: ^2.5.0
  riverpod_annotation: ^2.5.0
  drift: ^2.18.0
```

**プロンプト例**:
```
Flutter 3.27、Dart 3.6環境で、
2025年10月時点の最新パッケージを使って実装してください。
第4章のパッケージ推奨リストを参照してください。
```

## 効果的なプロンプトの書き方

### プロンプトテンプレート

#### 基本形

```
【環境】
- Flutter: 3.27.x
- Dart: 3.6.x
- 状態管理: Riverpod 3.x（コード生成）

【要件】
[具体的にやりたいこと]

【制約】
- [使ってはいけないパッケージやパターン]
- [守るべきアーキテクチャやルール]

【参考】
[該当する章の内容やコード例]
```

#### 実例1: CRUD機能の実装

```
【環境】
- Flutter 3.27
- Riverpod 3.x（@riverpod）
- Drift 2.18
- go_router 15.x

【要件】
Todoアプリの以下の機能を実装してください：
1. Todo一覧の表示
2. Todo追加機能
3. Todo完了トグル
4. Todo削除機能

【制約】
- StateProviderやStateNotifierProviderは使わない
- sqfliteは使わず、Driftを使う
- Clean Architectureに従う（Domain, Data, Presentation層）

【参考】
第2章のClean Architectureパターン
第3章のRiverpod 3.x実装例
第6章のDrift実装例
を参照してください。
```

#### 実例2: API連携

```
【環境】
- Flutter 3.27
- Dio 5.4 + Retrofit 4.1
- Riverpod 3.x
- Freezed 2.5（JSONシリアライゼーション）

【要件】
JSONPlaceholder API（https://jsonplaceholder.typicode.com）から
投稿一覧を取得し、表示する機能を実装してください。

【制約】
- http パッケージは使わず、Dio + Retrofit を使う
- エラーハンドリングを適切に行う
- ローディング状態を表示する
- オフライン時はキャッシュから表示

【参考】
第6章の「HTTP通信」セクション
第6章の「オフライン対応」セクション
を参照してください。
```

### プロンプトのベストプラクティス

#### ✅ DO

1. **バージョンを明示する**
   ```
   ❌ Riverpodを使って実装してください
   ✅ Riverpod 3.xのコード生成（@riverpod）を使って実装してください
   ```

2. **使ってはいけないものを明示する**
   ```
   ❌ データベースを実装してください
   ✅ Drift 2.18を使ってデータベースを実装してください。sqfliteは使わないでください。
   ```

3. **参照すべきドキュメントを指定する**
   ```
   ❌ 状態管理を実装してください
   ✅ 第3章のRiverpod実装例を参考に、状態管理を実装してください
   ```

4. **期待する出力形式を指定する**
   ```
   ✅ 以下のファイル構成で実装してください：
   - features/todo/domain/entities/todo.dart
   - features/todo/data/repositories/todo_repository_impl.dart
   - features/todo/presentation/providers/todo_provider.dart
   ```

#### ❌ DON'T

1. **曖昧な指示**
   ```
   ❌ いい感じに実装してください
   ❌ 最適化してください
   ```

2. **バージョン指定なし**
   ```
   ❌ Flutterで実装してください
   ```

3. **前提知識がない指示**
   ```
   ❌ 既存のコードを修正してください（どのコードか不明）
   ```

## コードレビューチェックリスト

### AIが生成したコードの確認項目

#### 1. パッケージバージョン

```dart
// ❌ チェック前
dependencies:
  provider: ^6.0.0  // 古い！

// ✅ チェック後
dependencies:
  flutter_riverpod: ^2.5.0  // 2025年推奨
```

#### 2. 状態管理の記法

```dart
// ❌ 古い記法
final counterProvider = StateProvider<int>((ref) => 0);

// ✅ 2025年推奨
@riverpod
class Counter extends _$Counter {
  @override
  int build() => 0;
}
```

#### 3. Null安全性

```dart
// ❌ 危険
final user = users.first;  // NoSuchElementError の可能性

// ✅ 安全
final user = users.firstOrNull;
if (user == null) return;
```

#### 4. const の使用

```dart
// ❌ 非効率
Widget build(BuildContext context) {
  return Column(
    children: [
      Text('Title'),  // constなし
      SizedBox(height: 16),  // constなし
    ],
  );
}

// ✅ 効率的
Widget build(BuildContext context) {
  return Column(
    children: [
      const Text('Title'),  // const付き
      const SizedBox(height: 16),  // const付き
    ],
  );
}
```

#### 5. リソースの破棄

```dart
// ❌ メモリリーク
class MyWidget extends StatefulWidget {
  @override
  State<MyWidget> createState() => _MyWidgetState();
}

class _MyWidgetState extends State<MyWidget> {
  final _controller = TextEditingController();
  // disposeがない！
}

// ✅ 適切な破棄
class _MyWidgetState extends State<MyWidget> {
  final _controller = TextEditingController();

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }
}
```

### 自動チェックツール

#### 1. flutter_lints

```yaml
dev_dependencies:
  flutter_lints: ^4.0.0
```

```bash
# 静的解析
flutter analyze
```

#### 2. custom_lint + riverpod_lint

```yaml
dev_dependencies:
  custom_lint: ^0.6.0
  riverpod_lint: ^2.3.0
```

```yaml
# analysis_options.yaml
analyzer:
  plugins:
    - custom_lint
```

#### 3. dart fix

```bash
# 自動修正可能な問題を修正
dart fix --apply
```

## バージョン差異の確認方法

### 1. pub.dev で確認

**手順**:
1. https://pub.dev/ にアクセス
2. パッケージ名で検索
3. 「Versions」タブで履歴を確認
4. 「Changelog」で変更内容を確認

### 2. パッケージの変更履歴

```bash
# パッケージのバージョン情報を表示
flutter pub outdated

# 特定パッケージの詳細
flutter pub deps --style=compact | grep riverpod
```

### 3. マイグレーションガイド

**主要パッケージの公式マイグレーションガイド**:

| パッケージ | ガイドURL |
|-----------|---------|
| Riverpod | https://riverpod.dev/docs/migration |
| go_router | https://pub.dev/packages/go_router#migration |
| Drift | https://drift.simonbinder.eu/docs/migrations/ |

### 4. Breaking Changes の確認

```bash
# CHANGELOGを確認
flutter pub deps --style=compact | grep package_name
# pub.devでCHANGELOGタブを確認
```

## AI開発のワークフロー

### 推奨フロー

```
1. 【仕様を明確化】
   ↓ 何を作るか、制約は何かを整理

2. 【プロンプト作成】
   ↓ 環境・要件・制約・参考資料を明記

3. 【AIにコード生成を依頼】
   ↓

4. 【コードレビュー】
   ↓ チェックリストで確認

5. 【テスト実行】
   ↓ flutter test

6. 【動作確認】
   ↓ flutter run

7. 【リファクタリング】
   ↓ 必要に応じて修正

8. 【完成】
```

### デバッグ時のAI活用

**エラーメッセージをAIに渡す際のテンプレート**:

```
【環境】
- Flutter: 3.27.0
- Dart: 3.6.0
- OS: macOS / Windows / Linux

【エラーメッセージ】
[エラーの全文をコピー]

【発生状況】
[何をした時にエラーが出たか]

【試したこと】
[すでに試した解決方法]

【質問】
このエラーを解決する方法を教えてください。
```

## よくある質問と回答

### Q1: AIが古いコードを生成した場合

**A**: プロンプトにバージョンを明示し、この書籍の該当章を参考資料として渡してください。

### Q2: 生成されたコードが動かない

**A**:
1. `flutter pub get` を実行
2. `dart run build_runner build` を実行（コード生成が必要な場合）
3. エラーメッセージを確認
4. 必要に応じてAIに再度質問

### Q3: どこまでAIに任せるべきか

**A**:
- ✅ AI に任せてよい: ボイラープレート、定型的なコード
- ⚠️ 注意が必要: ビジネスロジック、セキュリティ関連
- ❌ 人間が確認すべき: アーキテクチャ設計、重要な判断

## チェックリスト

### AI生成コードのレビュー時

- [ ] パッケージバージョンが2025年推奨版か
- [ ] 状態管理の記法が最新か（Riverpod 3.x）
- [ ] Null安全性が保たれているか
- [ ] constが適切に使われているか
- [ ] リソースが適切に破棄されているか
- [ ] テストが通るか
- [ ] 静的解析（flutter analyze）がパスするか

### プロンプト作成時

- [ ] 環境（Flutter, Dart, パッケージバージョン）を明示
- [ ] 要件を具体的に記述
- [ ] 制約（使ってはいけないもの）を明示
- [ ] 参考資料（本書の該当章）を指定

## 次のステップ

AI開発のベストプラクティスを理解したら、[第12章: よくある落とし穴とトラブルシューティング](12_troubleshooting.md)で問題解決方法を学びましょう。

---

**この章の活用法**:
生成AIにコード生成を依頼する際は、常にこの章のプロンプトテンプレートを使用してください。また、生成されたコードは必ずチェックリストで確認しましょう。
