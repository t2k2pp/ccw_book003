# 付録B: Claude Code用CLAUDE.md作成ガイド

## この付録について

この付録は、**Claude Code**を使ったFlutter開発を最大限に効率化するための`CLAUDE.md`ファイルの作成ガイドです。

`CLAUDE.md`は、Claude Codeが起動時に自動的に読み込む特別なファイルで、プロジェクト固有の情報をClaude Codeに伝えることができます。

### 参考情報

- 公式ベストプラクティス: https://www.anthropic.com/engineering/claude-code-best-practices
- Claude Codeドキュメント: https://docs.claude.com/claude-code

---

## Part 1: CLAUDE.mdの基本

### CLAUDE.mdとは

**CLAUDE.md**は、Claude Code起動時に自動的にコンテキストに引き込まれる特別なマークダウンファイルです。

#### 主な目的

1. **プロジェクト固有の情報を共有**
   - アーキテクチャ、命名規則、開発ルールなど

2. **よく使うコマンドを記録**
   - ビルド、テスト、デプロイコマンドなど

3. **Claude Codeの動作をガイド**
   - コードスタイル、禁止事項、優先事項など

4. **開発効率の向上**
   - 毎回同じ説明をする必要がなくなる

### 配置場所と優先順位

Claude Codeは以下の順序でCLAUDE.mdを検索します：

#### 1. プロジェクトルート（最優先・推奨）

```
your_flutter_project/
├── CLAUDE.md          ← ここに配置（推奨）
├── lib/
├── test/
├── pubspec.yaml
└── README.md
```

**用途**: チーム全体で共有するプロジェクト固有の情報
**Gitで管理**: ✅ Yes（.gitignoreに含めない）

#### 2. 親ディレクトリ（モノレポ向け）

```
workspace/
├── CLAUDE.md          ← 共通情報
├── mobile_app/
│   └── CLAUDE.md      ← モバイルアプリ固有
└── admin_app/
    └── CLAUDE.md      ← 管理画面固有
```

**用途**: モノレポで複数のCLAUDE.mdを使い分ける

#### 3. ホームディレクトリ（グローバル設定）

```
~/.claude/CLAUDE.md
```

**用途**: すべてのプロジェクトで共通の設定
**例**: 個人的なコーディングスタイル、よく使うスニペット

#### 4. CLAUDE.local.md（個人用）

```
your_flutter_project/
├── CLAUDE.md          ← チーム共有
├── CLAUDE.local.md    ← 個人用（.gitignoreに追加）
└── .gitignore
```

**.gitignore**:
```
CLAUDE.local.md
```

**用途**: 個人的なメモ、ローカル環境固有の設定

### 優先順位のまとめ

```
CLAUDE.local.md（最優先）
  ↓
プロジェクトルートのCLAUDE.md
  ↓
親ディレクトリのCLAUDE.md
  ↓
~/.claude/CLAUDE.md（最後）
```

**複数のCLAUDE.mdがある場合**: すべてが読み込まれ、マージされます

### 基本構造

CLAUDE.mdには特定のフォーマット要件はありませんが、以下の構造が推奨されます：

```markdown
# プロジェクト名

## プロジェクト概要
[簡潔な説明]

## 重要な注意事項
**IMPORTANT**: [必ず守るべきルール]

## 開発環境
- Flutter: X.X.X
- Dart: X.X.X
- [その他のツール]

## よく使うコマンド
```bash
# ビルド
flutter build apk

# テスト
flutter test
```

## コードスタイル
- [スタイルガイドライン]

## アーキテクチャ
- [プロジェクト構造の説明]

## その他の情報
- [プロジェクト固有の情報]
```

### フォーマットのベストプラクティス

#### 1. 簡潔で読みやすく

❌ **悪い例**:
```markdown
このプロジェクトでは、Riverpod 3.xを使用した状態管理を採用しており、
Clean Architectureに基づいたレイヤー分けを行い、Feature-First
アプローチで機能ごとにディレクトリを分割し...（長文が続く）
```

✅ **良い例**:
```markdown
## 状態管理
- Riverpod 3.x（@riverpod アノテーション）
- StateProvider / StateNotifierProvider は使わない

## アーキテクチャ
- Clean Architecture（Feature-First）
- Domain / Data / Presentation の3層構造
```

#### 2. 強調表現を活用

Claude Codeに確実に従わせたい項目は強調します：

```markdown
**IMPORTANT**: StateProviderは使わず、必ず@riverpodを使うこと

**YOU MUST**: すべてのWidgetにconstを付けること

**NEVER**: GetXパッケージは使わないこと
```

#### 3. コードブロックを使う

```markdown
## コマンド例

正しいコマンド:
\`\`\`bash
flutter pub run build_runner build --delete-conflicting-outputs
\`\`\`

間違ったコマンド:
\`\`\`bash
flutter packages pub run build_runner build  # 古い記法
\`\`\`
```

### 更新とメンテナンス

#### Claude Codeから直接更新

Claude Code内で`#`キーを押すと、Claude命令を入力できます：

```
# CLAUDE.mdに以下を追加して：
- 新しく追加したfirebase_messagingの設定方法
- プッシュ通知のテストコマンド
```

Claude Codeが自動的にCLAUDE.mdを更新します。

#### 定期的な見直し

**月次レビュー**（推奨）:
- 実際に効果があった項目を残す
- 使われていない項目を削除
- 新しい慣習を追加

**効果測定**:
```markdown
<!-- 追加日: 2025-10-15 -->
<!-- 効果: Claude Codeが正しくRiverpod 3.xのコードを生成するようになった -->
**IMPORTANT**: Riverpod 3.xのコード生成を使うこと
```

コメントで追加日と効果を記録しておくと、後で見直しやすくなります。

---

## Part 2: Flutter開発での必須項目

### 1. プロジェクト情報（必須）

```markdown
# [プロジェクト名]

## プロジェクト概要
- **目的**: [アプリの目的を1-2行で]
- **対象ユーザー**: [ターゲットユーザー]
- **プラットフォーム**: iOS / Android / Web

## チーム情報
- **開発チーム**: [チーム名]
- **連絡先**: [緊急連絡先やSlackチャンネル]
```

**なぜ必須か**: Claude Codeがプロジェクトのコンテキストを理解し、適切な提案ができます。

### 2. 開発環境（必須）

```markdown
## 開発環境

**IMPORTANT**: 以下のバージョンを厳守すること

### Flutter & Dart
- Flutter: 3.27.x（fvmで管理）
- Dart: 3.6.x

### バージョン管理
```bash
# fvmでFlutterバージョンを確認
fvm flutter --version

# プロジェクトで使用するFlutterバージョン
fvm use 3.27.0
```

### 主要パッケージ
- Riverpod: 3.x（@riverpodアノテーション必須）
- go_router: 15.x
- Drift: 2.18
- Dio: 5.4

### 開発ツール
- IDE: VS Code / Android Studio
- エディタ設定: `.vscode/settings.json` 参照
```

**参考**: 詳細は本書[第1章: 開発環境セットアップ](../chapters/01_development_environment.md)を参照

### 3. よく使うコマンド（必須）

```markdown
## よく使うコマンド

### セットアップ
```bash
# 初回セットアップ
flutter pub get
dart run build_runner build --delete-conflicting-outputs
```

### 開発
```bash
# ホットリロード付きで実行
fvm flutter run

# デバイスを指定して実行
fvm flutter run -d chrome        # Web
fvm flutter run -d iPhone        # iOS Simulator
```

### コード生成
```bash
# Riverpod, Freezed, JSON等のコード生成
dart run build_runner build --delete-conflicting-outputs

# ファイル監視モード（開発中便利）
dart run build_runner watch --delete-conflicting-outputs
```

### テスト
```bash
# 全テスト実行
fvm flutter test

# カバレッジ付き
fvm flutter test --coverage

# 特定のテストファイルのみ
fvm flutter test test/features/auth/auth_test.dart
```

### ビルド
```bash
# Android APK（デバッグ）
fvm flutter build apk --debug

# Android AAB（リリース）
fvm flutter build appbundle --release

# iOS（リリース）
fvm flutter build ios --release

# Web（リリース）
fvm flutter build web --release
```

### クリーンアップ
```bash
# ビルドキャッシュをクリア
fvm flutter clean
fvm flutter pub get
dart run build_runner build --delete-conflicting-outputs
```

### 便利なエイリアス（オプション）
```bash
# Makefileを使っている場合
make get          # flutter pub get
make build-runner # コード生成
make test         # テスト実行
make clean        # クリーンアップ
```
```

**参考**: コマンドの詳細は本書[第1章: 開発環境セットアップ](../chapters/01_development_environment.md)を参照

### 4. コードスタイルガイドライン（必須）

```markdown
## コードスタイル

**IMPORTANT**: 以下のガイドラインを厳守すること

### 状態管理
**YOU MUST**:
- Riverpod 3.x のコード生成（@riverpod）を使う
- StateProvider、StateNotifierProviderは使わない

**NEVER**:
- GetX パッケージは使わない
- Provider（単体）は使わない

例:
```dart
// ✅ 正しい（2025年推奨）
@riverpod
class Counter extends _$Counter {
  @override
  int build() => 0;

  void increment() => state++;
}

// ❌ 間違い（古い記法）
final counterProvider = StateProvider<int>((ref) => 0);
```

### ナビゲーション
**YOU MUST**:
- go_router 15.x を使う
- 古いNavigator.pushは使わない

```dart
// ✅ 正しい
context.go('/details');
context.goNamed('details', pathParameters: {'id': '123'});

// ❌ 間違い
Navigator.of(context).push(MaterialPageRoute(...));
```

### データベース
**YOU MUST**:
- Drift 2.18 を使う
- sqfliteの直接使用は禁止

### パフォーマンス
**YOU MUST**:
- 可能な限りconstコンストラクタを使う
- ListView.builderを使う（Listviewは避ける）

```dart
// ✅ 正しい
const SizedBox(height: 16)
const Text('Hello')

// ❌ 間違い
SizedBox(height: 16)  // constが抜けている
```

### ファイル命名規則
- スネークケース: `user_profile_screen.dart`
- 接尾辞:
  - 画面: `*_screen.dart`
  - Widget: `*_widget.dart` または `*.dart`
  - Provider: `*_provider.dart`
  - Model: `*_model.dart`
  - Repository: `*_repository.dart`

### インポート順序
1. Dart標準ライブラリ
2. Flutterパッケージ
3. サードパーティパッケージ
4. プロジェクト内のファイル

```dart
// ✅ 正しい
import 'dart:async';

import 'package:flutter/material.dart';

import 'package:riverpod_annotation/riverpod_annotation.dart';

import '../models/user.dart';
import '../repositories/user_repository.dart';
```
```

**参考**: 詳細は本書[第2章: プロジェクト構造](../chapters/02_project_structure.md)と[第11章: AI開発のベストプラクティス](../chapters/11_ai_development_tips.md)を参照

### 5. アーキテクチャ情報（必須）

```markdown
## プロジェクト構造

### アーキテクチャ
Clean Architecture + Feature-First

### ディレクトリ構造
```
lib/
├── core/                 # 共通機能
│   ├── router/          # go_router設定
│   ├── theme/           # テーマ設定
│   └── utils/           # ユーティリティ
├── features/            # 機能ごとに分割
│   ├── auth/
│   │   ├── domain/      # ビジネスロジック
│   │   ├── data/        # データ層
│   │   └── presentation/  # UI層
│   └── home/
│       ├── domain/
│       ├── data/
│       └── presentation/
└── main.dart
```

### レイヤーの責務
**IMPORTANT**: 依存関係の方向を守ること

```
Presentation → Domain ← Data
```

- **Domain**: ビジネスロジック、他に依存しない
- **Data**: API通信、DB、Domain への変換
- **Presentation**: UI、ユーザー操作、Providerで状態管理

### 新機能追加時の手順
1. `features/[feature_name]/` ディレクトリを作成
2. Domain層から実装（Entity → Repository interface → UseCase）
3. Data層を実装（Model → Repository実装 → DataSource）
4. Presentation層を実装（Provider → Screen → Widget）

**参考**: 詳細は本書[第2章: プロジェクト構造](../chapters/02_project_structure.md)を参照
```

---

## Part 3: Flutter開発でのオプション項目

以下はプロジェクトによって追加を検討する項目です。

### 1. API情報（API使用時）

```markdown
## API情報

### エンドポイント
- **開発環境**: `https://dev-api.example.com`
- **ステージング**: `https://staging-api.example.com`
- **本番環境**: `https://api.example.com`

### 認証
- **方式**: Bearer Token（JWT）
- **トークン取得**: POST `/auth/login`
- **トークンリフレッシュ**: POST `/auth/refresh`

### 環境変数
```.env
API_BASE_URL=https://dev-api.example.com
API_KEY=your_api_key_here
```

**IMPORTANT**: APIキーは.envファイルで管理し、Gitにコミットしない
```

**参考**: 詳細は本書[第6章: データ永続化とAPI連携](../chapters/06_data_persistence.md)を参照

### 2. テスト戦略（テスト重視プロジェクト）

```markdown
## テスト戦略

### テストカバレッジ目標
- ユニットテスト: 80%以上
- Widgetテスト: 50%以上
- インテグレーションテスト: 主要フローのみ

### テスト実行
```bash
# 全テスト実行
flutter test

# カバレッジ測定
flutter test --coverage
genhtml coverage/lcov.info -o coverage/html
open coverage/html/index.html
```

### Providerのテスト例
```dart
test('Counter increments correctly', () {
  final container = ProviderContainer();
  addTearDown(container.dispose);

  expect(container.read(counterProvider), 0);
  container.read(counterProvider.notifier).increment();
  expect(container.read(counterProvider), 1);
});
```

**IMPORTANT**: すべてのビジネスロジックにユニットテストを書くこと
```

**参考**: 詳細は本書[第7章: テスト戦略](../chapters/07_testing.md)を参照

### 3. CI/CD情報（自動化プロジェクト）

```markdown
## CI/CD

### GitHub Actions
- **テスト**: プルリクエスト時に自動実行
- **ビルド**: mainブランチマージ時に自動実行
- **デプロイ**: タグプッシュ時に自動デプロイ

### ワークフロー
```bash
# タグを作成してプッシュ（リリース）
git tag v1.0.0
git push origin v1.0.0
```

### Fastlane
```bash
# iOS TestFlightにアップロード
cd ios && fastlane beta

# Android内部テストにアップロード
cd android && fastlane internal
```
```

**参考**: 詳細は本書[第10章: CI/CDとデプロイメント](../chapters/10_ci_cd.md)を参照

### 4. プラットフォーム固有設定（マルチプラットフォーム）

```markdown
## プラットフォーム固有設定

### Android
- **minSdk**: 24（Android 7.0以上）
- **targetSdk**: 34
- **権限**:
  - INTERNET（必須）
  - ACCESS_FINE_LOCATION（位置情報機能使用時）
  - CAMERA（カメラ機能使用時）

### iOS
- **最小バージョン**: iOS 13.0
- **権限説明**（Info.plist）:
  - NSCameraUsageDescription: "カメラを使用して写真を撮影します"
  - NSLocationWhenInUseUsageDescription: "現在地を取得します"

### Web
- **PWA対応**: あり
- **レンダラー**: CanvasKit（高品質）
```

**参考**: 詳細は本書[第13章: プラットフォーム固有実装](../chapters/13_platform_specific.md)を参照

### 5. よくある問題と解決策（トラブルシューティング）

```markdown
## トラブルシューティング

### コード生成エラー
**問題**: `part` directive missing

**解決策**:
```dart
import 'package:riverpod_annotation/riverpod_annotation.dart';

part 'counter_provider.g.dart';  // この行を追加

@riverpod
class Counter extends _$Counter { ... }
```

### ビルドエラー
**問題**: Gradle sync failed

**解決策**:
```bash
flutter clean
cd android && ./gradlew clean
cd ..
flutter pub get
```

### パフォーマンス問題
**問題**: スクロールがカクつく

**解決策**:
- `ListView.builder` を使う
- `const` を付ける
- 不要な再ビルドを防ぐ（`Consumer` or `select` 使用）
```

**参考**: 詳細は本書[第12章: トラブルシューティング](../chapters/12_troubleshooting.md)を参照

---

## Part 4: プロジェクト規模別テンプレート

### 小規模プロジェクト（~5画面）

```markdown
# [プロジェクト名] - Flutter App

## 概要
[アプリの簡単な説明]

## 開発環境
- Flutter: 3.27.x
- Riverpod: 3.x

## セットアップ
```bash
flutter pub get
dart run build_runner build --delete-conflicting-outputs
flutter run
```

## コードスタイル
**IMPORTANT**:
- Riverpod 3.x（@riverpod）を使う
- constを付ける
- go_router 15.xを使う

## プロジェクト構造
```
lib/
├── models/
├── screens/
├── widgets/
├── providers/
└── main.dart
```
```

### 中規模プロジェクト（10-50画面）

```markdown
# [プロジェクト名] - Flutter App

## プロジェクト概要
- **目的**: [アプリの目的]
- **プラットフォーム**: iOS / Android / Web
- **ユーザー数**: [想定ユーザー数]

## 開発環境
**IMPORTANT**: バージョンを厳守すること

### Flutter & Dart
- Flutter: 3.27.x（fvm管理）
- Dart: 3.6.x

### 主要パッケージ
- Riverpod: 3.x（@riverpod必須）
- go_router: 15.x
- Drift: 2.18
- Dio: 5.4 + Retrofit: 4.1

## セットアップ
```bash
# fvmでFlutterバージョンを設定
fvm use 3.27.0

# 依存関係インストール
fvm flutter pub get

# コード生成
dart run build_runner build --delete-conflicting-outputs

# 実行
fvm flutter run
```

## よく使うコマンド
```bash
# コード生成（監視モード）
dart run build_runner watch --delete-conflicting-outputs

# テスト実行
fvm flutter test --coverage

# ビルド
fvm flutter build apk --release
```

## コードスタイル

**YOU MUST**:
- Riverpod 3.xのコード生成を使う
- go_router 15.xを使う
- Drift 2.18を使う（sqflite直接使用禁止）
- constを可能な限り付ける

**NEVER**:
- StateProvider / StateNotifierProvider
- GetX
- Navigator.push（古い記法）

## プロジェクト構造
Clean Architecture + Feature-First

```
lib/
├── core/
│   ├── router/
│   ├── theme/
│   └── utils/
├── features/
│   ├── auth/
│   │   ├── domain/
│   │   ├── data/
│   │   └── presentation/
│   └── home/
│       ├── domain/
│       ├── data/
│       └── presentation/
└── main.dart
```

## API情報
- **開発**: `https://dev-api.example.com`
- **本番**: `https://api.example.com`
- **認証**: Bearer Token（JWT）

## テスト
```bash
# テスト実行
flutter test

# カバレッジ
flutter test --coverage
```

**目標カバレッジ**:
- ユニットテスト: 80%以上
- Widgetテスト: 50%以上

## トラブルシューティング
詳細は本書[第12章: トラブルシューティング](../chapters/12_troubleshooting.md)参照
```

### 大規模プロジェクト（50画面以上、複数チーム）

```markdown
# [プロジェクト名] - Enterprise Flutter App

## プロジェクト概要
- **目的**: [アプリの目的]
- **対象**: [ターゲットユーザー]
- **プラットフォーム**: iOS / Android / Web
- **想定ユーザー数**: [ユーザー数]

## チーム情報
- **開発チーム**: [チーム名]
- **テックリード**: @username
- **連絡先**: #slack-channel

## 開発環境

**CRITICAL**: 以下のバージョンを厳守すること。異なるバージョンでの開発は禁止。

### Flutter & Dart
- Flutter: 3.27.0（fvm厳守）
- Dart: 3.6.0

### バージョン確認
```bash
fvm flutter --version
# Flutter 3.27.0 であることを確認
```

### 主要パッケージ（2025年10月時点）
```yaml
dependencies:
  flutter_riverpod: ^2.5.0
  riverpod_annotation: ^2.5.0
  go_router: ^15.0.0
  drift: ^2.18.0
  dio: ^5.4.0
  retrofit: ^4.1.0
  freezed_annotation: ^2.5.0

dev_dependencies:
  riverpod_generator: ^2.5.0
  build_runner: ^2.4.0
  freezed: ^2.5.0
  json_serializable: ^6.8.0
```

## セットアップ（初回）

**IMPORTANT**: 以下の手順を順番通りに実行すること

```bash
# 1. fvmでFlutterバージョン設定
fvm use 3.27.0

# 2. 依存関係インストール
fvm flutter pub get

# 3. コード生成
dart run build_runner build --delete-conflicting-outputs

# 4. 環境変数設定
cp .env.example .env
# .envを編集してAPIキー等を設定

# 5. 実行
fvm flutter run
```

## よく使うコマンド

### 開発
```bash
# コード生成（監視モード）
make watch  # または dart run build_runner watch

# 実行
make run    # または fvm flutter run

# ログ確認
make logs   # または fvm flutter logs
```

### テスト
```bash
# 全テスト
make test

# カバレッジ付き
make test-coverage

# 特定のテスト
fvm flutter test test/features/auth/
```

### ビルド
```bash
# Android（開発）
make build-android-dev

# Android（本番）
make build-android-prod

# iOS（本番）
make build-ios-prod
```

## コードスタイル

**CRITICAL RULES**:

### 状態管理
**YOU MUST**:
- Riverpod 3.x のコード生成（@riverpod）を使う
- AsyncNotifierProviderを非同期処理に使う

**NEVER**:
- StateProvider
- StateNotifierProvider
- Provider（単体）
- GetX

### ナビゲーション
**YOU MUST**:
- go_router 15.x
- 型安全なルーティング

**NEVER**:
- Navigator.push（古い記法）

### データベース
**YOU MUST**:
- Drift 2.18（型安全なSQL）

**NEVER**:
- sqflite直接使用

### API通信
**YOU MUST**:
- Dio 5.4 + Retrofit 4.1
- エラーハンドリング必須

**NEVER**:
- httpパッケージ（簡易な用途を除く）

### パフォーマンス
**YOU MUST**:
- constコンストラクタを最大限活用
- ListView.builderを使用
- 不要な再ビルドを防ぐ（select使用）

## プロジェクト構造

### アーキテクチャ
Clean Architecture + Feature-First + Monorepo（オプション）

### ディレクトリ構造
```
lib/
├── main.dart
├── app.dart
├── config/               # 設定
│   ├── env/             # 環境変数
│   ├── router/          # ルーティング
│   └── theme/           # テーマ
├── core/                # 共通機能
│   ├── error/
│   ├── network/
│   ├── utils/
│   └── widgets/
├── features/            # 機能（Feature-First）
│   ├── auth/
│   │   ├── domain/      # ビジネスロジック
│   │   │   ├── entities/
│   │   │   ├── repositories/
│   │   │   └── usecases/
│   │   ├── data/        # データ層
│   │   │   ├── models/
│   │   │   ├── repositories/
│   │   │   └── data_sources/
│   │   └── presentation/  # UI層
│   │       ├── screens/
│   │       ├── widgets/
│   │       └── providers/
│   ├── home/
│   ├── profile/
│   └── settings/
└── shared/              # 共有コンポーネント
```

### レイヤー間の依存関係
**CRITICAL**: 依存方向を厳守

```
Presentation → Domain ← Data
     ↓           ↓        ↓
     Provider   UseCase  Repository
```

## API情報

### エンドポイント
- **開発**: `https://dev-api.example.com`
- **ステージング**: `https://staging-api.example.com`
- **本番**: `https://api.example.com`

### 認証
- **方式**: Bearer Token（JWT）
- **リフレッシュ**: 自動（Dio Interceptor）
- **トークン保存**: Drift（暗号化）

### レート制限
- **開発**: 100req/min
- **本番**: 1000req/min

## テスト戦略

### カバレッジ目標（CI/CDで自動チェック）
- **ユニットテスト**: 80%以上（必須）
- **Widgetテスト**: 60%以上
- **インテグレーションテスト**: 主要フローのみ

### テストピラミッド
```
      /\
     /E2E\         10%
    /------\
   / Widget \      20%
  /----------\
 /   Unit     \    70%
/--------------\
```

### テスト実行（CI/CD）
```bash
# ローカル実行
make test-all

# CI/CD（GitHub Actions）
# - PRごとに自動実行
# - カバレッジがしきい値を下回るとFail
```

## CI/CD

### GitHub Actions
- **テスト**: すべてのPRで自動実行
- **ビルド**: mainマージ時に自動ビルド
- **デプロイ**: タグプッシュ時に自動デプロイ

### ブランチ戦略
- **main**: 本番環境
- **develop**: 開発環境
- **feature/**: 新機能開発
- **hotfix/**: 緊急修正

### リリースフロー
```bash
# 1. バージョンアップ
make bump-version

# 2. タグ作成
git tag v1.2.3

# 3. プッシュ（自動デプロイ開始）
git push origin v1.2.3
```

## プラットフォーム固有設定

### Android
- **minSdk**: 24
- **targetSdk**: 34
- **署名**: release.keystoreで署名

### iOS
- **最小バージョン**: 13.0
- **証明書**: Match（Fastlane）で管理

### Web
- **PWA**: 有効
- **レンダラー**: CanvasKit

## セキュリティ

**CRITICAL**:
- APIキーは.envで管理（Gitコミット禁止）
- 認証トークンは暗号化して保存
- ProGuard/R8有効化（Android）
- 個人情報は必ず暗号化

## トラブルシューティング

### よくある問題
1. **コード生成エラー**: `part` directive missing
   → `part 'xxx.g.dart';` を追加

2. **ビルドエラー**: Gradle sync failed
   → `flutter clean && cd android && ./gradlew clean`

3. **パフォーマンス問題**: スクロールがカクつく
   → ListView.builder使用、const追加

詳細は本書[第12章: トラブルシューティング](../chapters/12_troubleshooting.md)参照

## 参考資料
- Flutter Best Practices 2025: [../README.md](../README.md)
- 第11章: AI開発のベストプラクティス
- 第12章: トラブルシューティング
```

---

## Part 5: カスタマイズのヒント

### 1. チーム固有のルールを追加

```markdown
## チーム固有ルール

### コミットメッセージ
**形式**: `[種類] 説明`

**種類**:
- `feat`: 新機能
- `fix`: バグ修正
- `docs`: ドキュメント
- `refactor`: リファクタリング
- `test`: テスト追加

**例**:
```bash
git commit -m "feat: ログイン機能を追加"
git commit -m "fix: パスワードバリデーションのバグ修正"
```

### PRレビュールール
- 2人以上の承認が必要
- CI/CDがパスしていること
- カバレッジが下がっていないこと
```

### 2. 外部サービス情報を追加

```markdown
## 外部サービス

### Firebase
- **プロジェクトID**: `my-app-prod`
- **設定ファイル**:
  - Android: `android/app/google-services.json`
  - iOS: `ios/Runner/GoogleService-Info.plist`

### Stripe（決済）
- **公開鍵**: `.env`で管理
- **Webhook URL**: `https://api.example.com/stripe/webhook`

### Sentry（エラー追跡）
- **DSN**: `.env`で管理
- **環境**: dev / staging / prod
```

### 3. 定期メンテナンス項目を追加

```markdown
## 定期メンテナンス

### 月次
- [ ] Flutter SDKの更新確認
- [ ] パッケージの更新（`flutter pub outdated`）
- [ ] CLAUDE.mdの見直し

### 四半期ごと
- [ ] 依存関係の大幅更新
- [ ] セキュリティ監査
- [ ] パフォーマンステスト
```

### 4. プロジェクト固有のスニペットを追加

```markdown
## よく使うコードスニペット

### Riverpod Provider
```dart
@riverpod
class FeatureName extends _$FeatureName {
  @override
  Future<DataType> build() async {
    // 初期データ取得
  }

  Future<void> someMethod() async {
    state = const AsyncValue.loading();
    state = await AsyncValue.guard(() async {
      // 処理
    });
  }
}
```

### Screen Template
```dart
class FeatureScreen extends ConsumerWidget {
  const FeatureScreen({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final dataAsync = ref.watch(featureProvider);

    return Scaffold(
      appBar: AppBar(title: const Text('Feature')),
      body: dataAsync.when(
        data: (data) => _buildContent(data),
        loading: () => const CircularProgressIndicator(),
        error: (error, stack) => ErrorWidget(error: error),
      ),
    );
  }
}
```
```

---

## 使用例とベストプラクティス

### CLAUDE.mdを最大限活用するために

#### 1. 段階的に追加する

**最初（最小構成）**:
- プロジェクト概要
- 開発環境
- よく使うコマンド

**次に追加**:
- コードスタイル
- アーキテクチャ

**最後に追加**:
- API情報
- テスト戦略
- トラブルシューティング

#### 2. 効果を測定する

追加した項目にコメントで効果を記録：

```markdown
<!-- 追加日: 2025-10-15 -->
<!-- 効果: Claude Codeが正しくRiverpod 3.xを生成するようになった ✅ -->
**IMPORTANT**: Riverpod 3.xのコード生成を使うこと
```

効果がなかった項目は削除します。

#### 3. チームで共有・レビュー

- 新メンバー加入時にCLAUDE.mdをレビュー
- 月次でチーム全体でレビュー会を開く
- 改善提案を積極的に取り入れる

#### 4. 本書と組み合わせる

CLAUDE.mdには簡潔な情報のみを記載し、詳細は本書を参照：

```markdown
## 状態管理
- Riverpod 3.x（@riverpod）を使う
- 詳細: 本書第3章を参照
```

これにより、CLAUDE.mdをコンパクトに保ちつつ、必要な情報にアクセスできます。

---

## チェックリスト

### CLAUDE.md作成時

- [ ] プロジェクトルートに配置
- [ ] プロジェクト概要を記載
- [ ] 開発環境（Flutter、パッケージバージョン）を記載
- [ ] よく使うコマンドを記載
- [ ] コードスタイルガイドラインを記載
- [ ] 強調表現（IMPORTANT、YOU MUST）を使用
- [ ] 本書の該当章へのリンクを追加

### 定期メンテナンス時

- [ ] 実際に効果があった項目のみ残す
- [ ] 古い情報を更新
- [ ] 新しい慣習を追加
- [ ] チームでレビュー

---

## まとめ

CLAUDE.mdは、Claude Codeとの開発を効率化する強力なツールです。

### 重要なポイント

1. **簡潔に**: 長すぎると読まれない
2. **具体的に**: 曖昧な表現は避ける
3. **強調する**: IMPORTANT、YOU MUSTで重要事項を明示
4. **更新する**: 定期的に見直して改善
5. **本書と組み合わせる**: 詳細は本書を参照

### 次のステップ

1. プロジェクトルートにCLAUDE.mdを作成
2. 最小構成（概要、環境、コマンド）から始める
3. Claude Codeで開発しながら必要な項目を追加
4. 効果を測定して改善

---

**参考資料**:
- [付録A: 生成AIプロンプト集](appendix_a_prompts.md)
- [第11章: AI開発のベストプラクティス](../chapters/11_ai_development_tips.md)
- [Anthropic Claude Code Best Practices](https://www.anthropic.com/engineering/claude-code-best-practices)
