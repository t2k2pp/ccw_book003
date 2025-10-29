# 第0章: はじめに

## 本書の目的

このドキュメントは、**生成AIを活用したFlutter開発**を成功させるために作成されました。

2025年時点でのFlutterは非常に成熟したフレームワークですが、その進化のスピードゆえに、生成AIが古い情報や非推奨のパターンを提案してしまうことがあります。本書は、そうした問題を防ぎ、最新のベストプラクティスに基づいた開発を支援します。

## 本書の使い方

### 章ごとに独立した設計

各章は独立して読めるように設計されています。全文を読む必要はなく、必要な情報がある章だけを参照してください。

**例：状態管理を実装したい場合**
```
[第3章]の内容だけを生成AIに渡す
→ コンテキストウィンドウを節約しながら正確な実装が可能
```

### クイックリファレンス

困ったときは以下を確認：

| 症状 | 参照先 |
|------|--------|
| ビルドエラーが出る | [第12章](12_troubleshooting.md) |
| どのパッケージを使うか迷う | [第4章](04_essential_packages.md) |
| AIが古いコードを生成する | [第11章](11_ai_development_tips.md) |
| パフォーマンスが悪い | [第8章](08_performance.md) |

## Flutter 3.x系の重要な変更点（2024-2025）

### 1. Material Design 3がデフォルトに

**変更内容**:
```dart
// Flutter 3.16以降はMaterial 3がデフォルト
MaterialApp(
  theme: ThemeData(
    useMaterial3: true, // デフォルトでtrue
    colorScheme: ColorScheme.fromSeed(seedColor: Colors.blue),
  ),
);
```

**AIが間違いやすいポイント**:
- 古いAIモデルは`useMaterial3: false`のコードを生成することがある
- Material 2とMaterial 3でWidgetの見た目が大きく異なる

### 2. Null Safety完全移行

**変更内容**:
- すべての公式パッケージがNull Safetyに対応完了
- `--no-sound-null-safety`フラグは非推奨

**AIが間違いやすいポイント**:
- 古いコード例では`?`や`!`の使い方が不適切なことがある

### 3. Web・デスクトップが安定版に

**変更内容**:
- Flutter 3.0でWindows/macOS/Linuxが安定版に
- Flutter 3.7でWebレンダリングエンジンが改善

### 4. Dart 3.0の新機能

**主な機能**:
- Records（レコード型）
- Patterns（パターンマッチング）
- Class modifiers（sealed, interface, etc.）

```dart
// Dart 3.0のRecords
(String, int) getUserInfo() => ('Alice', 30);

// Pattern matching
switch (user) {
  case (String name, int age) when age >= 18:
    print('$name is an adult');
  case (String name, _):
    print('$name is a minor');
}
```

**AIが間違いやすいポイント**:
- Dart 2.x時代のパターンを提案することがある
- Recordsの代わりに不要なクラスを作成してしまう

## 2025年のFlutterエコシステム概観

### 主要な変更・トレンド

#### 1. 状態管理
- **Riverpod 3.x**が主流に
  - コード生成による型安全性向上
  - AsyncNotifierProviderの普及
- Provider 6.xはメンテナンスモードへ移行
- Blocは依然として企業での採用が多い

#### 2. ナビゲーション
- **go_router**がデファクトスタンダードに
- Navigation 2.0ベースの実装が推奨
- 宣言的ルーティングへの移行

#### 3. データ永続化
- **Drift**（旧Moor）がSQLiteの主要選択肢
- **Isar**がNoSQLの高速選択肢として台頭
- SharedPreferencesは小規模データのみに

#### 4. HTTP通信
- **dio**が依然としてトップ
- **http**パッケージもシンプルな用途では有効
- Retrofitによる型安全なAPI通信

#### 5. コード生成
- **freezed**でイミュータブルクラス生成
- **json_serializable**でJSON変換
- **build_runner**の実行が開発フローに組み込み

### 非推奨・避けるべきパッケージ

| パッケージ | 状態 | 代替 |
|-----------|------|------|
| provider（単体） | メンテナンスモード | Riverpod |
| get（GetX） | 推奨されない | Riverpod + go_router |
| sqflite（直接使用） | 低レベルすぎ | Drift, Isar |
| shared_preferences（大規模データ） | 用途外使用 | Drift, Isar, Hive |

**重要**: AIが古い情報でこれらのパッケージを提案することがあります！

## FlutterとKotlinの違い（概要）

詳細は[第9章](09_platform_integration.md)で説明しますが、重要なポイント：

| 観点 | Flutter/Dart | Kotlin |
|------|-------------|--------|
| UI構築 | 宣言的（Widget Tree） | 命令的（XMLまたはCompose） |
| 状態管理 | Riverpod等のライブラリ | ViewModel + LiveData/Flow |
| 非同期 | Future/Stream | Coroutines/Flow |
| Null安全 | 言語レベルで強制 | 言語レベルで強制 |
| クロスプラットフォーム | iOS/Android/Web/デスクトップ | KMP（Multiplatform）が必要 |

**いつFlutterを使うべきか**:
- iOS/Androidの両方を効率的に開発したい
- Webやデスクトップへの展開も視野に入れている
- 一つのコードベースで管理したい

**いつKotlin（ネイティブ）を使うべきか**:
- プラットフォーム固有の機能を多用する
- 既存のネイティブアプリへの段階的導入
- ARCore/ARKitなどの高度なネイティブ機能が必須

## 本書の構成

### Part 1: 基礎編（第1-2章）
開発環境とプロジェクト構造の確立

### Part 2: アーキテクチャ編（第3-6章）
状態管理、ナビゲーション、データ層の設計

### Part 3: 品質編（第7-8章）
テストとパフォーマンス最適化

### Part 4: 実践編（第9-10章）
ネイティブ連携とデプロイメント

### Part 5: AI開発編（第11-12章）
生成AI活用とトラブルシューティング

## 各章の想定読了時間

| 章 | 時間 | 優先度 |
|----|------|--------|
| 第1章 | 10分 | 高（初回のみ） |
| 第2章 | 15分 | 高 |
| 第3章 | 20分 | 高 |
| 第4章 | 15分 | 高 |
| 第5章 | 15分 | 中 |
| 第6章 | 20分 | 高 |
| 第7章 | 15分 | 中 |
| 第8章 | 15分 | 中 |
| 第9章 | 20分 | 低（必要時） |
| 第10章 | 15分 | 中 |
| 第11章 | 10分 | 高（AI使用時） |
| 第12章 | 参照型 | 高（問題発生時） |

## 本書の更新方針

Flutterは3-4ヶ月ごとに安定版がリリースされます。このドキュメントは以下の方針で更新されます：

- **メジャーバージョンアップ時**（Flutter 4.0など）: 全面改訂
- **マイナーバージョンアップ時**: 該当章のみ更新
- **重要なパッケージ更新時**: 第4章を更新

## 次のステップ

1. **新規プロジェクト開始の場合**: [第1章](01_development_environment.md) → [第2章](02_project_structure.md)
2. **既存プロジェクトへの参加**: [第11章](11_ai_development_tips.md) → 必要な章を参照
3. **特定の機能実装**: 目次から該当章へジャンプ

---

**では、必要な章から読み始めてください！**
