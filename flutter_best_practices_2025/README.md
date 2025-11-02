# Flutter開発ベストプラクティス 2025

> 生成AI時代のFlutter開発リファレンスガイド

## 本書について

このドキュメントは、2025年10月時点の最新情報に基づいたFlutter開発のベストプラクティスをまとめたものです。

### 対象読者

- 生成AIを活用してFlutter開発を行う開発者
- 最新のFlutterエコシステムをキャッチアップしたい開発者
- Kotlin/Swiftとの違いや使い分けを理解したい開発者

### 本書の特徴

1. **章ごとに独立した構成**: 必要な情報に素早くアクセスできます
2. **2025年最新情報**: 生成AIが古い情報で誤る可能性がある箇所を重点的にカバー
3. **実践的なコード例**: すぐに使える実装パターンを掲載
4. **AI開発に最適化**: 各章を生成AIに渡してコンテキストとして活用できます

### 使い方

**シナリオ別の参照ガイド**

| やりたいこと | 参照する章 |
|------------|----------|
| プロジェクトを新規作成したい | [第1章](chapters/01_development_environment.md), [第2章](chapters/02_project_structure.md) |
| アプリを作るプロンプトが欲しい | [付録A](appendix/appendix_a_prompts.md) 📝 |
| Claude用のCLAUDE.mdを作りたい | [付録B](appendix/appendix_b_claude_md.md) 🤖 |
| 状態管理の実装方法を知りたい | [第3章](chapters/03_state_management.md) |
| どのパッケージを使うべきか知りたい | [第4章](chapters/04_essential_packages.md) |
| 画面遷移を実装したい | [第5章](chapters/05_navigation.md) |
| API通信やローカルDB | [第6章](chapters/06_data_persistence.md) |
| テストコードを書きたい | [第7章](chapters/07_testing.md) |
| アプリが遅い・重い | [第8章](chapters/08_performance.md) |
| ネイティブコードと連携したい | [第9章](chapters/09_platform_integration.md) |
| Android/iOS/Web固有の実装 | [第13章](chapters/13_platform_specific.md) |
| CI/CDを構築したい | [第10章](chapters/10_ci_cd.md) |
| AI生成コードの品質を上げたい | [第11章](chapters/11_ai_development_tips.md) |
| エラーが出て困っている | [第12章](chapters/12_troubleshooting.md) |

## 目次

### [第0章: はじめに](chapters/00_introduction.md)
- 本書の使い方
- Flutter 3.x系の変更点
- 2025年のFlutterエコシステム概観

### [第1章: 開発環境セットアップ（2025年最新）](chapters/01_development_environment.md)
- Flutter SDK 3.27+ のインストール
- エディタ設定（VS Code / Android Studio）
- バージョン管理ツール（fvm）
- 必須プラグイン・拡張機能

### [第2章: プロジェクト構造とアーキテクチャパターン](chapters/02_project_structure.md)
- フォルダ構成のベストプラクティス
- Feature-First vs Layer-First
- Clean Architectureの実践
- パッケージ分割戦略

### [第3章: 状態管理（Riverpod中心）](chapters/03_state_management.md)
- Riverpod 3.x の使い方
- Provider / Bloc / GetXとの比較
- AsyncNotifierProvider活用法
- コード生成（riverpod_generator）

### [第4章: 主要ライブラリとパッケージ（2025年推奨）](chapters/04_essential_packages.md)
- 2025年に使うべきパッケージ一覧
- 非推奨・メンテナンス停止パッケージ
- バージョン互換性マトリクス
- パッケージ選定基準

### [第5章: ナビゲーションとルーティング](chapters/05_navigation.md)
- go_router 15.x+ の使い方
- ディープリンク対応
- 型安全なルーティング
- Navigation 2.0の理解

### [第6章: データ永続化とAPI連携](chapters/06_data_persistence.md)
- HTTP通信（dio, http）
- ローカルDB（Drift, Isar, Hive）
- キャッシュ戦略
- オフライン対応

### [第7章: テスト戦略](chapters/07_testing.md)
- ユニットテスト
- Widgetテスト
- インテグレーションテスト
- モック・スタブの作成

### [第8章: パフォーマンス最適化](chapters/08_performance.md)
- ビルド最適化
- メモリリーク対策
- 画像最適化
- DevToolsの活用

### [第9章: Kotlin/Swiftとの連携とプラットフォーム固有実装](chapters/09_platform_integration.md)
- Method Channel / Platform Channel
- FlutterとKotlinの違い
- FFI（Foreign Function Interface）
- Pigeonによる型安全な通信

### [第10章: CI/CDとデプロイメント](chapters/10_ci_cd.md)
- GitHub Actions設定
- Fastlane統合
- 自動リリース
- ストア申請のベストプラクティス

### [第11章: 生成AI開発時の注意点とベストプラクティス](chapters/11_ai_development_tips.md)
- AIが間違いやすいポイント
- 効果的なプロンプト例
- コードレビューのチェックリスト
- バージョン差異の確認方法

### [第12章: よくある落とし穴とトラブルシューティング](chapters/12_troubleshooting.md)
- ビルドエラー解決法
- プラットフォーム別の問題
- パッケージ競合の解決
- パフォーマンス問題の診断

### [第13章: プラットフォーム固有実装（Android/iOS/Web）](chapters/13_platform_specific.md)
- Android固有の実装とベストプラクティス
- iOS固有の実装とベストプラクティス
- Web固有の実装とベストプラクティス
- プラットフォーム別のUI/UX対応

---

## コンテキストウィンドウ最適化のヒント

生成AIに渡す際は、以下のような形で必要な章だけを選択してください：

```
以下のFlutterベストプラクティスを参考に、○○を実装してください：

[第3章: 状態管理の内容をコピー]
[第6章: API連携の内容をコピー]
```

これにより、トークン使用量を最小限に抑えながら、必要な情報を正確に伝えられます。

## 付録

### [付録A: 生成AIプロンプト集](appendix/appendix_a_prompts.md)

**すぐに使える実践的なプロンプト集**

コピー＆ペーストですぐに使える形式で、以下のプロンプトを収録：

- **基本アプリ**: カウンター、Todo、計算機
- **中規模アプリ**: ニュースリーダー、天気予報
- **大規模アプリ**: SNS（Twitter風）、ECアプリ
- **機能追加**: プッシュ通知、ダークモード、多言語対応
- **デバッグ・修正**: エラー修正、パフォーマンス改善
- **リファクタリング**: Clean Architecture移行、Riverpod 3.x移行
- **テスト**: ユニットテスト、Widgetテスト
- **CI/CD**: GitHub Actions設定

各プロンプトには参照すべき章が明記されており、生成AIに渡す際に最適化されています。

### [付録B: Claude Code用CLAUDE.md作成ガイド](appendix/appendix_b_claude_md.md)

**Claude Codeで自動読み込みされるCLAUDE.mdの作成ガイド**

Flutterプロジェクトに最適なCLAUDE.mdファイルの作成方法を解説：

- **Part 1: 基礎知識**: CLAUDE.mdとは、配置場所、優先順位
- **Part 2: Flutter開発に必須の項目**: プロジェクト情報、開発環境、コマンド、コードスタイル、アーキテクチャ
- **Part 3: 任意の項目**: API情報、テスト戦略、CI/CD、プラットフォーム設定、トラブルシューティング
- **Part 4: プロジェクト規模別テンプレート**: 小規模、中規模、大規模・エンタープライズ
- **Part 5: カスタマイズのヒント**: 効果的な記述方法、本書との連携

Claude CodeユーザーがAIに正確な開発ガイドを提供できるよう、公式ベストプラクティスとFlutter固有の知識を統合したガイドです。

## バージョン情報

- **執筆時点**: 2025年10月
- **Flutter SDK**: 3.27.x
- **Dart**: 3.6.x

## ライセンス

このドキュメントは学習・開発目的で自由に利用できます。

---

**Updated**: 2025-10-29
