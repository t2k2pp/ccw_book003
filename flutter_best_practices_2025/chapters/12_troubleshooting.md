# 第12章: よくある落とし穴とトラブルシューティング

## この章で学べること

- ビルドエラーの解決法
- プラットフォーム別の問題
- パッケージ競合の解決
- パフォーマンス問題の診断

## ビルドエラー

### 1. コード生成エラー

#### エラー: "Missing part directive"

```
Error: This library is missing a 'part' directive.
```

**原因**: `part` ディレクティブが不足

**解決方法**:
```dart
// ❌ partがない
import 'package:riverpod_annotation/riverpod_annotation.dart';

@riverpod
class Counter extends _$Counter {
  // ...
}

// ✅ partを追加
import 'package:riverpod_annotation/riverpod_annotation.dart';

part 'counter_provider.g.dart';  // 追加

@riverpod
class Counter extends _$Counter {
  // ...
}
```

```bash
# コード生成を実行
dart run build_runner build --delete-conflicting-outputs
```

#### エラー: "Conflicting outputs"

```
Error: Conflicting outputs were detected.
```

**解決方法**:
```bash
# 既存の生成ファイルを削除して再生成
dart run build_runner build --delete-conflicting-outputs

# または手動で削除
find . -name "*.g.dart" -type f -delete
find . -name "*.freezed.dart" -type f -delete
dart run build_runner build
```

### 2. Gradle/ビルドツールエラー

#### エラー: "Gradle version too old"

```
The current Gradle version is too old.
```

**解決方法**:
```gradle
// android/gradle/wrapper/gradle-wrapper.properties
distributionUrl=https\://services.gradle.org/distributions/gradle-8.5-all.zip
```

#### エラー: "minSdkVersion is too low"

```
Error: The minSdkVersion is too low.
```

**解決方法**:
```gradle
// android/app/build.gradle
android {
    defaultConfig {
        minSdk 24  // 21から24に変更（2025年推奨）
        targetSdk 34
    }
}
```

### 3. CocoaPods / iOS ビルドエラー

#### エラー: "Pod install failed"

```
[!] CocoaPods could not find compatible versions
```

**解決方法**:
```bash
# キャッシュをクリア
cd ios
rm -rf Pods
rm Podfile.lock

# 再インストール
pod repo update
pod install

# それでもダメなら
flutter clean
flutter pub get
cd ios && pod install
```

#### エラー: "Xcode version too old"

```
Error: Xcode version must be at least 15.0
```

**解決方法**:
1. Xcodeを最新版にアップデート（App Store）
2. Command Line Toolsを更新
```bash
sudo xcode-select --switch /Applications/Xcode.app/Contents/Developer
```

### 4. Null Safety エラー

#### エラー: "Null check operator used on a null value"

```dart
// ❌ nullの可能性があるのに ! を使用
final user = users.first!;  // NoSuchElementError

// ✅ nullチェックを追加
final user = users.firstOrNull;
if (user == null) {
  // エラーハンドリング
  return;
}
```

### 5. パッケージバージョン競合

#### エラー: "Version solving failed"

```
Because app depends on package_a ^2.0.0 which requires package_b ^1.0.0,
  and app depends on package_c which requires package_b ^2.0.0,
  version solving failed.
```

**解決方法**:
```yaml
# 1. 一方のパッケージバージョンを変更
dependencies:
  package_a: ^2.0.0
  package_c: ^1.0.0  # バージョンを下げる

# 2. dependency_overridesで強制（非推奨：最終手段）
dependency_overrides:
  package_b: ^2.0.0
```

```bash
# 依存関係ツリーを確認
flutter pub deps
```

## プラットフォーム別の問題

### Android

#### 問題: "Execution failed for task ':app:processReleaseResources'"

**原因**: リソースファイルの問題

**解決方法**:
```bash
# ビルドキャッシュをクリア
flutter clean
cd android && ./gradlew clean
cd ..
flutter pub get
flutter build apk
```

#### 問題: "MultiDex エラー"

```
Cannot fit requested classes in a single dex file
```

**解決方法**:
```gradle
// android/app/build.gradle
android {
    defaultConfig {
        multiDexEnabled true
    }
}

dependencies {
    implementation 'androidx.multidex:multidex:2.0.1'
}
```

#### 問題: "Permission denied"（実機デバッグ）

**解決方法**:
1. USB デバッグを有効化
2. 開発者向けオプションを有効化
3. PCを信頼済みデバイスとして登録

```bash
# デバイスが認識されているか確認
adb devices

# 認識されない場合
adb kill-server
adb start-server
```

### iOS

#### 問題: "Code signing error"

```
Signing for "Runner" requires a development team.
```

**解決方法**:
1. Xcodeでプロジェクトを開く
2. Signing & Capabilities タブ
3. Team を選択
4. Bundle Identifier をユニークなものに変更

#### 問題: "Unable to install app"（実機）

**解決方法**:
```bash
# 1. デバイスのキャッシュをクリア
flutter clean
rm -rf ios/Pods
cd ios && pod install

# 2. Xcodeでクリーン
cd ios
xcodebuild clean

# 3. 派生データを削除
rm -rf ~/Library/Developer/Xcode/DerivedData
```

#### 問題: "App Transport Security"

```
NSURLConnection finished with error - code -1022
```

**解決方法**:
```xml
<!-- ios/Runner/Info.plist -->
<key>NSAppTransportSecurity</key>
<dict>
  <key>NSAllowsArbitraryLoads</key>
  <true/>
  <!-- 本番環境では削除推奨 -->
</dict>
```

### Web

#### 問題: "Failed to load asset"

**原因**: CORSの問題

**解決方法**:
```bash
# Chrome で CORS を無効化して起動（開発時のみ）
flutter run -d chrome --web-browser-flag "--disable-web-security"
```

#### 問題: "Null safety issue in JavaScript"

**解決方法**:
```bash
# null safety を強制
flutter build web --release --no-sound-null-safety
```

## パフォーマンス問題

### 1. アプリが重い・遅い

#### 診断手順

```bash
# プロファイルビルドで実行
flutter run --profile

# DevToolsを開く
flutter pub global run devtools
```

**確認項目**:
- [ ] FPS が 60 を維持しているか
- [ ] Buildメソッドの実行時間
- [ ] 不要な再ビルドがないか
- [ ] 画像サイズは適切か

#### よくある原因と対策

| 原因 | 対策 |
|------|------|
| 大きすぎる画像 | `cacheWidth`/`cacheHeight`で縮小 |
| ListView の誤用 | `ListView.builder` に変更 |
| constの不足 | const を追加 |
| 不要な再ビルド | `Consumer`/`select`で最適化 |

### 2. メモリリーク

#### 診断方法

```dart
// DevToolsのMemoryタブで確認
1. アプリを操作
2. GCボタンをクリック
3. メモリが減らない → リーク疑い
```

#### よくあるリークの原因

```dart
// ❌ StreamSubscriptionを破棄していない
class _MyWidgetState extends State<MyWidget> {
  late StreamSubscription _subscription;

  @override
  void initState() {
    super.initState();
    _subscription = stream.listen((_) {});
  }
  // disposeでcancelしていない！
}

// ✅ 適切に破棄
@override
void dispose() {
  _subscription.cancel();
  super.dispose();
}
```

### 3. アプリサイズが大きい

```bash
# サイズを分析
flutter build apk --analyze-size

# APKを分割
flutter build apk --split-per-abi
```

**削減方法**:
- [ ] 不要なパッケージを削除
- [ ] 使用していないリソースを削除
- [ ] ProGuard/R8 を有効化
- [ ] 画像を最適化（WebP形式）

## デバッグテクニック

### 1. print デバッグ

```dart
import 'dart:developer' as developer;

// ❌ print（リリースビルドにも出力される）
print('Debug: $value');

// ✅ debugPrint（開発時のみ）
debugPrint('Debug: $value');

// ✅ log（より詳細な情報）
developer.log(
  'User login',
  name: 'auth',
  error: error,
  stackTrace: stackTrace,
);
```

### 2. ブレークポイント

VS Code / Android Studio:
1. 行番号の左をクリック
2. `F5` でデバッグ開始
3. 変数の値を確認

### 3. Widget Inspector

```bash
# アプリ実行中に
flutter run
```

- `p`: Widget階層を表示
- `w`: Widgetツリーをダンプ
- `r`: Hot Reload
- `R`: Hot Restart

### 4. Flutter DevTools

```bash
flutter pub global activate devtools
flutter pub global run devtools
```

**主な機能**:
- **Inspector**: Widget階層の可視化
- **Performance**: フレームレート分析
- **Memory**: メモリ使用量
- **Network**: API通信のモニタリング
- **Logging**: ログの一覧表示

## よくある質問

### Q1: "Hot Reload が効かない"

**A**:
```bash
# Hot Restart を試す
flutter run
# アプリ実行中に 'R' を押す

# それでもダメなら
flutter clean
flutter run
```

### Q2: "flutter doctor で警告が出る"

```bash
flutter doctor -v

# Android licenses
flutter doctor --android-licenses

# Xcode (macOS)
sudo xcode-select --switch /Applications/Xcode.app/Contents/Developer
```

### Q3: "パッケージが見つからない"

```
Error: Could not find package 'xxx'
```

**A**:
```bash
# キャッシュをクリア
flutter pub cache repair

# 再取得
flutter clean
flutter pub get
```

### Q4: "ビルドが途中で止まる"

**A**:
```bash
# verbose モードで詳細を確認
flutter build apk --verbose

# Gradle のメモリを増やす
# android/gradle.properties
org.gradle.jvmargs=-Xmx4096m
```

### Q5: "実機で動かない（エミュレータでは動く）"

**A**:
1. リリースビルドで試す
```bash
flutter run --release
```
2. ログを確認
```bash
# Android
adb logcat | grep flutter

# iOS
flutter logs
```

## エラーメッセージ検索のコツ

### Google検索のテクニック

```
# バージョンを指定
"Flutter 3.27" "error message"

# サイトを限定
site:stackoverflow.com flutter error message

# 期間を指定（検索ツール → 期間指定）
過去1年以内

# 完全一致
"exact error message"
```

### 公式リソース

| リソース | URL |
|---------|-----|
| Flutter 公式ドキュメント | https://docs.flutter.dev/ |
| Flutter Issues | https://github.com/flutter/flutter/issues |
| Stack Overflow | https://stackoverflow.com/questions/tagged/flutter |
| Flutter Community | https://flutter.dev/community |

## 緊急時のチェックリスト

### ビルドが通らない

- [ ] `flutter clean`
- [ ] `flutter pub get`
- [ ] `dart run build_runner build --delete-conflicting-outputs`
- [ ] Android: `cd android && ./gradlew clean`
- [ ] iOS: `cd ios && rm -rf Pods && pod install`
- [ ] Flutter/Dart SDKのバージョン確認

### アプリがクラッシュする

- [ ] エラーログを確認（Crashlytics等）
- [ ] デバッグビルドで再現するか確認
- [ ] 最近の変更を確認
- [ ] try-catch でエラーをキャッチ
- [ ] Null安全性を確認

### パフォーマンスが悪い

- [ ] DevTools でプロファイリング
- [ ] 画像サイズを確認
- [ ] 不要な再ビルドを確認
- [ ] リリースビルドで確認
- [ ] メモリリークを確認

## 最後の手段

```bash
# 完全クリーンアップ
flutter clean
rm -rf .dart_tool
rm -rf build
rm pubspec.lock

# Android
cd android
./gradlew clean
rm -rf .gradle
cd ..

# iOS
cd ios
rm -rf Pods
rm Podfile.lock
rm -rf .symlinks
pod repo update
pod install
cd ..

# 再ビルド
flutter pub get
flutter run
```

## まとめ

このドキュメントは2025年10月時点の情報に基づいています。

### 次のアクション

1. **必要な章を読む**: 目次から該当する章にジャンプ
2. **生成AIに渡す**: 必要な章のみをコンテキストとして使用
3. **コードを生成**: 第11章のプロンプトテンプレートを活用
4. **レビュー**: 第11章のチェックリストで確認
5. **トラブル時**: この第12章を参照

### フィードバック

問題や改善提案があれば、GitHubのIssueで報告してください。

---

**Happy Coding with Flutter & AI! 🚀**
