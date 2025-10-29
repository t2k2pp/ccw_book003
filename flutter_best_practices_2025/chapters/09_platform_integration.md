# 第9章: Kotlin/Swiftとの連携とプラットフォーム固有実装

## この章で学べること

- Method Channel / Platform Channelの使い方
- FlutterとKotlin/Swiftの違い
- Pigeonによる型安全な通信
- プラットフォーム固有機能の実装

## FlutterとKotlin/Swiftの比較

### 言語レベルの違い

| 特徴 | Flutter/Dart | Kotlin | Swift |
|------|-------------|--------|-------|
| Null安全 | ✅ 言語レベル | ✅ 言語レベル | ✅ 言語レベル |
| 非同期 | Future/Stream | Coroutines/Flow | async/await |
| UI構築 | Widget Tree | XML/Compose | UIKit/SwiftUI |
| 型推論 | ✅ | ✅ | ✅ |
| 関数型プログラミング | ✅ | ✅ | ✅ |

### いつネイティブコードが必要か

| シナリオ | Flutter | ネイティブ |
|---------|---------|----------|
| 基本的なUI | ⭐⭐⭐⭐⭐ | ⭐⭐ |
| API通信 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| カメラ・GPS | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| Bluetooth | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| AR/VR | ⭐⭐ | ⭐⭐⭐⭐⭐ |
| ウィジェット（ホーム画面） | ⭐ | ⭐⭐⭐⭐⭐ |
| バックグラウンド処理 | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ |

## Platform Channel（Method Channel）

### 基本的な使い方

#### Flutter側（Dart）

```dart
import 'package:flutter/services.dart';

class BatteryService {
  static const platform = MethodChannel('com.example.app/battery');

  Future<int> getBatteryLevel() async {
    try {
      final int result = await platform.invokeMethod('getBatteryLevel');
      return result;
    } on PlatformException catch (e) {
      print("Failed to get battery level: '${e.message}'.");
      rethrow;
    }
  }
}
```

#### Android側（Kotlin）

```kotlin
// android/app/src/main/kotlin/com/example/app/MainActivity.kt
import io.flutter.embedding.android.FlutterActivity
import io.flutter.embedding.engine.FlutterEngine
import io.flutter.plugin.common.MethodChannel
import android.content.Context
import android.content.ContextWrapper
import android.content.Intent
import android.content.IntentFilter
import android.os.BatteryManager
import android.os.Build.VERSION
import android.os.Build.VERSION_CODES

class MainActivity: FlutterActivity() {
    private val CHANNEL = "com.example.app/battery"

    override fun configureFlutterEngine(flutterEngine: FlutterEngine) {
        super.configureFlutterEngine(flutterEngine)
        MethodChannel(flutterEngine.dartExecutor.binaryMessenger, CHANNEL).setMethodCallHandler {
            call, result ->
            if (call.method == "getBatteryLevel") {
                val batteryLevel = getBatteryLevel()

                if (batteryLevel != -1) {
                    result.success(batteryLevel)
                } else {
                    result.error("UNAVAILABLE", "Battery level not available.", null)
                }
            } else {
                result.notImplemented()
            }
        }
    }

    private fun getBatteryLevel(): Int {
        val batteryLevel: Int
        if (VERSION.SDK_INT >= VERSION_CODES.LOLLIPOP) {
            val batteryManager = getSystemService(Context.BATTERY_SERVICE) as BatteryManager
            batteryLevel = batteryManager.getIntProperty(BatteryManager.BATTERY_PROPERTY_CAPACITY)
        } else {
            val intent = ContextWrapper(applicationContext).registerReceiver(
                null, IntentFilter(Intent.ACTION_BATTERY_CHANGED)
            )
            batteryLevel = intent!!.getIntExtra(BatteryManager.EXTRA_LEVEL, -1) * 100 /
                    intent.getIntExtra(BatteryManager.EXTRA_SCALE, -1)
        }

        return batteryLevel
    }
}
```

#### iOS側（Swift）

```swift
// ios/Runner/AppDelegate.swift
import UIKit
import Flutter

@UIApplicationMain
@objc class AppDelegate: FlutterAppDelegate {
  override func application(
    _ application: UIApplication,
    didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
  ) -> Bool {
    let controller : FlutterViewController = window?.rootViewController as! FlutterViewController
    let batteryChannel = FlutterMethodChannel(
      name: "com.example.app/battery",
      binaryMessenger: controller.binaryMessenger
    )

    batteryChannel.setMethodCallHandler({
      (call: FlutterMethodCall, result: @escaping FlutterResult) -> Void in
      guard call.method == "getBatteryLevel" else {
        result(FlutterMethodNotImplemented)
        return
      }
      self.receiveBatteryLevel(result: result)
    })

    GeneratedPluginRegistrant.register(with: self)
    return super.application(application, didFinishLaunchingWithOptions: launchOptions)
  }

  private func receiveBatteryLevel(result: FlutterResult) {
    let device = UIDevice.current
    device.isBatteryMonitoringEnabled = true
    if device.batteryState == UIDevice.BatteryState.unknown {
      result(FlutterError(code: "UNAVAILABLE",
                          message: "Battery level not available.",
                          details: nil))
    } else {
      result(Int(device.batteryLevel * 100))
    }
  }
}
```

### 引数付きのメソッド呼び出し

#### Flutter側

```dart
class FileService {
  static const platform = MethodChannel('com.example.app/file');

  Future<String> saveFile(String content, String filename) async {
    try {
      final String path = await platform.invokeMethod('saveFile', {
        'content': content,
        'filename': filename,
      });
      return path;
    } on PlatformException catch (e) {
      throw Exception("Failed to save file: ${e.message}");
    }
  }
}
```

#### Android側（Kotlin）

```kotlin
MethodChannel(flutterEngine.dartExecutor.binaryMessenger, CHANNEL)
    .setMethodCallHandler { call, result ->
        when (call.method) {
            "saveFile" -> {
                val content = call.argument<String>("content")
                val filename = call.argument<String>("filename")

                if (content != null && filename != null) {
                    val path = saveFile(content, filename)
                    result.success(path)
                } else {
                    result.error("INVALID_ARGUMENT", "Missing arguments", null)
                }
            }
            else -> result.notImplemented()
        }
    }

private fun saveFile(content: String, filename: String): String {
    val file = File(filesDir, filename)
    file.writeText(content)
    return file.absolutePath
}
```

## Pigeon（型安全な通信）

### Pigeonとは

Google公式のコード生成ツール。型安全なPlatform Channel通信を実現。

### セットアップ

```yaml
dev_dependencies:
  pigeon: ^18.0.0
```

### APIの定義

```dart
// pigeons/messages.dart
import 'package:pigeon/pigeon.dart';

class UserData {
  String? id;
  String? name;
  String? email;
}

@HostApi()
abstract class UserApi {
  @async
  UserData getUser(String userId);

  @async
  void saveUser(UserData user);
}
```

### コード生成

```bash
flutter pub run pigeon \
  --input pigeons/messages.dart \
  --dart_out lib/pigeon_messages.dart \
  --kotlin_out android/app/src/main/kotlin/com/example/app/Messages.kt \
  --swift_out ios/Runner/Messages.swift
```

### Android実装（Kotlin）

```kotlin
class UserApiImpl : UserApi {
    override fun getUser(userId: String, result: Result<UserData>) {
        // データベースまたはAPIから取得
        val user = UserData().apply {
            id = userId
            name = "John Doe"
            email = "john@example.com"
        }
        result.success(user)
    }

    override fun saveUser(user: UserData, result: Result<Void>) {
        // データベースに保存
        result.success(null)
    }
}

// MainActivityでセットアップ
override fun configureFlutterEngine(flutterEngine: FlutterEngine) {
    super.configureFlutterEngine(flutterEngine)
    UserApi.setUp(flutterEngine.dartExecutor.binaryMessenger, UserApiImpl())
}
```

### Flutter側での使用

```dart
final api = UserApi();

// ユーザー取得
final user = await api.getUser('123');
print(user.name);

// ユーザー保存
await api.saveUser(UserData(
  id: '123',
  name: 'Jane Doe',
  email: 'jane@example.com',
));
```

## Event Channel（ストリーム通信）

### センサーデータのストリーム

#### Flutter側

```dart
class SensorService {
  static const EventChannel _eventChannel =
      EventChannel('com.example.app/sensor');

  Stream<Map<String, dynamic>> get sensorStream {
    return _eventChannel.receiveBroadcastStream().map((event) {
      return Map<String, dynamic>.from(event);
    });
  }
}

// 使用例
sensorService.sensorStream.listen((data) {
  print('X: ${data['x']}, Y: ${data['y']}, Z: ${data['z']}');
});
```

#### Android側（Kotlin）

```kotlin
import android.hardware.Sensor
import android.hardware.SensorEvent
import android.hardware.SensorEventListener
import android.hardware.SensorManager
import io.flutter.plugin.common.EventChannel

class SensorStreamHandler(private val sensorManager: SensorManager) :
    EventChannel.StreamHandler, SensorEventListener {

    private var events: EventChannel.EventSink? = null

    override fun onListen(arguments: Any?, events: EventChannel.EventSink?) {
        this.events = events
        val sensor = sensorManager.getDefaultSensor(Sensor.TYPE_ACCELEROMETER)
        sensorManager.registerListener(this, sensor, SensorManager.SENSOR_DELAY_NORMAL)
    }

    override fun onCancel(arguments: Any?) {
        sensorManager.unregisterListener(this)
        events = null
    }

    override fun onSensorChanged(event: SensorEvent?) {
        event?.let {
            val data = mapOf(
                "x" to it.values[0],
                "y" to it.values[1],
                "z" to it.values[2]
            )
            events?.success(data)
        }
    }

    override fun onAccuracyChanged(sensor: Sensor?, accuracy: Int) {}
}

// MainActivity
override fun configureFlutterEngine(flutterEngine: FlutterEngine) {
    super.configureFlutterEngine(flutterEngine)

    val sensorManager = getSystemService(Context.SENSOR_SERVICE) as SensorManager
    EventChannel(flutterEngine.dartExecutor.binaryMessenger, "com.example.app/sensor")
        .setStreamHandler(SensorStreamHandler(sensorManager))
}
```

## FFI（Foreign Function Interface）

### C/C++ライブラリの呼び出し

```yaml
dependencies:
  ffi: ^2.1.0
```

#### C側

```c
// native/calculator.c
int add(int a, int b) {
    return a + b;
}
```

#### Dart側

```dart
import 'dart:ffi' as ffi;
import 'dart:io' show Platform;

// C関数の型定義
typedef AddNative = ffi.Int32 Function(ffi.Int32 a, ffi.Int32 b);
typedef AddDart = int Function(int a, int b);

class Calculator {
  late final AddDart _add;

  Calculator() {
    // ネイティブライブラリを読み込み
    final dylib = Platform.isAndroid
        ? ffi.DynamicLibrary.open('libnative.so')
        : ffi.DynamicLibrary.process();

    // 関数をバインド
    _add = dylib.lookupFunction<AddNative, AddDart>('add');
  }

  int add(int a, int b) => _add(a, b);
}

// 使用例
final calculator = Calculator();
print(calculator.add(5, 3));  // 8
```

## プラットフォーム固有UIの統合

### Android View

```dart
// Flutterから AndroidViewを呼び出す
AndroidView(
  viewType: 'com.example.app/mapview',
  creationParams: <String, dynamic>{
    'latitude': 35.6895,
    'longitude': 139.6917,
  },
  creationParamsCodec: const StandardMessageCodec(),
)
```

### iOS UIView

```dart
UiKitView(
  viewType: 'com.example.app/mapview',
  creationParams: <String, dynamic>{
    'latitude': 35.6895,
    'longitude': 139.6917,
  },
  creationParamsCodec: const StandardMessageCodec(),
)
```

## ベストプラクティス

### 1. プラットフォームチェック

```dart
import 'dart:io' show Platform;

if (Platform.isAndroid) {
  // Android固有の処理
} else if (Platform.isIOS) {
  // iOS固有の処理
}
```

### 2. エラーハンドリング

```dart
Future<T> safePlatformCall<T>(Future<T> Function() call) async {
  try {
    return await call();
  } on PlatformException catch (e) {
    print('Platform error: ${e.code}, ${e.message}');
    rethrow;
  } catch (e) {
    print('Unknown error: $e');
    rethrow;
  }
}

// 使用例
final batteryLevel = await safePlatformCall(() =>
  platform.invokeMethod<int>('getBatteryLevel')
);
```

### 3. Method Channelの抽象化

```dart
abstract class PlatformService {
  Future<int> getBatteryLevel();
}

class PlatformServiceImpl implements PlatformService {
  static const platform = MethodChannel('com.example.app/battery');

  @override
  Future<int> getBatteryLevel() async {
    return await platform.invokeMethod<int>('getBatteryLevel') ?? 0;
  }
}

// Riverpodでの提供
@riverpod
PlatformService platformService(PlatformServiceRef ref) {
  return PlatformServiceImpl();
}
```

## AI開発時の注意点

### AIが間違いやすいパターン

#### 1. プラットフォーム固有の API を考慮しない

**問題**:
```dart
// iOSには存在しないAndroid APIを使おうとする
```

**対策**:
```dart
if (Platform.isAndroid) {
  // Android専用API
} else if (Platform.isIOS) {
  // iOS専用API
}
```

#### 2. 古いMethod Channel記法

**AIの出力（古い）**:
```dart
// 型指定なし
final result = await platform.invokeMethod('method');
```

**推奨（2025年）**:
```dart
// 型を明示
final result = await platform.invokeMethod<int>('method');
```

#### 3. エラーハンドリングの欠如

**AIの出力**:
```dart
final result = await platform.invokeMethod('method');
```

**推奨**:
```dart
try {
  final result = await platform.invokeMethod('method');
} on PlatformException catch (e) {
  // エラー処理
}
```

## チェックリスト

### Method Channel実装時
- [ ] チャネル名を定義
- [ ] Flutter側でMethodChannelを作成
- [ ] Android（Kotlin）側で実装
- [ ] iOS（Swift）側で実装
- [ ] エラーハンドリング
- [ ] 型安全性の確保

### Pigeon使用時
- [ ] APIを定義（pigeons/*.dart）
- [ ] コード生成を実行
- [ ] Android/iOS側で実装
- [ ] テスト

## 次のステップ

プラットフォーム連携を学んだら、[第10章: CI/CDとデプロイメント](10_ci_cd.md)で自動化を学びましょう。

---

**AI開発のヒント**:
生成AIに「ネイティブコードと連携するコードを書いて」と依頼する際は、この章の「Platform Channel」セクションを一緒に渡すと、正確な実装をしてくれます。
