# 第13章: プラットフォーム固有実装（Android/iOS/Web）

## この章で学べること

- Android固有の実装とベストプラクティス
- iOS固有の実装とベストプラクティス
- Web固有の実装とベストプラクティス
- プラットフォーム別のUI/UX対応

## Android固有実装

### プロジェクト構造

```
android/
├── app/
│   ├── src/
│   │   ├── main/
│   │   │   ├── kotlin/
│   │   │   │   └── com/example/app/
│   │   │   │       └── MainActivity.kt
│   │   │   ├── res/
│   │   │   │   ├── drawable/
│   │   │   │   ├── mipmap/
│   │   │   │   └── values/
│   │   │   └── AndroidManifest.xml
│   │   ├── debug/
│   │   └── release/
│   ├── build.gradle
│   └── proguard-rules.pro
├── gradle/
├── build.gradle
└── gradle.properties
```

### build.gradle 設定（2025年推奨）

#### プロジェクトレベル（android/build.gradle）

```gradle
buildscript {
    ext.kotlin_version = '1.9.22'
    repositories {
        google()
        mavenCentral()
    }

    dependencies {
        classpath 'com.android.tools.build:gradle:8.2.0'
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"
        classpath 'com.google.gms:google-services:4.4.0'  // Firebase
    }
}

allprojects {
    repositories {
        google()
        mavenCentral()
    }
}
```

#### アプリレベル（android/app/build.gradle）

```gradle
plugins {
    id "com.android.application"
    id "kotlin-android"
    id "dev.flutter.flutter-gradle-plugin"
}

def localProperties = new Properties()
def localPropertiesFile = rootProject.file('local.properties')
if (localPropertiesFile.exists()) {
    localPropertiesFile.withReader('UTF-8') { reader ->
        localProperties.load(reader)
    }
}

def flutterVersionCode = localProperties.getProperty('flutter.versionCode')
if (flutterVersionCode == null) {
    flutterVersionCode = '1'
}

def flutterVersionName = localProperties.getProperty('flutter.versionName')
if (flutterVersionName == null) {
    flutterVersionName = '1.0'
}

android {
    namespace "com.example.app"
    compileSdk 34  // 2025年推奨

    compileOptions {
        sourceCompatibility JavaVersion.VERSION_17
        targetCompatibility JavaVersion.VERSION_17
    }

    kotlinOptions {
        jvmTarget = '17'
    }

    defaultConfig {
        applicationId "com.example.app"
        minSdk 24  // Android 7.0以上（2025年推奨）
        targetSdk 34
        versionCode flutterVersionCode.toInteger()
        versionName flutterVersionName

        // MultiDex対応
        multiDexEnabled true

        // ネイティブライブラリの分割
        ndk {
            abiFilters 'armeabi-v7a', 'arm64-v8a', 'x86_64'
        }
    }

    signingConfigs {
        release {
            def keystoreProperties = new Properties()
            def keystorePropertiesFile = rootProject.file('key.properties')
            if (keystorePropertiesFile.exists()) {
                keystoreProperties.load(new FileInputStream(keystorePropertiesFile))
            }

            keyAlias keystoreProperties['keyAlias']
            keyPassword keystoreProperties['keyPassword']
            storeFile keystoreProperties['storeFile'] ? file(keystoreProperties['storeFile']) : null
            storePassword keystoreProperties['storePassword']
        }
    }

    buildTypes {
        debug {
            applicationIdSuffix ".debug"
            debuggable true
        }

        release {
            signingConfig signingConfigs.release

            // コード難読化
            minifyEnabled true
            shrinkResources true

            // ProGuard設定
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }

    // フレーバー設定（dev/staging/prod）
    flavorDimensions "environment"
    productFlavors {
        dev {
            dimension "environment"
            applicationIdSuffix ".dev"
            resValue "string", "app_name", "MyApp Dev"
        }
        staging {
            dimension "environment"
            applicationIdSuffix ".staging"
            resValue "string", "app_name", "MyApp Staging"
        }
        prod {
            dimension "environment"
            resValue "string", "app_name", "MyApp"
        }
    }
}

flutter {
    source '../..'
}

dependencies {
    implementation "org.jetbrains.kotlin:kotlin-stdlib-jdk8:$kotlin_version"
    implementation 'androidx.multidex:multidex:2.0.1'
}
```

### AndroidManifest.xml

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android">
    <!-- 権限 -->
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="android.permission.CAMERA" />
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
    <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
    <uses-permission android:name="android.permission.POST_NOTIFICATIONS" />  <!-- Android 13+ -->

    <!-- カメラ機能を必須にしない -->
    <uses-feature android:name="android.hardware.camera" android:required="false" />

    <application
        android:label="@string/app_name"
        android:name="${applicationName}"
        android:icon="@mipmap/ic_launcher"
        android:usesCleartextTraffic="false"
        android:allowBackup="false"
        android:fullBackupContent="false"
        android:requestLegacyExternalStorage="false">

        <activity
            android:name=".MainActivity"
            android:exported="true"
            android:launchMode="singleTop"
            android:theme="@style/LaunchTheme"
            android:configChanges="orientation|keyboardHidden|keyboard|screenSize|smallestScreenSize|locale|layoutDirection|fontScale|screenLayout|density|uiMode"
            android:hardwareAccelerated="true"
            android:windowSoftInputMode="adjustResize">

            <!-- スプラッシュスクリーン -->
            <meta-data
                android:name="io.flutter.embedding.android.SplashScreenDrawable"
                android:resource="@drawable/launch_background" />

            <!-- メインインテント -->
            <intent-filter>
                <action android:name="android.intent.action.MAIN"/>
                <category android:name="android.intent.category.LAUNCHER"/>
            </intent-filter>

            <!-- ディープリンク -->
            <intent-filter>
                <action android:name="android.intent.action.VIEW" />
                <category android:name="android.intent.category.DEFAULT" />
                <category android:name="android.intent.category.BROWSABLE" />
                <data
                    android:scheme="https"
                    android:host="example.com"
                    android:pathPrefix="/app" />
            </intent-filter>

            <!-- アプリリンク（App Links） -->
            <intent-filter android:autoVerify="true">
                <action android:name="android.intent.action.VIEW" />
                <category android:name="android.intent.category.DEFAULT" />
                <category android:name="android.intent.category.BROWSABLE" />
                <data
                    android:scheme="https"
                    android:host="example.com" />
            </intent-filter>
        </activity>

        <!-- Don't delete the meta-data below -->
        <meta-data
            android:name="flutterEmbedding"
            android:value="2" />

        <!-- Firebase Cloud Messaging -->
        <service
            android:name="com.google.firebase.messaging.FirebaseMessagingService"
            android:exported="false">
            <intent-filter>
                <action android:name="com.google.firebase.MESSAGING_EVENT" />
            </intent-filter>
        </service>
    </application>
</manifest>
```

### 権限のランタイムリクエスト

```dart
import 'package:permission_handler/permission_handler.dart';

Future<bool> requestCameraPermission() async {
  final status = await Permission.camera.request();

  if (status.isGranted) {
    return true;
  } else if (status.isDenied) {
    // 拒否された
    return false;
  } else if (status.isPermanentlyDenied) {
    // 永続的に拒否された → 設定画面へ誘導
    await openAppSettings();
    return false;
  }

  return false;
}

// Android 13+ の通知権限
Future<bool> requestNotificationPermission() async {
  if (Platform.isAndroid) {
    final androidInfo = await DeviceInfoPlugin().androidInfo;
    if (androidInfo.version.sdkInt >= 33) {  // Android 13+
      final status = await Permission.notification.request();
      return status.isGranted;
    }
  }
  return true;  // Android 12以下は不要
}
```

### バックグラウンドタスク

```kotlin
// android/app/src/main/kotlin/com/example/app/BackgroundService.kt
import android.content.Context
import androidx.work.Worker
import androidx.work.WorkerParameters

class BackgroundWorker(
    context: Context,
    params: WorkerParameters
) : Worker(context, params) {

    override fun doWork(): Result {
        // バックグラウンド処理
        return Result.success()
    }
}
```

```dart
// Flutter側
import 'package:workmanager/workmanager.dart';

void callbackDispatcher() {
  Workmanager().executeTask((task, inputData) {
    print("Background task: $task");
    return Future.value(true);
  });
}

void main() {
  WidgetsFlutterBinding.ensureInitialized();
  Workmanager().initialize(callbackDispatcher);

  // 定期実行タスク
  Workmanager().registerPeriodicTask(
    "1",
    "syncTask",
    frequency: const Duration(hours: 1),
  );

  runApp(MyApp());
}
```

### プッシュ通知（FCM）

```kotlin
// android/app/src/main/kotlin/com/example/app/MyFirebaseMessagingService.kt
import com.google.firebase.messaging.FirebaseMessagingService
import com.google.firebase.messaging.RemoteMessage

class MyFirebaseMessagingService : FirebaseMessagingService() {

    override fun onMessageReceived(remoteMessage: RemoteMessage) {
        // 通知を受信
        remoteMessage.notification?.let {
            showNotification(it.title, it.body)
        }
    }

    override fun onNewToken(token: String) {
        // 新しいトークンを取得
        sendTokenToServer(token)
    }

    private fun showNotification(title: String?, body: String?) {
        // 通知を表示
    }
}
```

### ProGuard設定（難読化）

```proguard
# android/app/proguard-rules.pro

# Flutter
-keep class io.flutter.app.** { *; }
-keep class io.flutter.plugin.**  { *; }
-keep class io.flutter.util.**  { *; }
-keep class io.flutter.view.**  { *; }
-keep class io.flutter.**  { *; }
-keep class io.flutter.plugins.**  { *; }

# Firebase
-keep class com.google.firebase.** { *; }
-dontwarn com.google.firebase.**

# Gson
-keepattributes Signature
-keepattributes *Annotation*
-keep class com.google.gson.** { *; }

# カスタムモデルクラス
-keep class com.example.app.models.** { *; }
```

## iOS固有実装

### プロジェクト構造

```
ios/
├── Runner/
│   ├── AppDelegate.swift
│   ├── Assets.xcassets/
│   ├── Base.lproj/
│   └── Info.plist
├── Runner.xcodeproj/
├── Runner.xcworkspace/
├── Podfile
└── Podfile.lock
```

### Info.plist 設定

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <!-- アプリ名 -->
    <key>CFBundleDisplayName</key>
    <string>MyApp</string>

    <!-- バージョン -->
    <key>CFBundleShortVersionString</key>
    <string>$(FLUTTER_BUILD_NAME)</string>
    <key>CFBundleVersion</key>
    <string>$(FLUTTER_BUILD_NUMBER)</string>

    <!-- 最小iOSバージョン -->
    <key>MinimumOSVersion</key>
    <string>13.0</string>

    <!-- 権限の説明文（必須） -->
    <key>NSCameraUsageDescription</key>
    <string>カメラを使用して写真を撮影します</string>

    <key>NSPhotoLibraryUsageDescription</key>
    <string>写真ライブラリから画像を選択します</string>

    <key>NSPhotoLibraryAddUsageDescription</key>
    <string>写真を保存します</string>

    <key>NSLocationWhenInUseUsageDescription</key>
    <string>現在地を取得します</string>

    <key>NSLocationAlwaysAndWhenInUseUsageDescription</key>
    <string>位置情報を常に取得します</string>

    <key>NSMicrophoneUsageDescription</key>
    <string>音声を録音します</string>

    <key>NSContactsUsageDescription</key>
    <string>連絡先にアクセスします</string>

    <key>NSCalendarsUsageDescription</key>
    <string>カレンダーにアクセスします</string>

    <key>NSUserTrackingUsageDescription</key>
    <string>広告のパーソナライズに使用します</string>

    <!-- URLスキーム -->
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

    <!-- ユニバーサルリンク -->
    <key>com.apple.developer.associated-domains</key>
    <array>
        <string>applinks:example.com</string>
    </array>

    <!-- App Transport Security -->
    <key>NSAppTransportSecurity</key>
    <dict>
        <key>NSAllowsArbitraryLoads</key>
        <false/>
        <!-- 特定のドメインのみHTTPを許可 -->
        <key>NSExceptionDomains</key>
        <dict>
            <key>example.com</key>
            <dict>
                <key>NSExceptionAllowsInsecureHTTPLoads</key>
                <true/>
                <key>NSIncludesSubdomains</key>
                <true/>
            </dict>
        </dict>
    </dict>

    <!-- バックグラウンド実行 -->
    <key>UIBackgroundModes</key>
    <array>
        <string>fetch</string>
        <string>remote-notification</string>
        <string>processing</string>
    </array>

    <!-- ステータスバー -->
    <key>UIViewControllerBasedStatusBarAppearance</key>
    <false/>
    <key>UIStatusBarStyle</key>
    <string>UIStatusBarStyleLightContent</string>

    <!-- 画面の向き -->
    <key>UISupportedInterfaceOrientations</key>
    <array>
        <string>UIInterfaceOrientationPortrait</string>
        <string>UIInterfaceOrientationLandscapeLeft</string>
        <string>UIInterfaceOrientationLandscapeRight</string>
    </array>

    <key>UISupportedInterfaceOrientations~ipad</key>
    <array>
        <string>UIInterfaceOrientationPortrait</string>
        <string>UIInterfaceOrientationPortraitUpsideDown</string>
        <string>UIInterfaceOrientationLandscapeLeft</string>
        <string>UIInterfaceOrientationLandscapeRight</string>
    </array>
</dict>
</plist>
```

### Podfile 設定

```ruby
# ios/Podfile

platform :ios, '13.0'

# CocoaPods analytics sends network stats synchronously affecting flutter build latency.
ENV['COCOAPODS_DISABLE_STATS'] = 'true'

project 'Runner', {
  'Debug' => :debug,
  'Profile' => :release,
  'Release' => :release,
}

def flutter_root
  generated_xcode_build_settings_path = File.expand_path(File.join('..', 'Flutter', 'Generated.xcconfig'), __FILE__)
  unless File.exist?(generated_xcode_build_settings_path)
    raise "#{generated_xcode_build_settings_path} must exist. If you're running pod install manually, make sure flutter pub get is executed first"
  end

  File.foreach(generated_xcode_build_settings_path) do |line|
    matches = line.match(/FLUTTER_ROOT\=(.*)/)
    return matches[1].strip if matches
  end
  raise "FLUTTER_ROOT not found in #{generated_xcode_build_settings_path}. Try deleting Generated.xcconfig, then run flutter pub get"
end

require File.expand_path(File.join('packages', 'flutter_tools', 'bin', 'podhelper'), flutter_root)

flutter_ios_podfile_setup

target 'Runner' do
  use_frameworks!
  use_modular_headers!

  flutter_install_all_ios_pods File.dirname(File.realpath(__FILE__))

  # カスタムPod
  # pod 'SomeLibrary', '~> 1.0'
end

post_install do |installer|
  installer.pods_project.targets.each do |target|
    flutter_additional_ios_build_settings(target)

    target.build_configurations.each do |config|
      # 最小iOSバージョン
      config.build_settings['IPHONEOS_DEPLOYMENT_TARGET'] = '13.0'

      # Bitcodeを無効化（2025年は不要）
      config.build_settings['ENABLE_BITCODE'] = 'NO'

      # Swift最適化
      config.build_settings['SWIFT_VERSION'] = '5.0'
    end
  end
end
```

### AppDelegate.swift

```swift
import UIKit
import Flutter
import UserNotifications

@UIApplicationMain
@objc class AppDelegate: FlutterAppDelegate {
  override func application(
    _ application: UIApplication,
    didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
  ) -> Bool {
    // プッシュ通知の許可をリクエスト
    if #available(iOS 10.0, *) {
      UNUserNotificationCenter.current().delegate = self
      let authOptions: UNAuthorizationOptions = [.alert, .badge, .sound]
      UNUserNotificationCenter.current().requestAuthorization(
        options: authOptions,
        completionHandler: { _, _ in }
      )
    } else {
      let settings: UIUserNotificationSettings =
        UIUserNotificationSettings(types: [.alert, .badge, .sound], categories: nil)
      application.registerUserNotificationSettings(settings)
    }

    application.registerForRemoteNotifications()

    GeneratedPluginRegistrant.register(with: self)
    return super.application(application, didFinishLaunchingWithOptions: launchOptions)
  }

  // プッシュ通知トークン取得
  override func application(
    _ application: UIApplication,
    didRegisterForRemoteNotificationsWithDeviceToken deviceToken: Data
  ) {
    let token = deviceToken.map { String(format: "%02.2hhx", $0) }.joined()
    print("Device Token: \(token)")
    // サーバーに送信
  }

  // プッシュ通知受信
  override func userNotificationCenter(
    _ center: UNUserNotificationCenter,
    willPresent notification: UNNotification,
    withCompletionHandler completionHandler: @escaping (UNNotificationPresentationOptions) -> Void
  ) {
    completionHandler([.alert, .badge, .sound])
  }

  // ディープリンク/ユニバーサルリンク
  override func application(
    _ app: UIApplication,
    open url: URL,
    options: [UIApplication.OpenURLOptionsKey : Any] = [:]
  ) -> Bool {
    // URLスキームの処理
    return super.application(app, open: url, options: options)
  }

  override func application(
    _ application: UIApplication,
    continue userActivity: NSUserActivity,
    restorationHandler: @escaping ([UIUserActivityRestoring]?) -> Void
  ) -> Bool {
    // ユニバーサルリンクの処理
    return super.application(application, continue: userActivity, restorationHandler: restorationHandler)
  }
}
```

### バックグラウンド実行

```swift
// ios/Runner/BackgroundTask.swift
import BackgroundTasks

class BackgroundTaskManager {
    static let shared = BackgroundTaskManager()

    func registerBackgroundTasks() {
        BGTaskScheduler.shared.register(
            forTaskWithIdentifier: "com.example.app.refresh",
            using: nil
        ) { task in
            self.handleAppRefresh(task: task as! BGAppRefreshTask)
        }
    }

    func scheduleAppRefresh() {
        let request = BGAppRefreshTaskRequest(identifier: "com.example.app.refresh")
        request.earliestBeginDate = Date(timeIntervalSinceNow: 15 * 60) // 15分後

        do {
            try BGTaskScheduler.shared.submit(request)
        } catch {
            print("Could not schedule app refresh: \(error)")
        }
    }

    func handleAppRefresh(task: BGAppRefreshTask) {
        scheduleAppRefresh()  // 次回のタスクをスケジュール

        task.expirationHandler = {
            // タスクがキャンセルされた時の処理
        }

        // バックグラウンド処理
        performBackgroundWork { success in
            task.setTaskCompleted(success: success)
        }
    }

    func performBackgroundWork(completion: @escaping (Bool) -> Void) {
        // 実際の処理
        completion(true)
    }
}
```

## Web固有実装

### プロジェクト構造

```
web/
├── index.html
├── manifest.json
├── favicon.png
├── icons/
│   ├── Icon-192.png
│   └── Icon-512.png
└── splash/
    └── img/
```

### index.html カスタマイズ

```html
<!DOCTYPE html>
<html>
<head>
  <base href="$FLUTTER_BASE_HREF">

  <meta charset="UTF-8">
  <meta content="IE=Edge" http-equiv="X-UA-Compatible">
  <meta name="description" content="A Flutter application">

  <!-- iOS meta tags -->
  <meta name="apple-mobile-web-app-capable" content="yes">
  <meta name="apple-mobile-web-app-status-bar-style" content="black">
  <meta name="apple-mobile-web-app-title" content="MyApp">
  <link rel="apple-touch-icon" href="icons/Icon-192.png">

  <!-- Favicon -->
  <link rel="icon" type="image/png" href="favicon.png"/>

  <title>MyApp</title>
  <link rel="manifest" href="manifest.json">

  <!-- PWA設定 -->
  <meta name="theme-color" content="#2196F3">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">

  <!-- SEO -->
  <meta name="keywords" content="flutter, app">
  <meta property="og:title" content="MyApp">
  <meta property="og:description" content="A Flutter application">
  <meta property="og:image" content="icons/Icon-512.png">
  <meta property="og:url" content="https://example.com">
  <meta name="twitter:card" content="summary_large_image">

  <style>
    /* ローディング画面 */
    .loading {
      display: flex;
      justify-content: center;
      align-items: center;
      height: 100vh;
      background: #ffffff;
    }
    .spinner {
      border: 4px solid #f3f3f3;
      border-top: 4px solid #2196F3;
      border-radius: 50%;
      width: 40px;
      height: 40px;
      animation: spin 1s linear infinite;
    }
    @keyframes spin {
      0% { transform: rotate(0deg); }
      100% { transform: rotate(360deg); }
    }
  </style>
</head>
<body>
  <!-- ローディング表示 -->
  <div class="loading">
    <div class="spinner"></div>
  </div>

  <script>
    // Service Workerの登録
    if ('serviceWorker' in navigator) {
      window.addEventListener('load', function () {
        navigator.serviceWorker.register('flutter_service_worker.js');
      });
    }

    // アナリティクスなど
    // Google Analytics
    // window.dataLayer = window.dataLayer || [];
    // function gtag(){dataLayer.push(arguments);}
    // gtag('js', new Date());
    // gtag('config', 'G-XXXXXXXXXX');
  </script>

  <script src="flutter.js" defer></script>
  <script>
    window.addEventListener('load', function(ev) {
      _flutter.loader.loadEntrypoint({
        serviceWorker: {
          serviceWorkerVersion: serviceWorkerVersion,
        },
        onEntrypointLoaded: function(engineInitializer) {
          engineInitializer.initializeEngine().then(function(appRunner) {
            appRunner.runApp();
          });
        }
      });
    });
  </script>
</body>
</html>
```

### manifest.json（PWA）

```json
{
  "name": "MyApp",
  "short_name": "MyApp",
  "start_url": ".",
  "display": "standalone",
  "background_color": "#ffffff",
  "theme_color": "#2196F3",
  "description": "A Flutter application",
  "orientation": "portrait-primary",
  "prefer_related_applications": false,
  "icons": [
    {
      "src": "icons/Icon-192.png",
      "sizes": "192x192",
      "type": "image/png",
      "purpose": "any maskable"
    },
    {
      "src": "icons/Icon-512.png",
      "sizes": "512x512",
      "type": "image/png",
      "purpose": "any maskable"
    }
  ]
}
```

### Web固有のコード

```dart
import 'dart:html' as html;
import 'package:flutter/foundation.dart' show kIsWeb;

class WebUtils {
  // プラットフォームチェック
  static bool get isWeb => kIsWeb;

  // URLからパラメータを取得
  static String? getQueryParameter(String key) {
    if (!kIsWeb) return null;
    return Uri.base.queryParameters[key];
  }

  // ブラウザバックの制御
  static void preventBackNavigation() {
    if (!kIsWeb) return;
    html.window.history.pushState(null, '', html.window.location.href);
    html.window.onPopState.listen((event) {
      html.window.history.pushState(null, '', html.window.location.href);
    });
  }

  // ダウンロード機能
  static void downloadFile(String url, String filename) {
    if (!kIsWeb) return;
    final anchor = html.AnchorElement(href: url)
      ..setAttribute('download', filename)
      ..click();
  }

  // クリップボードにコピー
  static Future<void> copyToClipboard(String text) async {
    if (!kIsWeb) return;
    await html.window.navigator.clipboard?.writeText(text);
  }

  // フルスクリーン切り替え
  static void toggleFullscreen() {
    if (!kIsWeb) return;
    if (html.document.fullscreenElement == null) {
      html.document.documentElement?.requestFullscreen();
    } else {
      html.document.exitFullscreen();
    }
  }

  // ローカルストレージ
  static void saveToLocalStorage(String key, String value) {
    if (!kIsWeb) return;
    html.window.localStorage[key] = value;
  }

  static String? getFromLocalStorage(String key) {
    if (!kIsWeb) return null;
    return html.window.localStorage[key];
  }
}
```

### レスポンシブ対応

```dart
import 'package:flutter/material.dart';

class ResponsiveLayout extends StatelessWidget {
  final Widget mobile;
  final Widget? tablet;
  final Widget desktop;

  const ResponsiveLayout({
    required this.mobile,
    this.tablet,
    required this.desktop,
  });

  static bool isMobile(BuildContext context) =>
      MediaQuery.of(context).size.width < 650;

  static bool isTablet(BuildContext context) =>
      MediaQuery.of(context).size.width >= 650 &&
      MediaQuery.of(context).size.width < 1100;

  static bool isDesktop(BuildContext context) =>
      MediaQuery.of(context).size.width >= 1100;

  @override
  Widget build(BuildContext context) {
    final size = MediaQuery.of(context).size;

    if (size.width >= 1100) {
      return desktop;
    } else if (size.width >= 650) {
      return tablet ?? mobile;
    } else {
      return mobile;
    }
  }
}

// 使用例
class MyHomePage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return ResponsiveLayout(
      mobile: MobileLayout(),
      tablet: TabletLayout(),
      desktop: DesktopLayout(),
    );
  }
}
```

### SEO対策（Web）

```dart
// lib/web/seo_helper.dart
import 'package:flutter/foundation.dart';
import 'dart:html' as html;

class SeoHelper {
  static void setPageTitle(String title) {
    if (!kIsWeb) return;
    html.document.title = title;
  }

  static void setMetaDescription(String description) {
    if (!kIsWeb) return;
    final meta = html.document.querySelector('meta[name="description"]');
    meta?.setAttribute('content', description);
  }

  static void setCanonicalUrl(String url) {
    if (!kIsWeb) return;
    var link = html.document.querySelector('link[rel="canonical"]') as html.LinkElement?;
    if (link == null) {
      link = html.LinkElement()
        ..rel = 'canonical'
        ..href = url;
      html.document.head?.append(link);
    } else {
      link.href = url;
    }
  }
}

// 使用例
class ProductDetailScreen extends StatelessWidget {
  final Product product;

  @override
  Widget build(BuildContext context) {
    if (kIsWeb) {
      SeoHelper.setPageTitle('${product.name} - MyApp');
      SeoHelper.setMetaDescription(product.description);
      SeoHelper.setCanonicalUrl('https://example.com/products/${product.id}');
    }

    return Scaffold(
      // ...
    );
  }
}
```

## プラットフォーム判定

### ベストプラクティス

```dart
import 'dart:io' show Platform;
import 'package:flutter/foundation.dart' show kIsWeb;

class PlatformUtils {
  static bool get isAndroid => !kIsWeb && Platform.isAndroid;
  static bool get isIOS => !kIsWeb && Platform.isIOS;
  static bool get isWeb => kIsWeb;
  static bool get isMobile => isAndroid || isIOS;
  static bool get isDesktop => !kIsWeb && (Platform.isMacOS || Platform.isWindows || Platform.isLinux);

  static String get platformName {
    if (isWeb) return 'Web';
    if (isAndroid) return 'Android';
    if (isIOS) return 'iOS';
    return 'Unknown';
  }
}

// 使用例
Widget buildPlatformSpecificWidget() {
  if (PlatformUtils.isWeb) {
    return WebSpecificWidget();
  } else if (PlatformUtils.isAndroid) {
    return AndroidSpecificWidget();
  } else if (PlatformUtils.isIOS) {
    return IOSSpecificWidget();
  }
  return DefaultWidget();
}
```

## ビルドコマンド

### Android

```bash
# デバッグビルド
flutter build apk --debug

# リリースビルド（APK）
flutter build apk --release

# リリースビルド（AAB - Play Store用）
flutter build appbundle --release

# 分割APK（サイズ削減）
flutter build apk --split-per-abi --release

# フレーバー指定
flutter build apk --flavor prod --release
```

### iOS

```bash
# デバッグビルド
flutter build ios --debug

# リリースビルド
flutter build ios --release

# IPAファイル作成
flutter build ipa --release

# フレーバー指定
flutter build ios --flavor prod --release
```

### Web

```bash
# デバッグビルド
flutter build web --debug

# リリースビルド
flutter build web --release

# PWA対応
flutter build web --release --pwa-strategy offline-first

# HTML renderer指定
flutter build web --release --web-renderer canvaskit  # 高品質
flutter build web --release --web-renderer html  # 軽量
flutter build web --release --web-renderer auto  # 自動選択（推奨）
```

## プラットフォーム別のUI/UX

### Material vs Cupertino

```dart
import 'package:flutter/material.dart';
import 'package:flutter/cupertino.dart';

class AdaptiveButton extends StatelessWidget {
  final VoidCallback onPressed;
  final String text;

  const AdaptiveButton({
    required this.onPressed,
    required this.text,
  });

  @override
  Widget build(BuildContext context) {
    if (PlatformUtils.isIOS) {
      return CupertinoButton(
        onPressed: onPressed,
        child: Text(text),
      );
    }
    return ElevatedButton(
      onPressed: onPressed,
      child: Text(text),
    );
  }
}

// ダイアログ
void showAdaptiveDialog(BuildContext context) {
  if (PlatformUtils.isIOS) {
    showCupertinoDialog(
      context: context,
      builder: (context) => CupertinoAlertDialog(
        title: const Text('Title'),
        content: const Text('Content'),
        actions: [
          CupertinoDialogAction(
            child: const Text('Cancel'),
            onPressed: () => Navigator.pop(context),
          ),
          CupertinoDialogAction(
            child: const Text('OK'),
            onPressed: () => Navigator.pop(context),
          ),
        ],
      ),
    );
  } else {
    showDialog(
      context: context,
      builder: (context) => AlertDialog(
        title: const Text('Title'),
        content: const Text('Content'),
        actions: [
          TextButton(
            child: const Text('Cancel'),
            onPressed: () => Navigator.pop(context),
          ),
          TextButton(
            child: const Text('OK'),
            onPressed: () => Navigator.pop(context),
          ),
        ],
      ),
    );
  }
}
```

## チェックリスト

### Android実装時
- [ ] minSdk 24以上に設定
- [ ] MultiDex有効化
- [ ] ProGuard設定
- [ ] 権限の説明文を追加
- [ ] ディープリンク設定
- [ ] アプリアイコンを追加
- [ ] 署名鍵の設定

### iOS実装時
- [ ] 最小iOSバージョン13.0以上
- [ ] Info.plistに権限説明を追加
- [ ] ユニバーサルリンク設定
- [ ] CocoaPods最新化
- [ ] 証明書とプロビジョニングプロファイル
- [ ] アプリアイコンを追加

### Web実装時
- [ ] manifest.json設定
- [ ] PWA対応
- [ ] SEO対策（メタタグ）
- [ ] レスポンシブ対応
- [ ] Service Worker設定
- [ ] favicon設定

## 次のステップ

プラットフォーム固有実装を理解したら、実際のアプリに適用してみましょう。

---

**AI開発のヒント**:
生成AIに「Androidでプッシュ通知を実装して」と依頼する際は、この章の「Android固有実装 > プッシュ通知」セクションを参考資料として渡すと、正確な実装をしてくれます。
