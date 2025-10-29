# 第6章: データ永続化とAPI連携

## この章で学べること

- Drift（SQLite）でのローカルデータベース
- Isar（NoSQL）の使い方
- Dioを使ったHTTP通信
- オフライン対応とキャッシュ戦略

## ローカルデータベース

### Drift（推奨：SQLite）

#### セットアップ

```yaml
dependencies:
  drift: ^2.18.0
  sqlite3_flutter_libs: ^0.5.0
  path_provider: ^2.1.0
  path: ^1.9.0

dev_dependencies:
  drift_dev: ^2.18.0
  build_runner: ^2.4.0
```

#### テーブル定義

```dart
import 'package:drift/drift.dart';

part 'database.g.dart';

class Todos extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get title => text().withLength(min: 1, max: 100)();
  TextColumn get content => text().nullable()();
  BoolColumn get isCompleted => boolean().withDefault(const Constant(false))();
  DateTimeColumn get createdAt => dateTime().withDefault(currentDateAndTime)();
}

@DriftDatabase(tables: [Todos])
class AppDatabase extends _$AppDatabase {
  AppDatabase() : super(_openConnection());

  @override
  int get schemaVersion => 1;

  // CRUDメソッド
  Future<List<Todo>> getAllTodos() => select(todos).get();

  Stream<List<Todo>> watchAllTodos() => select(todos).watch();

  Future<int> insertTodo(TodosCompanion todo) => into(todos).insert(todo);

  Future<bool> updateTodo(Todo todo) => update(todos).replace(todo);

  Future<int> deleteTodo(int id) =>
      (delete(todos)..where((t) => t.id.equals(id))).go();
}

LazyDatabase _openConnection() {
  return LazyDatabase(() async {
    final dbFolder = await getApplicationDocumentsDirectory();
    final file = File(p.join(dbFolder.path, 'db.sqlite'));
    return NativeDatabase.createInBackground(file);
  });
}
```

#### Riverpodでの統合

```dart
@riverpod
AppDatabase appDatabase(AppDatabaseRef ref) {
  final database = AppDatabase();
  ref.onDispose(() => database.close());
  return database;
}

@riverpod
Stream<List<Todo>> todos(TodosRef ref) {
  final database = ref.watch(appDatabaseProvider);
  return database.watchAllTodos();
}

@riverpod
class TodoOperations extends _$TodoOperations {
  @override
  FutureOr<void> build() {}

  Future<void> addTodo(String title, String? content) async {
    final database = ref.read(appDatabaseProvider);
    await database.insertTodo(
      TodosCompanion.insert(
        title: title,
        content: Value(content),
      ),
    );
    ref.invalidate(todosProvider);
  }

  Future<void> toggleTodo(Todo todo) async {
    final database = ref.read(appDatabaseProvider);
    await database.updateTodo(
      todo.copyWith(isCompleted: !todo.isCompleted),
    );
  }
}
```

### Isar（高速NoSQL）

#### セットアップ

```yaml
dependencies:
  isar: ^3.1.0
  isar_flutter_libs: ^3.1.0
  path_provider: ^2.1.0

dev_dependencies:
  isar_generator: ^3.1.0
  build_runner: ^2.4.0
```

#### モデル定義

```dart
import 'package:isar/isar.dart';

part 'user.g.dart';

@collection
class User {
  Id id = Isar.autoIncrement;

  @Index(type: IndexType.value)
  late String email;

  late String name;

  late DateTime createdAt;

  // リレーション
  final posts = IsarLinks<Post>();
}

@collection
class Post {
  Id id = Isar.autoIncrement;

  late String title;
  late String content;
  late DateTime publishedAt;

  @Backlink(to: 'posts')
  final author = IsarLink<User>();
}
```

#### データ操作

```dart
@riverpod
Future<Isar> isar(IsarRef ref) async {
  final dir = await getApplicationDocumentsDirectory();
  final isar = await Isar.open(
    [UserSchema, PostSchema],
    directory: dir.path,
  );
  ref.onDispose(() => isar.close());
  return isar;
}

@riverpod
class UserRepository extends _$UserRepository {
  @override
  FutureOr<void> build() {}

  Future<void> addUser(User user) async {
    final isar = await ref.read(isarProvider.future);
    await isar.writeTxn(() async {
      await isar.users.put(user);
    });
  }

  Future<List<User>> getAllUsers() async {
    final isar = await ref.read(isarProvider.future);
    return await isar.users.where().findAll();
  }

  Stream<List<User>> watchUsers() async* {
    final isar = await ref.read(isarProvider.future);
    yield* isar.users.where().watch(fireImmediately: true);
  }

  Future<User?> getUserByEmail(String email) async {
    final isar = await ref.read(isarProvider.future);
    return await isar.users.filter().emailEqualTo(email).findFirst();
  }
}
```

## HTTP通信

### Dioのセットアップ

```yaml
dependencies:
  dio: ^5.4.0
  retrofit: ^4.1.0
  json_annotation: ^4.9.0

dev_dependencies:
  retrofit_generator: ^8.1.0
  json_serializable: ^6.8.0
  build_runner: ^2.4.0
```

### Dio インスタンスの作成

```dart
@riverpod
Dio dio(DioRef ref) {
  final dio = Dio(BaseOptions(
    baseUrl: 'https://api.example.com',
    connectTimeout: const Duration(seconds: 5),
    receiveTimeout: const Duration(seconds: 3),
  ));

  // ロギング（開発時のみ）
  dio.interceptors.add(LogInterceptor(
    request: true,
    requestBody: true,
    responseBody: true,
    error: true,
  ));

  // 認証トークン
  dio.interceptors.add(InterceptorsWrapper(
    onRequest: (options, handler) async {
      final token = await ref.read(authTokenProvider.future);
      if (token != null) {
        options.headers['Authorization'] = 'Bearer $token';
      }
      handler.next(options);
    },
    onError: (error, handler) async {
      // 401エラー時に自動リフレッシュ
      if (error.response?.statusCode == 401) {
        // トークンリフレッシュ処理
        try {
          await ref.read(authNotifierProvider.notifier).refreshToken();
          // リトライ
          final response = await dio.fetch(error.requestOptions);
          handler.resolve(response);
        } catch (e) {
          handler.next(error);
        }
      } else {
        handler.next(error);
      }
    },
  ));

  return dio;
}
```

### Retrofit APIクライアント

```dart
import 'package:dio/dio.dart';
import 'package:retrofit/retrofit.dart';

part 'api_client.g.dart';

@RestApi(baseUrl: 'https://api.example.com')
abstract class ApiClient {
  factory ApiClient(Dio dio, {String baseUrl}) = _ApiClient;

  @GET('/users')
  Future<List<User>> getUsers();

  @GET('/users/{id}')
  Future<User> getUserById(@Path('id') String id);

  @POST('/users')
  Future<User> createUser(@Body() User user);

  @PUT('/users/{id}')
  Future<User> updateUser(
    @Path('id') String id,
    @Body() User user,
  );

  @DELETE('/users/{id}')
  Future<void> deleteUser(@Path('id') String id);

  @GET('/posts')
  Future<List<Post>> getPosts({
    @Query('page') int? page,
    @Query('limit') int? limit,
  });
}

@riverpod
ApiClient apiClient(ApiClientRef ref) {
  final dio = ref.watch(dioProvider);
  return ApiClient(dio);
}
```

## リポジトリパターン

### データソースの抽象化

```dart
// domain/repositories/user_repository.dart
abstract class UserRepository {
  Future<List<User>> getUsers();
  Future<User> getUserById(String id);
  Future<void> saveUser(User user);
  Future<void> deleteUser(String id);
}

// data/repositories/user_repository_impl.dart
class UserRepositoryImpl implements UserRepository {
  final ApiClient _apiClient;
  final AppDatabase _database;
  final NetworkInfo _networkInfo;

  UserRepositoryImpl(this._apiClient, this._database, this._networkInfo);

  @override
  Future<List<User>> getUsers() async {
    if (await _networkInfo.isConnected) {
      try {
        // オンライン：APIから取得
        final users = await _apiClient.getUsers();
        // キャッシュに保存
        await _cacheUsers(users);
        return users;
      } catch (e) {
        // API失敗時はキャッシュから取得
        return await _getCachedUsers();
      }
    } else {
      // オフライン：キャッシュから取得
      return await _getCachedUsers();
    }
  }

  Future<void> _cacheUsers(List<User> users) async {
    // Driftへの保存処理
  }

  Future<List<User>> _getCachedUsers() async {
    // Driftからの取得処理
  }
}

@riverpod
UserRepository userRepository(UserRepositoryRef ref) {
  return UserRepositoryImpl(
    ref.watch(apiClientProvider),
    ref.watch(appDatabaseProvider),
    ref.watch(networkInfoProvider),
  );
}
```

## オフライン対応

### ネットワーク状態の監視

```yaml
dependencies:
  connectivity_plus: ^6.0.0
```

```dart
@riverpod
class NetworkInfo extends _$NetworkInfo {
  StreamSubscription? _subscription;

  @override
  bool build() {
    _subscription = Connectivity().onConnectivityChanged.listen((result) {
      state = result != ConnectivityResult.none;
    });

    ref.onDispose(() => _subscription?.cancel());

    return true; // 初期値
  }

  Future<bool> get isConnected async {
    final result = await Connectivity().checkConnectivity();
    return result != ConnectivityResult.none;
  }
}
```

### キャッシュ戦略

#### 1. Cache-First（キャッシュ優先）

```dart
Future<User> getUser(String id) async {
  // まずキャッシュを確認
  final cached = await _getCachedUser(id);
  if (cached != null) {
    // バックグラウンドで更新
    _updateUserInBackground(id);
    return cached;
  }

  // キャッシュがなければAPIから取得
  return await _fetchAndCacheUser(id);
}
```

#### 2. Network-First（ネットワーク優先）

```dart
Future<User> getUser(String id) async {
  try {
    // まずAPIから取得を試みる
    final user = await _apiClient.getUserById(id);
    await _cacheUser(user);
    return user;
  } catch (e) {
    // 失敗時はキャッシュから
    final cached = await _getCachedUser(id);
    if (cached != null) return cached;
    rethrow;
  }
}
```

#### 3. Stale-While-Revalidate

```dart
Stream<User> watchUser(String id) async* {
  // すぐにキャッシュを返す
  final cached = await _getCachedUser(id);
  if (cached != null) {
    yield cached;
  }

  // その後、最新データを取得して返す
  try {
    final fresh = await _apiClient.getUserById(id);
    await _cacheUser(fresh);
    yield fresh;
  } catch (e) {
    // エラー時はキャッシュのまま
  }
}
```

## 同期処理（オフライン時の操作）

### 操作キューの実装

```dart
@collection
class SyncOperation {
  Id id = Isar.autoIncrement;

  @enumerated
  late OperationType type; // CREATE, UPDATE, DELETE

  late String entityType; // 'user', 'post', etc.
  late String entityId;
  late String data; // JSONシリアライズされたデータ

  late DateTime createdAt;
}

@riverpod
class SyncManager extends _$SyncManager {
  @override
  FutureOr<void> build() {}

  Future<void> queueOperation(
    OperationType type,
    String entityType,
    String entityId,
    Map<String, dynamic> data,
  ) async {
    final isar = await ref.read(isarProvider.future);
    await isar.writeTxn(() async {
      await isar.syncOperations.put(
        SyncOperation()
          ..type = type
          ..entityType = entityType
          ..entityId = entityId
          ..data = jsonEncode(data)
          ..createdAt = DateTime.now(),
      );
    });
  }

  Future<void> syncAll() async {
    if (!await ref.read(networkInfoProvider.future)) {
      return; // オフライン時は同期しない
    }

    final isar = await ref.read(isarProvider.future);
    final operations = await isar.syncOperations
        .where()
        .sortByCreatedAt()
        .findAll();

    for (final operation in operations) {
      try {
        await _executeOperation(operation);
        // 成功したら削除
        await isar.writeTxn(() async {
          await isar.syncOperations.delete(operation.id);
        });
      } catch (e) {
        // エラー処理
        print('Sync failed: $e');
      }
    }
  }

  Future<void> _executeOperation(SyncOperation operation) async {
    // 操作を実際に実行
  }
}
```

## エラーハンドリング

### カスタム例外

```dart
abstract class AppException implements Exception {
  final String message;
  const AppException(this.message);
}

class NetworkException extends AppException {
  const NetworkException([String message = 'Network error']) : super(message);
}

class ServerException extends AppException {
  final int statusCode;
  const ServerException(this.statusCode, [String message = 'Server error'])
      : super(message);
}

class CacheException extends AppException {
  const CacheException([String message = 'Cache error']) : super(message);
}
```

### エラーハンドリング

```dart
Future<Result<User>> getUser(String id) async {
  try {
    final user = await _apiClient.getUserById(id);
    return Result.success(user);
  } on DioException catch (e) {
    if (e.type == DioExceptionType.connectionTimeout) {
      return Result.failure(NetworkException('Connection timeout'));
    } else if (e.response != null) {
      return Result.failure(
        ServerException(e.response!.statusCode!, e.response!.statusMessage),
      );
    } else {
      return Result.failure(NetworkException());
    }
  } catch (e) {
    return Result.failure(AppException('Unknown error: $e'));
  }
}
```

## AI開発時の注意点

### AIが間違いやすいパターン

#### 1. sqfliteを直接使う

**NG**:
```dart
final db = await openDatabase('my_db.db');
await db.insert('users', user.toMap());
```

**OK（2025年推奨）**:
```dart
// Driftを使う
await database.insertUser(user);
```

#### 2. エラーハンドリングの欠如

**NG**:
```dart
final response = await dio.get('/users');
return response.data;
```

**OK**:
```dart
try {
  final response = await dio.get('/users');
  return Result.success(response.data);
} on DioException catch (e) {
  return Result.failure(NetworkException(e.message));
}
```

## チェックリスト

### データベース実装時
- [ ] DriftまたはIsarを選択
- [ ] テーブル/コレクション定義
- [ ] CRUD操作の実装
- [ ] マイグレーション戦略（Driftの場合）

### API連携実装時
- [ ] Dioインスタンスをセットアップ
- [ ] Retrofitでエンドポイント定義
- [ ] 認証トークンのインターセプター
- [ ] エラーハンドリング

### オフライン対応時
- [ ] ネットワーク状態の監視
- [ ] キャッシュ戦略の選択
- [ ] 同期処理の実装（必要に応じて）

## 次のステップ

データ層の実装ができたら、[第7章: テスト戦略](07_testing.md)でテストの書き方を学びましょう。

---

**AI開発のヒント**:
生成AIに「DriftでTodoデータベースを実装して」と依頼する際は、この章の「Drift」セクションを一緒に渡すと、正確な実装をしてくれます。
