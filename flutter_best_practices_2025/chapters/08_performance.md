# 第8章: パフォーマンス最適化

## この章で学べること

- ビルド最適化テクニック
- メモリリーク対策
- 画像最適化
- DevToolsを使ったパフォーマンス分析

## パフォーマンス分析ツール

### Flutter DevTools

```bash
# DevToolsを起動
flutter pub global activate devtools
flutter pub global run devtools

# アプリ実行中にDevToolsを開く
flutter run --profile
```

**主な機能**:
- **Performance**: フレームレート、ビルド時間の計測
- **Memory**: メモリ使用量、リークの検出
- **Network**: API通信の監視
- **Debugger**: ブレークポイント、変数確認

## ビルド最適化

### 1. const コンストラクタの活用

**NG（非効率）**:
```dart
Column(
  children: [
    Text('Hello'),  // 毎回再ビルド
    SizedBox(height: 16),  // 毎回再ビルド
  ],
)
```

**OK（最適化）**:
```dart
Column(
  children: [
    const Text('Hello'),  // 再ビルドされない
    const SizedBox(height: 16),  // 再ビルドされない
  ],
)
```

**効果**: 不要な再ビルドを防ぎ、パフォーマンス向上

### 2. Widgetの分割

**NG（大きすぎるWidget）**:
```dart
class HomeScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Column(
        children: [
          // 100行以上のWidget定義...
          AppBar(...),
          Body(...),
          Footer(...),
        ],
      ),
    );
  }
}
```

**OK（分割）**:
```dart
class HomeScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return const Scaffold(
      body: Column(
        children: [
          _AppBar(),
          _Body(),
          _Footer(),
        ],
      ),
    );
  }
}

class _AppBar extends StatelessWidget {
  const _AppBar();

  @override
  Widget build(BuildContext context) {
    // ...
  }
}
```

**効果**: 必要な部分だけ再ビルドされる

### 3. ConsumerWidget vs Consumer

**全体が再ビルドされる**:
```dart
class UserScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final user = ref.watch(userProvider);

    return Scaffold(
      appBar: AppBar(title: Text(user.name)),  // 再ビルド
      body: Column(
        children: [
          Text('Welcome'),  // 再ビルド（不要）
          UserProfile(user: user),  // 再ビルド
        ],
      ),
    );
  }
}
```

**必要な部分だけ再ビルド**:
```dart
class UserScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Consumer(
          builder: (context, ref, child) {
            final user = ref.watch(userProvider);
            return Text(user.name);
          },
        ),
      ),
      body: Column(
        children: [
          const Text('Welcome'),  // 再ビルドされない
          Consumer(
            builder: (context, ref, child) {
              final user = ref.watch(userProvider);
              return UserProfile(user: user);
            },
          ),
        ],
      ),
    );
  }
}
```

### 4. Selectorsで部分購読

**全体が再ビルドされる**:
```dart
final user = ref.watch(userProvider);
return Text(user.name);  // userの他のフィールドが変わっても再ビルド
```

**必要な部分だけ購読**:
```dart
final userName = ref.watch(userProvider.select((user) => user.name));
return Text(userName);  // nameが変わった時だけ再ビルド
```

### 5. ListView.builder の使用

**NG（全アイテムを一度に生成）**:
```dart
ListView(
  children: items.map((item) => ItemWidget(item)).toList(),
)
```

**OK（スクロール位置に応じて生成）**:
```dart
ListView.builder(
  itemCount: items.length,
  itemBuilder: (context, index) {
    return ItemWidget(items[index]);
  },
)
```

**効果**: 表示領域外のWidgetは生成されない

### 6. AutomaticKeepAliveClientMixin

**タブ切り替えで状態が失われる**:
```dart
class TabContent extends StatefulWidget {
  @override
  State<TabContent> createState() => _TabContentState();
}

class _TabContentState extends State<TabContent> {
  // タブを切り替えると破棄される
}
```

**状態を保持**:
```dart
class _TabContentState extends State<TabContent>
    with AutomaticKeepAliveClientMixin {
  @override
  bool get wantKeepAlive => true;

  @override
  Widget build(BuildContext context) {
    super.build(context);  // 必須
    // ...
  }
}
```

## メモリ管理

### 1. StreamSubscriptionの適切な破棄

**NG（メモリリーク）**:
```dart
class MyWidget extends StatefulWidget {
  @override
  State<MyWidget> createState() => _MyWidgetState();
}

class _MyWidgetState extends State<MyWidget> {
  late StreamSubscription _subscription;

  @override
  void initState() {
    super.initState();
    _subscription = someStream.listen((data) {
      // ...
    });
  }
  // disposeでcancelしていない → メモリリーク
}
```

**OK（適切な破棄）**:
```dart
class _MyWidgetState extends State<MyWidget> {
  late StreamSubscription _subscription;

  @override
  void initState() {
    super.initState();
    _subscription = someStream.listen((data) {
      // ...
    });
  }

  @override
  void dispose() {
    _subscription.cancel();  // 破棄
    super.dispose();
  }
}
```

**Riverpodでの自動破棄**:
```dart
@riverpod
Stream<Data> dataStream(DataStreamRef ref) {
  final stream = someApi.watchData();
  // refが破棄されると自動でStreamもキャンセルされる
  return stream;
}
```

### 2. Controllerの破棄

**NG**:
```dart
class MyWidget extends StatefulWidget {
  @override
  State<MyWidget> createState() => _MyWidgetState();
}

class _MyWidgetState extends State<MyWidget> {
  final _controller = TextEditingController();
  // disposeしていない
}
```

**OK**:
```dart
class _MyWidgetState extends State<MyWidget> {
  final _controller = TextEditingController();

  @override
  void dispose() {
    _controller.dispose();  // 破棄
    super.dispose();
  }
}
```

### 3. 大きな画像の扱い

**NG（メモリを大量消費）**:
```dart
Image.asset('assets/large_image.jpg')  // 5MB
```

**OK（サイズを指定して読み込み）**:
```dart
Image.asset(
  'assets/large_image.jpg',
  cacheWidth: 500,  // 幅500pxで読み込み
  cacheHeight: 500,
)
```

## 画像最適化

### 1. 適切な画像フォーマット

| 用途 | 推奨フォーマット | 理由 |
|------|----------------|------|
| 写真 | JPEG | サイズが小さい |
| アイコン・ロゴ | SVG | 拡大しても綺麗 |
| 透過が必要 | PNG | 透過対応 |
| アニメーション | WebP | サイズ小・高画質 |

### 2. cached_network_image の使用

```yaml
dependencies:
  cached_network_image: ^3.3.0
```

```dart
CachedNetworkImage(
  imageUrl: 'https://example.com/image.jpg',
  placeholder: (context, url) => const CircularProgressIndicator(),
  errorWidget: (context, url, error) => const Icon(Icons.error),
  fadeInDuration: const Duration(milliseconds: 300),
  memCacheWidth: 500,  // メモリキャッシュサイズ
)
```

**効果**:
- ネットワークから一度だけ取得
- ディスクにキャッシュ
- メモリにもキャッシュ

### 3. 画像の遅延読み込み

```dart
ListView.builder(
  itemCount: images.length,
  itemBuilder: (context, index) {
    return CachedNetworkImage(
      imageUrl: images[index],
      // スクロールして表示領域に入ったら読み込まれる
    );
  },
)
```

## アニメーション最適化

### 1. RepaintBoundaryの使用

**NG（全体が再描画）**:
```dart
Stack(
  children: [
    ComplexBackground(),  // 毎フレーム再描画
    AnimatedWidget(),  // アニメーション
  ],
)
```

**OK（背景は再描画しない）**:
```dart
Stack(
  children: [
    RepaintBoundary(
      child: ComplexBackground(),  // 一度だけ描画
    ),
    AnimatedWidget(),
  ],
)
```

### 2. AnimatedBuilderの使用

**NG（毎フレーム全体が再ビルド）**:
```dart
class AnimatedContainer extends StatefulWidget {
  @override
  State<AnimatedContainer> createState() => _AnimatedContainerState();
}

class _AnimatedContainerState extends State<AnimatedContainer>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Transform.rotate(
          angle: _controller.value,
          child: Icon(Icons.star),
        ),
        Text('Static text'),  // 毎フレーム再ビルド（不要）
      ],
    );
  }
}
```

**OK（必要な部分だけ再ビルド）**:
```dart
@override
Widget build(BuildContext context) {
  return Column(
    children: [
      AnimatedBuilder(
        animation: _controller,
        builder: (context, child) {
          return Transform.rotate(
            angle: _controller.value,
            child: child,
          );
        },
        child: const Icon(Icons.star),  // 一度だけビルド
      ),
      const Text('Static text'),  // 再ビルドされない
    ],
  );
}
```

## 計算量の最適化

### 1. 高コストな処理を避ける

**NG（build内で重い処理）**:
```dart
@override
Widget build(BuildContext context) {
  final sortedList = heavySort(items);  // 毎回ソート
  return ListView(
    children: sortedList.map((item) => ItemWidget(item)).toList(),
  );
}
```

**OK（事前に計算）**:
```dart
@riverpod
List<Item> sortedItems(SortedItemsRef ref) {
  final items = ref.watch(itemsProvider);
  return heavySort(items);  // Providerが変わった時だけソート
}

@override
Widget build(BuildContext context, WidgetRef ref) {
  final sortedList = ref.watch(sortedItemsProvider);
  return ListView(
    children: sortedList.map((item) => ItemWidget(item)).toList(),
  );
}
```

### 2. Isolateで重い処理を分離

```dart
Future<List<Item>> processLargeData(List<RawData> rawData) async {
  return await compute(_processData, rawData);
}

List<Item> _processData(List<RawData> rawData) {
  // 重い処理（別スレッドで実行）
  return rawData.map((data) => Item.fromRaw(data)).toList();
}
```

## ビルド時間の短縮

### 1. 不要なパッケージの削除

```bash
# 未使用の依存関係を確認
flutter pub deps

# 未使用のimportを削除
dart fix --apply
```

### 2. ビルド設定の最適化

```gradle
// android/app/build.gradle

android {
    // ...

    buildTypes {
        release {
            // コードの難読化・最適化
            minifyEnabled true
            shrinkResources true

            // R8による最適化
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }
}
```

### 3. アプリサイズの削減

```bash
# アプリサイズを分析
flutter build apk --analyze-size

# APKを分割（アーキテクチャ別）
flutter build apk --split-per-abi
```

## DevToolsでのパフォーマンス分析

### 1. パフォーマンスタブの見方

**主要指標**:
- **FPS**: 60fps（16.67ms/フレーム）を維持
- **Build**: Widgetビルド時間
- **Layout**: レイアウト計算時間
- **Paint**: 描画時間

**問題の特定**:
```
Frame time > 16.67ms → ジャンク（カクつき）
Build time が長い → Widgetの最適化が必要
Paint time が長い → 描画の最適化が必要
```

### 2. メモリタブの見方

**リークの検出**:
1. アプリを操作
2. GCボタンをクリック
3. メモリが減らない → リーク疑い
4. スナップショットを取得して分析

### 3. Timeline

```dart
// カスタムタイムラインイベント
import 'dart:developer' as developer;

void heavyOperation() {
  developer.Timeline.startSync('heavyOperation');
  // 重い処理
  developer.Timeline.finishSync();
}
```

## ベストプラクティス

### パフォーマンスチェックリスト

#### ビルド最適化
- [ ] constコンストラクタを使用
- [ ] Widgetを適切に分割
- [ ] 不要な再ビルドを防ぐ
- [ ] ListView.builderを使用

#### メモリ管理
- [ ] StreamSubscriptionを破棄
- [ ] Controllerを破棄
- [ ] 大きな画像はサイズ指定して読み込み
- [ ] キャッシュを活用

#### 画像
- [ ] 適切なフォーマットを使用
- [ ] cached_network_imageを使用
- [ ] 遅延読み込みを実装

#### アニメーション
- [ ] RepaintBoundaryを使用
- [ ] AnimatedBuilderを使用

#### その他
- [ ] 重い処理はIsolateで実行
- [ ] DevToolsで定期的に分析
- [ ] リリースビルドで検証

## AI開発時の注意点

### AIが生成しがちな非効率なコード

#### 1. constを忘れる

**AIの出力**:
```dart
SizedBox(height: 16)
```

**最適化**:
```dart
const SizedBox(height: 16)
```

#### 2. 不要な再ビルド

**AIの出力**:
```dart
ref.watch(userProvider);  // Widgetの最上位
```

**最適化**:
```dart
ref.watch(userProvider.select((user) => user.name));
```

#### 3. ListView vs ListView.builder

**AIの出力**:
```dart
ListView(children: items.map(...).toList())
```

**最適化**:
```dart
ListView.builder(itemCount: items.length, itemBuilder: ...)
```

## 次のステップ

パフォーマンス最適化を学んだら、[第9章: Kotlin/Swiftとの連携とプラットフォーム固有実装](09_platform_integration.md)でネイティブコードとの連携を学びましょう。

---

**AI開発のヒント**:
生成AIに「パフォーマンスを最適化して」と依頼する際は、この章の「ビルド最適化」セクションを一緒に渡すと、適切な最適化を提案してくれます。
