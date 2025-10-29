# 付録A: 生成AIプロンプト集

## この付録について

この付録は、生成AIを使ってFlutterアプリを効率的に開発するための**実践的なプロンプト集**です。すぐにコピー＆ペーストして使えるように設計されています。

### 使い方

1. 作りたいアプリのプロンプトをコピー
2. 必要に応じてカスタマイズ
3. 参照章の内容を一緒に生成AIに渡す
4. 生成されたコードをレビュー（第11章のチェックリスト参照）

## プロンプトの基本構造

すべてのプロンプトは以下の構造に従っています：

```
【環境】
- Flutter: 3.27.x
- Dart: 3.6.x
- [使用するパッケージ]

【要件】
[具体的に作りたいもの]

【制約】
- [使ってはいけないもの]
- [守るべきルール]

【参考】
[本書の該当章]
```

---

## 1. 基本アプリ

### 1-1. カウンターアプリ

```
【環境】
- Flutter: 3.27.x
- Dart: 3.6.x
- 状態管理: Riverpod 3.x（@riverpod アノテーション）

【要件】
カウンターアプリを実装してください。
- カウント値の表示
- +ボタン（インクリメント）
- -ボタン（デクリメント）
- リセットボタン
- Material Design 3を使用

【制約】
- StateProvider、StateNotifierProviderは使わない
- @riverpod アノテーションでコード生成を使う
- constを適切に使用する

【参考】
第3章: 状態管理（Riverpod 3.x）の基本的な使い方を参照してください。
```

### 1-2. Todoアプリ（基本版）

```
【環境】
- Flutter: 3.27.x
- Riverpod 3.x（@riverpod）
- Drift 2.18（ローカルDB）

【要件】
Todoアプリを実装してください。

機能：
- Todo一覧の表示
- Todo追加（タイトルのみ）
- Todo完了トグル（チェックボックス）
- Todo削除（スワイプ）
- 永続化（アプリ再起動後も残る）

【制約】
- Clean Architectureに従う（Domain/Data/Presentation層）
- sqfliteではなくDrift 2.18を使う
- StateProviderは使わず@riverpodを使う

【参考】
第2章: Clean Architectureの実践
第3章: Riverpod 3.x
第6章: Drift（ローカルDB）
を参照してください。
```

### 1-3. 計算機アプリ

```
【環境】
- Flutter: 3.27.x
- Riverpod 3.x

【要件】
基本的な計算機アプリを実装してください。

機能：
- 数字ボタン（0-9）
- 演算子ボタン（+、-、×、÷）
- イコールボタン
- クリアボタン
- 表示部分（入力値と結果）

UI：
- グリッドレイアウト
- Material Design 3
- レスポンシブ対応

【制約】
- 状態管理にRiverpod 3.xを使う
- evalを使わず、安全に計算する

【参考】
第3章: 状態管理
第8章: パフォーマンス最適化（const の使用）
```

---

## 2. 中規模アプリ

### 2-1. Todoアプリ（完全版）

```
【環境】
- Flutter: 3.27.x
- Riverpod 3.x（@riverpod + コード生成）
- Drift 2.18
- Freezed 2.5
- go_router 15.x

【要件】
高機能なTodoアプリを実装してください。

機能：
1. Todo管理
   - 追加（タイトル、説明、期限、優先度）
   - 編集
   - 削除
   - 完了トグル

2. フィルター
   - すべて
   - 未完了のみ
   - 完了済みのみ
   - 優先度別

3. カテゴリ機能
   - カテゴリ作成・編集・削除
   - Todoをカテゴリにタグ付け

4. 画面
   - Todo一覧画面
   - Todo詳細・編集画面
   - カテゴリ管理画面
   - 設定画面

【アーキテクチャ】
Clean Architecture（Feature-First）

プロジェクト構造：
lib/
├── core/
│   ├── router/
│   └── theme/
├── features/
│   ├── todo/
│   │   ├── domain/
│   │   ├── data/
│   │   └── presentation/
│   └── category/
│       ├── domain/
│       ├── data/
│       └── presentation/
└── main.dart

【制約】
- sqfliteは使わずDriftを使う
- StateProviderは使わない
- 古いNavigatorは使わず、go_router 15.xを使う

【参考】
第2章: プロジェクト構造とClean Architecture
第3章: Riverpod 3.x（AsyncNotifierProvider、select）
第5章: go_router 15.x
第6章: Drift 2.18
を参照してください。
```

### 2-2. ニュースリーダーアプリ

```
【環境】
- Flutter: 3.27.x
- Riverpod 3.x
- Dio 5.4 + Retrofit 4.1
- Freezed 2.5（JSONシリアライズ）
- cached_network_image 3.3
- go_router 15.x

【要件】
ニュースリーダーアプリを実装してください。

機能：
1. ニュース一覧
   - NewsAPI（https://newsapi.org）から取得
   - 無限スクロール（ページング）
   - 画像とタイトル、要約を表示

2. ニュース詳細
   - タップで詳細画面へ遷移
   - WebViewで記事を表示

3. カテゴリ切り替え
   - トップニュース
   - テクノロジー
   - ビジネス
   - スポーツ

4. オフライン対応
   - 一度読み込んだニュースはキャッシュ
   - オフライン時はキャッシュから表示

【技術要件】
- API通信: Dio + Retrofit
- 画像キャッシュ: cached_network_image
- エラーハンドリング適切に実装
- ローディング状態を表示

【制約】
- httpパッケージではなくDio + Retrofitを使う
- 環境変数でAPIキーを管理（.envファイル）

【参考】
第4章: 推奨パッケージ（Dio、Retrofit、cached_network_image）
第5章: go_router（画面遷移）
第6章: Dio + Retrofit（HTTP通信）、オフライン対応
第8章: パフォーマンス最適化（画像、ListView.builder）
を参照してください。
```

### 2-3. 天気予報アプリ

```
【環境】
- Flutter: 3.27.x
- Riverpod 3.x
- Dio 5.4 + Retrofit 4.1
- geolocator 11.0（位置情報）
- permission_handler 11.3（権限管理）
- Freezed 2.5
- fl_chart 0.68（グラフ表示）

【要件】
天気予報アプリを実装してください。

機能：
1. 現在地の天気
   - GPSで現在地を取得
   - OpenWeatherMap APIで天気データ取得
   - 気温、湿度、風速、天気アイコン表示

2. 週間天気予報
   - 7日間の予報をリスト表示
   - 最高気温・最低気温をグラフ表示

3. 都市検索
   - 都市名で検索
   - 複数の都市を保存

4. 権限管理
   - 位置情報権限のリクエスト
   - 拒否時の適切なエラー表示

【プラットフォーム対応】
- Android: 位置情報権限（AndroidManifest.xml）
- iOS: 位置情報権限（Info.plist）

【制約】
- 位置情報権限を適切にリクエスト
- エラーハンドリング（ネットワークエラー、権限拒否）
- ローディング状態の表示

【参考】
第6章: API連携（Dio + Retrofit）
第13章: プラットフォーム固有実装（権限管理、AndroidManifest、Info.plist）
を参照してください。
```

---

## 3. 大規模アプリ

### 3-1. SNSアプリ（Twitter風）

```
【環境】
- Flutter: 3.27.x
- Riverpod 3.x（@riverpod）
- Supabase（バックエンド）
- go_router 15.x（認証ガード付き）
- cached_network_image 3.3
- image_picker 1.0（画像選択）

【要件】
Twitter風のSNSアプリを実装してください。

機能：
1. 認証
   - メール・パスワードでログイン/サインアップ
   - ログアウト
   - パスワードリセット

2. 投稿機能
   - テキスト投稿
   - 画像付き投稿
   - 投稿削除（自分の投稿のみ）

3. タイムライン
   - 全ユーザーの投稿を時系列表示
   - 無限スクロール
   - Pull to Refresh

4. ユーザープロフィール
   - プロフィール画像
   - 自己紹介
   - 投稿一覧

5. いいね機能
   - 投稿にいいねできる
   - いいね数の表示
   - いいねした投稿一覧

【アーキテクチャ】
Clean Architecture + Feature-First

lib/
├── core/
│   ├── router/（go_router、認証ガード）
│   ├── theme/
│   └── utils/
├── features/
│   ├── auth/
│   ├── timeline/
│   ├── post/
│   ├── profile/
│   └── like/
└── main.dart

【制約】
- 認証状態によるリダイレクト（go_router）
- エラーハンドリング
- ローディング状態
- 楽観的UI更新（いいね機能）

【参考】
第2章: Clean Architecture（Feature-First）
第3章: Riverpod 3.x（AsyncNotifierProvider）
第5章: go_router（認証ガード、リダイレクト）
第6章: API連携、オフライン対応
第7章: テスト（ユニット、Widget、E2E）
を参照してください。
```

### 3-2. ECアプリ

```
【環境】
- Flutter: 3.27.x
- Riverpod 3.x
- Stripe（決済）
- Firebase（認証、Firestore、Storage）
- go_router 15.x
- Drift 2.18（カート情報のローカル保存）

【要件】
ECアプリを実装してください。

機能：
1. 商品一覧
   - カテゴリ別表示
   - 検索機能
   - 並び替え（価格、人気順）

2. 商品詳細
   - 商品画像（複数、スワイプ可能）
   - 説明、価格、在庫状況
   - レビュー表示

3. カート機能
   - カートに追加
   - 数量変更
   - 削除
   - ローカルに永続化（ログイン前でも保持）

4. 注文
   - 配送先入力
   - 決済（Stripe）
   - 注文履歴

5. 認証
   - Firebase Authentication
   - メール・パスワード
   - Googleログイン

【アーキテクチャ】
Clean Architecture + Feature-First

【セキュリティ】
- 決済情報は直接扱わない（Stripe SDK使用）
- APIキーは環境変数で管理

【制約】
- ログイン必須画面と不要画面の適切な分離
- カートはローカルDB（Drift）で管理
- 決済はStripe公式SDKを使用

【参考】
第2章: Clean Architecture
第3章: Riverpod（複数Providerの組み合わせ）
第5章: go_router（認証ガード）
第6章: Drift（カート永続化）、API連携
第13章: プラットフォーム固有実装（決済）
を参照してください。
```

---

## 4. 機能追加のプロンプト

### 4-1. プッシュ通知を追加

```
【環境】
- Flutter: 3.27.x
- Firebase Cloud Messaging
- flutter_local_notifications 17.0

【要件】
既存のアプリにプッシュ通知機能を追加してください。

機能：
1. FCMトークンの取得と保存
2. フォアグラウンドでの通知表示
3. バックグラウンドでの通知受信
4. 通知タップ時の画面遷移
5. 通知権限のリクエスト（Android 13+、iOS）

【プラットフォーム対応】
Android:
- AndroidManifest.xmlにサービス追加
- 通知権限リクエスト（Android 13+）

iOS:
- Info.plistに権限説明追加
- APNs証明書設定

【参考】
第13章: プラットフォーム固有実装
- Android > プッシュ通知（FCM）
- iOS > プッシュ通知
を参照してください。
```

### 4-2. ダークモードを追加

```
【環境】
- Flutter: 3.27.x
- Riverpod 3.x
- shared_preferences 2.2（設定保存）

【要件】
アプリにダークモード切り替え機能を追加してください。

機能：
1. ライト/ダーク/システム設定の3モード
2. 設定画面でモード選択
3. 設定の永続化（アプリ再起動後も保持）
4. リアルタイムでテーマ切り替え

実装：
- Material Design 3のColorScheme使用
- ThemeDataをProviderで管理
- システム設定に従うオプション

【参考】
第3章: Riverpod 3.x（状態管理）
第6章: shared_preferences（設定保存）
を参照してください。
```

### 4-3. 多言語対応（国際化）

```
【環境】
- Flutter: 3.27.x
- flutter_localizations（標準）
- intl 0.19

【要件】
アプリを日本語と英語に対応させてください。

対応言語：
- 日本語（ja）
- 英語（en）

実装：
1. ARBファイルの作成
   - l10n/app_en.arb
   - l10n/app_ja.arb

2. 自動生成設定
   - pubspec.yamlにflutter設定
   - l10n.yamlの作成

3. 言語切り替え機能
   - 設定画面で言語選択
   - アプリ再起動後も保持

【参考】
第4章: 主要パッケージ（intl）
公式ドキュメント: https://docs.flutter.dev/ui/accessibility-and-localization/internationalization
を参照してください。
```

---

## 5. デバッグ・修正のプロンプト

### 5-1. エラー修正

```
【環境】
- Flutter: 3.27.x
- [使用しているパッケージ]

【エラー内容】
```
[エラーメッセージ全文をコピペ]
```

【発生状況】
[いつ、どの操作でエラーが発生したか]

【試したこと】
- flutter clean && flutter pub get
- [その他試したこと]

【質問】
このエラーを修正してください。

【参考】
第12章: トラブルシューティング
を参照してください。
```

### 5-2. パフォーマンス改善

```
【環境】
- Flutter: 3.27.x
- Riverpod 3.x

【問題】
以下のコードのパフォーマンスを改善してください。

```dart
[問題のあるコード]
```

【症状】
- アプリが重い
- フレームレートが低い
- スクロールがカクつく
- [その他の症状]

【改善ポイント】
- 不要な再ビルドを防ぐ
- constを適切に使う
- ListView.builderを使う
- 画像を最適化する

【参考】
第8章: パフォーマンス最適化
を参照してください。
```

---

## 6. リファクタリングのプロンプト

### 6-1. Clean Architectureへの移行

```
【環境】
- Flutter: 3.27.x
- Riverpod 3.x

【要件】
以下のコードをClean Architectureに従ってリファクタリングしてください。

【現在のコード】
```dart
[既存のコード]
```

【リファクタリング後の構造】
features/[feature_name]/
├── domain/
│   ├── entities/
│   ├── repositories/
│   └── usecases/
├── data/
│   ├── models/
│   ├── repositories/
│   └── data_sources/
└── presentation/
    ├── screens/
    ├── widgets/
    └── providers/

【制約】
- 既存の機能は維持
- テストしやすい構造に
- 依存関係を正しく（Domain → Data → Presentation）

【参考】
第2章: プロジェクト構造とClean Architecture
を参照してください。
```

### 6-2. StateProviderからRiverpod 3.xへの移行

```
【環境】
- Flutter: 3.27.x
- Riverpod 3.x（@riverpod）

【要件】
StateProvider / StateNotifierProviderを使っている以下のコードを、
Riverpod 3.xのコード生成（@riverpod）を使った形式にリファクタリングしてください。

【現在のコード】
```dart
final counterProvider = StateProvider<int>((ref) => 0);
```

【移行先】
@riverpodアノテーションを使ったコード生成形式

【参考】
第3章: 状態管理（Riverpod 3.x）
第11章: AI開発時の注意点（古い記法と推奨記法の比較）
を参照してください。
```

---

## 7. テストのプロンプト

### 7-1. ユニットテスト作成

```
【環境】
- Flutter: 3.27.x
- Riverpod 3.x
- Mockito 5.4

【要件】
以下のProviderのユニットテストを作成してください。

【テスト対象コード】
```dart
[テスト対象のコード]
```

【テスト項目】
- 初期値のテスト
- メソッド実行後の状態変化のテスト
- エラーハンドリングのテスト

【参考】
第7章: テスト戦略（Riverpod Providerのテスト）
を参照してください。
```

### 7-2. Widgetテスト作成

```
【環境】
- Flutter: 3.27.x
- flutter_test

【要件】
以下のWidgetのテストを作成してください。

【テスト対象Widget】
```dart
[Widgetのコード]
```

【テスト項目】
- 初期表示のテスト
- ボタンタップ時の挙動テスト
- 状態変化の表示テスト

【参考】
第7章: テスト戦略（Widgetテスト）
を参照してください。
```

---

## 8. CI/CDのプロンプト

### 8-1. GitHub Actions設定

```
【環境】
- Flutter: 3.27.x
- GitHub Actions

【要件】
以下の内容でGitHub Actionsワークフローを作成してください。

1. プルリクエスト時
   - flutter analyze実行
   - flutter test実行
   - カバレッジ測定
   - Codecovにアップロード

2. mainブランチへのマージ時
   - Androidリリースビルド（AAB）
   - iOSリリースビルド（IPA）
   - アーティファクトとして保存

【参考】
第10章: CI/CD（GitHub Actions設定）
を参照してください。
```

---

## プロンプトのカスタマイズ例

### 既存のプロンプトを組み合わせる

```
【組み合わせ例】
Todoアプリ + プッシュ通知 + ダークモード

上記3つのプロンプトを組み合わせて、
以下の機能を持つTodoアプリを実装してください：
- Todo管理（基本機能）
- 期限が近いTodoの通知
- ダークモード切り替え
```

### プロンプトに独自要件を追加

```
【カスタマイズ例】
ニュースリーダーアプリのプロンプトに、
以下の機能を追加してください：
- お気に入り機能
- 読んだ記事の既読管理
- シェア機能（SNS連携）
```

---

## 使用上の注意

### ✅ DO

1. **バージョンを明示する**: Flutter 3.27.x、Riverpod 3.xなど
2. **参照章を指定する**: AIに正確な情報を提供
3. **制約を明記する**: 使ってはいけないパッケージや記法
4. **段階的に開発**: 小さく始めて徐々に機能追加

### ❌ DON'T

1. **曖昧な指示をしない**: 「いい感じに作って」はNG
2. **すべてを一度に作らない**: 機能を分割して段階的に
3. **生成されたコードを盲信しない**: 必ずレビュー（第11章参照）
4. **古いパッケージを使わない**: 第4章の推奨リスト参照

---

## 次のステップ

1. **プロンプトをコピー**: 作りたいアプリのプロンプトを選択
2. **カスタマイズ**: プロジェクト固有の要件を追加
3. **参照章を確認**: 該当する章の内容を読む
4. **AIに依頼**: プロンプト + 参照章の内容を渡す
5. **コードレビュー**: 第11章のチェックリストで確認
6. **テスト**: 第7章を参考にテストを作成
7. **デプロイ**: 第10章を参考にCI/CD構築

---

**Happy Coding with AI! 🤖**
