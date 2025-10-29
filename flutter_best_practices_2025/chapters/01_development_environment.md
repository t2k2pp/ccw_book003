# 第1章: 開発環境セットアップ（2025年最新）

## この章で学べること

- Flutter SDK 3.27+ のインストール方法
- エディタの推奨設定
- バージョン管理ツール（fvm）の活用
- 必須プラグイン・拡張機能

## Flutter SDK 3.27+ のインストール

### 2025年10月時点の推奨バージョン

```bash
# 推奨バージョン
Flutter: 3.27.x（Stable Channel）
Dart: 3.6.x（Flutterに同梱）
```

### インストール方法

#### 方法1: 公式インストーラー（推奨：初心者向け）

**macOS / Linux:**
```bash
# Flutter公式サイトからダウンロード
# https://docs.flutter.dev/get-started/install

# PATHに追加
export PATH="$PATH:`pwd`/flutter/bin"

# zsh使用の場合は .zshrc に追加
echo 'export PATH="$PATH:$HOME/flutter/bin"' >> ~/.zshrc
source ~/.zshrc
```

**Windows:**
```powershell
# 公式サイトからZIPをダウンロードして解凍
# C:\src\flutter に配置推奨

# システム環境変数にPATHを追加
# C:\src\flutter\bin
```

#### 方法2: fvm経由（推奨：複数プロジェクト管理時）

```bash
# fvmをインストール
dart pub global activate fvm

# Flutterバージョンをインストール
fvm install 3.27.0

# グローバルバージョンとして設定
fvm global 3.27.0

# プロジェクトごとに異なるバージョンを使用する場合
cd your_project
fvm use 3.27.0
```

### インストール確認

```bash
flutter doctor -v
```

**期待される出力:**
```
Doctor summary (to see all details, run flutter doctor -v):
[✓] Flutter (Channel stable, 3.27.0, on macOS 14.0)
[✓] Android toolchain - develop for Android devices
[✓] Xcode - develop for iOS and macOS
[✓] Chrome - develop for the web
[✓] Android Studio (version 2024.1)
[✓] VS Code (version 1.95)
[✓] Connected device (1 available)
[✓] Network resources
```

### よくあるセットアップ問題

#### 問題1: Android licenses not accepted

```bash
# 解決方法
flutter doctor --android-licenses
# 全てのライセンスに'y'で同意
```

#### 問題2: Xcode or CocoaPods not properly installed

```bash
# Xcode Command Line Toolsをインストール
sudo xcode-select --switch /Applications/Xcode.app/Contents/Developer
sudo xcodebuild -runFirstLaunch

# CocoaPodsをインストール
sudo gem install cocoapods
pod setup
```

#### 問題3: PATH not set correctly

```bash
# 現在のPATHを確認
echo $PATH

# Flutterのパスが含まれているか確認
which flutter

# 含まれていない場合、再度PATHに追加
```

## エディタ設定

### VS Code（推奨）

#### 必須拡張機能

```json
// .vscode/extensions.json
{
  "recommendations": [
    "dart-code.dart-code",
    "dart-code.flutter",
    "usernamehw.errorlens",
    "streetsidesoftware.code-spell-checker",
    "ms-vscode.vscode-typescript-next"
  ]
}
```

#### 推奨設定

```json
// .vscode/settings.json
{
  // Dart & Flutter
  "dart.flutterSdkPath": "/Users/yourname/fvm/versions/3.27.0",
  "dart.lineLength": 80,
  "dart.previewFlutterUiGuides": true,
  "dart.previewFlutterUiGuidesCustomTracking": true,

  // Editor
  "[dart]": {
    "editor.formatOnSave": true,
    "editor.formatOnType": true,
    "editor.rulers": [80],
    "editor.selectionHighlight": false,
    "editor.suggestSelection": "first",
    "editor.tabCompletion": "onlySnippets",
    "editor.wordBasedSuggestions": "off"
  },

  // Code Generation
  "dart.runPubGetOnPubspecChanges": "always",

  // DevTools
  "dart.devToolsTheme": "dark",
  "dart.embedDevTools": true,

  // Linting
  "dart.analysisExcludedFolders": [
    "**/build/**",
    "**/.dart_tool/**"
  ]
}
```

#### 便利なキーボードショートカット

| 操作 | macOS | Windows/Linux |
|------|-------|---------------|
| Widget Wrap | Cmd+. | Ctrl+. |
| Quick Fix | Cmd+. | Ctrl+. |
| Go to Definition | F12 | F12 |
| Find References | Shift+F12 | Shift+F12 |
| Restart Debugging | Cmd+Shift+F5 | Ctrl+Shift+F5 |
| Hot Reload | r (in debug console) | r (in debug console) |
| Hot Restart | R (in debug console) | R (in debug console) |

### Android Studio（代替案）

#### 必須プラグイン

1. **Flutter Plugin**
2. **Dart Plugin**
3. **Rainbow Brackets**
4. **JSON To Dart Model**（オプション）

#### 推奨設定

```
Settings → Editor → Code Style → Dart
  - Line length: 80
  - Continuation indent: 4

Settings → Languages & Frameworks → Flutter
  - Format code on save: ✓
  - Organize imports on save: ✓

Settings → Editor → Inspections → Dart
  - Enable all Dart inspections: ✓
```

## バージョン管理ツール（fvm）

### fvmとは

Flutter Version Management - 複数のFlutterバージョンをプロジェクトごとに管理するツール

### なぜfvmを使うべきか

1. **プロジェクトごとに異なるFlutterバージョン**を使える
2. **チーム全体でFlutterバージョンを統一**できる
3. **Flutter SDKの更新を気にせず安定開発**できる

### fvmの基本的な使い方

#### インストール

```bash
# Dart経由でインストール
dart pub global activate fvm

# Homebrewでもインストール可能（macOS）
brew tap leoafarias/fvm
brew install fvm
```

#### 使用方法

```bash
# 利用可能なFlutterバージョンを確認
fvm releases

# 特定バージョンをインストール
fvm install 3.27.0
fvm install 3.24.0  # 古いプロジェクト用

# プロジェクトでバージョンを指定
cd your_project
fvm use 3.27.0

# これにより .fvm/fvm_config.json が作成される
# このファイルをGitで管理することでチーム全体で統一
```

#### .fvm/fvm_config.json の例

```json
{
  "flutterSdkVersion": "3.27.0",
  "flavors": {}
}
```

#### .gitignore に追加

```bash
# .gitignore
.fvm/flutter_sdk
```

**重要**: `.fvm/fvm_config.json`はコミットし、`flutter_sdk`はignoreする

#### fvmを使ったコマンド実行

```bash
# fvm経由でflutterコマンドを実行
fvm flutter pub get
fvm flutter run
fvm flutter build apk

# エイリアス設定（オプション）
alias flutter="fvm flutter"
alias dart="fvm dart"
```

### VSCodeでfvmを使う

```json
// .vscode/settings.json
{
  "dart.flutterSdkPath": ".fvm/versions/stable",
  // または
  "dart.flutterSdkPath": ".fvm/flutter_sdk"
}
```

## 必須ツール・パッケージ

### グローバルにインストールすべきツール

```bash
# build_runner（コード生成）
dart pub global activate build_runner

# flutterfire_cli（Firebase連携）
dart pub global activate flutterfire_cli

# rename（プロジェクト名変更）
dart pub global activate rename

# flutter_launcher_icons（アイコン生成）
dart pub global activate flutter_launcher_icons

# flutter_native_splash（スプラッシュ画面）
dart pub global activate flutter_native_splash
```

### 開発に便利なCLIツール

#### 1. Very Good CLI

```bash
# Very Good CLIのインストール
dart pub global activate very_good_cli

# 新規プロジェクト作成
very_good create flutter_app my_app

# テストカバレッジ付きテスト実行
very_good test --coverage
```

#### 2. Mason

```bash
# テンプレート管理ツール
dart pub global activate mason_cli

# テンプレートからファイル生成
mason make feature --name user
```

## プロジェクトごとの初期設定

### 1. analysis_options.yaml の設定

```yaml
include: package:flutter_lints/flutter.yaml

linter:
  rules:
    # 追加の厳格なルール
    - always_declare_return_types
    - always_put_required_named_parameters_first
    - avoid_print
    - avoid_unnecessary_containers
    - prefer_const_constructors
    - prefer_const_literals_to_create_immutables
    - prefer_final_fields
    - prefer_final_in_for_each
    - prefer_single_quotes
    - sort_child_properties_last
    - use_key_in_widget_constructors

analyzer:
  exclude:
    - "**/*.g.dart"
    - "**/*.freezed.dart"
  errors:
    invalid_annotation_target: ignore
```

### 2. Makefile の作成（タスク自動化）

```makefile
# Makefile
.PHONY: help clean get build-runner test

help:
	@echo "Available commands:"
	@echo "  make get          - flutter pub get"
	@echo "  make build-runner - Run code generation"
	@echo "  make test         - Run tests"
	@echo "  make clean        - Clean project"

get:
	fvm flutter pub get

build-runner:
	fvm dart run build_runner build --delete-conflicting-outputs

watch:
	fvm dart run build_runner watch --delete-conflicting-outputs

test:
	fvm flutter test

clean:
	fvm flutter clean
	fvm flutter pub get

format:
	fvm dart format lib/ test/

analyze:
	fvm flutter analyze
```

### 3. .env ファイルの準備

```bash
# .env.example（Gitにコミット）
API_BASE_URL=https://api.example.com
API_KEY=your_api_key_here
FIREBASE_PROJECT_ID=your_project_id
```

```bash
# .env（Gitignore）
API_BASE_URL=https://api.example.com
API_KEY=actual_secret_key
FIREBASE_PROJECT_ID=production_project_id
```

```bash
# .gitignore に追加
.env
```

## プラットフォーム固有の設定

### Android

#### build.gradle設定（2025年推奨）

```gradle
// android/app/build.gradle

android {
    namespace 'com.example.app'
    compileSdk 34  // 2025年は34以上推奨

    defaultConfig {
        minSdk 24  // 最低Android 7.0
        targetSdk 34
    }

    kotlinOptions {
        jvmTarget = '17'  // Java 17が推奨
    }
}
```

### iOS

#### Podfile設定

```ruby
# ios/Podfile

platform :ios, '13.0'  # 2025年は13.0以上推奨

post_install do |installer|
  installer.pods_project.targets.each do |target|
    flutter_additional_ios_build_settings(target)
    target.build_configurations.each do |config|
      config.build_settings['IPHONEOS_DEPLOYMENT_TARGET'] = '13.0'
    end
  end
end
```

## チェックリスト

### 初回セットアップ

- [ ] Flutter SDK 3.27+ インストール
- [ ] `flutter doctor`で全項目にチェックマークがつく
- [ ] エディタ（VS Code / Android Studio）に必須プラグインをインストール
- [ ] fvmをインストールして設定
- [ ] グローバルツール（build_runner等）をインストール

### プロジェクト開始時

- [ ] `fvm use`でFlutterバージョンを固定
- [ ] `analysis_options.yaml`を設定
- [ ] `.gitignore`を確認（.env, build/等）
- [ ] Makefileを作成（オプション）
- [ ] `flutter pub get`が成功することを確認

## 次のステップ

環境設定が完了したら、[第2章: プロジェクト構造とアーキテクチャパターン](02_project_structure.md)に進んで、プロジェクトの構造設計を学びましょう。

---

**AI開発のヒント**:
生成AIに「Flutter 3.27で新規プロジェクトをセットアップして」と依頼する際は、この章の内容を一緒に渡すと、正確な設定ファイルを生成してくれます。
