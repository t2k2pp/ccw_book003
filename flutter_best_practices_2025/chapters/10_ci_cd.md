# 第10章: CI/CDとデプロイメント

## この章で学べること

- GitHub Actionsでの自動ビルド・テスト
- Fastlaneによる自動デプロイ
- App StoreとGoogle Playへの申請
- バージョン管理とリリース戦略

## GitHub Actions

### 基本的なワークフロー

```yaml
# .github/workflows/flutter.yml
name: Flutter CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3

      - name: Setup Flutter
        uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.27.0'
          channel: 'stable'

      - name: Get dependencies
        run: flutter pub get

      - name: Run code generation
        run: dart run build_runner build --delete-conflicting-outputs

      - name: Analyze code
        run: flutter analyze

      - name: Run tests
        run: flutter test --coverage

      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v3
        with:
          file: ./coverage/lcov.info
          fail_ci_if_error: true
```

### Android APK/AABビルド

```yaml
# .github/workflows/android_release.yml
name: Android Release

on:
  push:
    tags:
      - 'v*'

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3

      - name: Setup Java
        uses: actions/setup-java@v3
        with:
          distribution: 'zulu'
          java-version: '17'

      - name: Setup Flutter
        uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.27.0'

      - name: Decode keystore
        run: |
          echo "${{ secrets.KEYSTORE_BASE64 }}" | base64 --decode > android/app/keystore.jks

      - name: Create key.properties
        run: |
          echo "storePassword=${{ secrets.KEYSTORE_PASSWORD }}" > android/key.properties
          echo "keyPassword=${{ secrets.KEY_PASSWORD }}" >> android/key.properties
          echo "keyAlias=${{ secrets.KEY_ALIAS }}" >> android/key.properties
          echo "storeFile=keystore.jks" >> android/key.properties

      - name: Get dependencies
        run: flutter pub get

      - name: Build AAB
        run: flutter build appbundle --release

      - name: Upload AAB to artifacts
        uses: actions/upload-artifact@v3
        with:
          name: release-aab
          path: build/app/outputs/bundle/release/app-release.aab

      - name: Upload to Play Console
        uses: r0adkll/upload-google-play@v1
        with:
          serviceAccountJsonPlainText: ${{ secrets.PLAY_STORE_JSON }}
          packageName: com.example.app
          releaseFiles: build/app/outputs/bundle/release/app-release.aab
          track: internal
          status: completed
```

### iOSビルド

```yaml
# .github/workflows/ios_release.yml
name: iOS Release

on:
  push:
    tags:
      - 'v*'

jobs:
  build:
    runs-on: macos-latest

    steps:
      - uses: actions/checkout@v3

      - name: Setup Flutter
        uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.27.0'

      - name: Install CocoaPods
        run: gem install cocoapods

      - name: Get dependencies
        run: flutter pub get

      - name: Build iOS
        run: |
          cd ios
          pod install
          cd ..
          flutter build ios --release --no-codesign

      - name: Build IPA with Fastlane
        run: |
          cd ios
          fastlane release
        env:
          FASTLANE_PASSWORD: ${{ secrets.FASTLANE_PASSWORD }}
          MATCH_PASSWORD: ${{ secrets.MATCH_PASSWORD }}

      - name: Upload IPA
        uses: actions/upload-artifact@v3
        with:
          name: release-ipa
          path: build/ios/ipa/*.ipa
```

## Fastlane

### セットアップ

```bash
# Fastlaneをインストール
sudo gem install fastlane

# Android
cd android
fastlane init

# iOS
cd ios
fastlane init
```

### Android Fastfile

```ruby
# android/fastlane/Fastfile
default_platform(:android)

platform :android do
  desc "Deploy to internal track"
  lane :internal do
    gradle(
      task: "bundle",
      build_type: 'Release'
    )

    upload_to_play_store(
      track: 'internal',
      aab: '../build/app/outputs/bundle/release/app-release.aab',
      skip_upload_screenshots: true,
      skip_upload_images: true
    )
  end

  desc "Deploy to production"
  lane :production do
    gradle(
      task: "bundle",
      build_type: 'Release'
    )

    upload_to_play_store(
      track: 'production',
      aab: '../build/app/outputs/bundle/release/app-release.aab'
    )
  end

  desc "Increment version code"
  lane :bump_version do
    increment_version_code(
      gradle_file_path: "app/build.gradle"
    )
  end
end
```

### iOS Fastfile

```ruby
# ios/fastlane/Fastfile
default_platform(:ios)

platform :ios do
  desc "Push a new release build to TestFlight"
  lane :beta do
    # 証明書とプロビジョニングプロファイルの取得
    match(type: "appstore", readonly: true)

    # ビルド番号をインクリメント
    increment_build_number(
      xcodeproj: "Runner.xcodeproj"
    )

    # ビルド
    build_app(
      workspace: "Runner.xcworkspace",
      scheme: "Runner",
      export_method: "app-store",
      output_directory: "../build/ios/ipa"
    )

    # TestFlightにアップロード
    upload_to_testflight(
      skip_waiting_for_build_processing: true
    )
  end

  desc "Deploy to App Store"
  lane :release do
    match(type: "appstore", readonly: true)

    build_app(
      workspace: "Runner.xcworkspace",
      scheme: "Runner",
      export_method: "app-store"
    )

    upload_to_app_store(
      submit_for_review: true,
      automatic_release: false,
      force: true,
      precheck_include_in_app_purchases: false
    )
  end

  desc "Increment version"
  lane :bump_version do
    increment_version_number(
      bump_type: "patch"  # major, minor, patch
    )
  end
end
```

## バージョン管理

### pubspec.yaml

```yaml
version: 1.2.3+10
#        │ │ │  └─ ビルド番号（内部バージョン）
#        │ │ └──── パッチバージョン
#        │ └────── マイナーバージョン
#        └──────── メジャーバージョン
```

### セマンティックバージョニング

| バージョン | 変更内容 | 例 |
|-----------|---------|---|
| Major | 互換性のない変更 | 1.0.0 → 2.0.0 |
| Minor | 後方互換性のある機能追加 | 1.0.0 → 1.1.0 |
| Patch | バグ修正 | 1.0.0 → 1.0.1 |

### ビルド番号の自動インクリメント

```bash
# Makefileを使った自動化
.PHONY: bump-patch bump-minor bump-major

bump-patch:
	./scripts/bump_version.sh patch

bump-minor:
	./scripts/bump_version.sh minor

bump-major:
	./scripts/bump_version.sh major
```

```bash
#!/bin/bash
# scripts/bump_version.sh

TYPE=$1
CURRENT=$(grep "version:" pubspec.yaml | sed 's/version: //')
VERSION=$(echo $CURRENT | cut -d'+' -f1)
BUILD=$(echo $CURRENT | cut -d'+' -f2)

IFS='.' read -r MAJOR MINOR PATCH <<< "$VERSION"

case $TYPE in
  major)
    MAJOR=$((MAJOR + 1))
    MINOR=0
    PATCH=0
    ;;
  minor)
    MINOR=$((MINOR + 1))
    PATCH=0
    ;;
  patch)
    PATCH=$((PATCH + 1))
    ;;
esac

BUILD=$((BUILD + 1))
NEW_VERSION="$MAJOR.$MINOR.$PATCH+$BUILD"

sed -i.bak "s/version: .*/version: $NEW_VERSION/" pubspec.yaml
rm pubspec.yaml.bak

echo "Version bumped to $NEW_VERSION"
```

## App Store申請

### 必要な準備

#### 1. App Store Connect

- [ ] Apple Developer Programに登録（年間$99）
- [ ] App Store Connectでアプリを作成
- [ ] App IDを設定
- [ ] 証明書とプロビジョニングプロファイルを作成

#### 2. アプリ情報

- [ ] アプリ名（30文字以内）
- [ ] サブタイトル（30文字以内）
- [ ] プライバシーポリシーURL
- [ ] サポートURL
- [ ] アプリの説明（4000文字以内）
- [ ] キーワード（100文字以内）
- [ ] スクリーンショット（必須）
  - iPhone 6.7インチ（Pro Max）
  - iPhone 6.5インチ（Plus）
  - iPad Pro 12.9インチ

#### 3. ビルドのアップロード

```bash
cd ios
fastlane beta
```

#### 4. レビュー提出

App Store Connectで：
1. ビルドを選択
2. 輸出コンプライアンス情報を入力
3. 広告識別子の使用を申告
4. レビュー用ノートを記入
5. 提出

### レビュー期間

- **平均**: 1-3日
- **初回**: 1週間程度
- **リジェクト後の再提出**: 1-2日

### よくあるリジェクト理由

1. **プライバシーポリシーの不備**
   - 解決: 適切なプライバシーポリシーを用意

2. **メタデータの不一致**
   - 解決: スクリーンショットと実際の機能を一致させる

3. **機能の不完全**
   - 解決: すべての機能が正常に動作することを確認

## Google Play申請

### 必要な準備

#### 1. Google Play Console

- [ ] Google Play開発者アカウント（初回$25）
- [ ] アプリを作成
- [ ] ストアの掲載情報を入力

#### 2. ストアの掲載情報

- [ ] アプリ名（50文字以内）
- [ ] 簡単な説明（80文字以内）
- [ ] 詳細な説明（4000文字以内）
- [ ] プライバシーポリシーURL
- [ ] スクリーンショット（最小2枚、最大8枚）
  - 電話: 最小320px
  - タブレット（7インチ）: 最小1024px
  - タブレット（10インチ）: 最小1800px
- [ ] アイコン（512x512 PNG）
- [ ] フィーチャーグラフィック（1024x500 PNG）

#### 3. AABのアップロード

```bash
cd android
fastlane internal
```

#### 4. リリース

1. 内部テスト → 最大100人
2. クローズドテスト → 制限なし
3. オープンテスト → 公開
4. 本番環境 → 全ユーザー

### レビュー期間

- **平均**: 数時間〜1日
- **初回**: 3-7日

## 署名鍵の管理

### Android Keystore

#### 生成

```bash
keytool -genkey -v -keystore keystore.jks \
  -keyalg RSA -keysize 2048 -validity 10000 \
  -alias upload
```

#### 設定

```gradle
// android/app/build.gradle

def keystoreProperties = new Properties()
def keystorePropertiesFile = rootProject.file('key.properties')
if (keystorePropertiesFile.exists()) {
    keystoreProperties.load(new FileInputStream(keystorePropertiesFile))
}

android {
    // ...

    signingConfigs {
        release {
            keyAlias keystoreProperties['keyAlias']
            keyPassword keystoreProperties['keyPassword']
            storeFile keystoreProperties['storeFile'] ? file(keystoreProperties['storeFile']) : null
            storePassword keystoreProperties['storePassword']
        }
    }

    buildTypes {
        release {
            signingConfig signingConfigs.release
        }
    }
}
```

### iOS証明書

#### Matchを使った管理（推奨）

```bash
# 初回セットアップ
fastlane match init

# 証明書を生成してGitに保存
fastlane match appstore

# 他の開発者が証明書を取得
fastlane match appstore --readonly
```

**match設定**（ios/fastlane/Matchfile）:
```ruby
git_url("https://github.com/yourname/certificates")
storage_mode("git")
type("appstore")

app_identifier(["com.example.app"])
username("apple@example.com")
```

## 環境変数の管理

### .env ファイル

```bash
# .env.development
API_BASE_URL=https://dev-api.example.com
ANALYTICS_KEY=dev-key-123

# .env.production
API_BASE_URL=https://api.example.com
ANALYTICS_KEY=prod-key-456
```

### envify を使った読み込み

```yaml
dependencies:
  envify: ^2.0.0

dev_dependencies:
  envify_generator: ^2.0.0
  build_runner: ^2.4.0
```

```dart
import 'package:envify/envify.dart';

part 'env.g.dart';

@Envify()
abstract class Env {
  static const apiBaseUrl = _Env.apiBaseUrl;
  static const analyticsKey = _Env.analyticsKey;
}
```

```bash
# ビルド時に環境を指定
flutter build apk --dart-define-from-file=.env.production
```

## デバッグビルド vs リリースビルド

### デバッグビルド

```bash
flutter run --debug
```

**特徴**:
- Hot Reloadが使える
- デバッグ情報が含まれる
- 最適化されていない
- サイズが大きい

### プロファイルビルド

```bash
flutter run --profile
```

**特徴**:
- パフォーマンス計測が可能
- DevToolsが使える
- 最適化されている

### リリースビルド

```bash
# Android
flutter build apk --release
flutter build appbundle --release

# iOS
flutter build ios --release
```

**特徴**:
- 完全に最適化
- サイズが最小
- デバッグ情報なし

## チェックリスト

### CI/CD設定
- [ ] GitHub Actionsワークフローを作成
- [ ] テストを自動実行
- [ ] カバレッジを測定
- [ ] ビルドを自動化

### リリース準備
- [ ] バージョン番号を更新
- [ ] CHANGELOG.mdを更新
- [ ] リリースノートを作成
- [ ] スクリーンショットを準備

### Android申請
- [ ] AABをビルド
- [ ] 署名鍵を設定
- [ ] ストアの掲載情報を入力
- [ ] リリース

### iOS申請
- [ ] IPAをビルド
- [ ] 証明書とプロファイルを設定
- [ ] App Store Connectで情報を入力
- [ ] TestFlightにアップロード
- [ ] レビュー提出

## 次のステップ

CI/CDを理解したら、[第11章: 生成AI開発時の注意点](11_ai_development_tips.md)でAI開発のベストプラクティスを学びましょう。

---

**AI開発のヒント**:
生成AIに「GitHub Actionsでビルドを自動化して」と依頼する際は、この章の「GitHub Actions」セクションを一緒に渡すと、適切なワークフローを生成してくれます。
