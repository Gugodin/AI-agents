---
name: blueprint-flutter-clean
description: Blueprint de arquitectura Flutter Clean Architecture - Estructura completa del proyecto AppCobranza con DataState, Freezed, BLoC, GetIt y más
---

# Blueprint: Arquitectura Flutter

> Documento de especificación técnica para replicar el boilerplate de este proyecto desde cero.

---

## 1. Stack Tecnológico

| Categoría | Librería / Tool | Versión |
|---|---|---|
| State Management | `flutter_bloc` | ^9.1.1 |
| Inyección de Dependencias | `get_it` | ^9.2.0 |
| HTTP Client | `dio` | ^5.9.1 |
| API Codegen | `retrofit` | ^4.9.2 |
| Code Generation | `build_runner`, `freezed`, `json_serializable`, `retrofit_generator` | latest |
| Inmutabilidad / Union Types | `freezed_annotation` | ^3.1.0 |
| Type Safety Serialization | `json_annotation` | ^4.9.0 |
| Concurrencia BLoC | `bloc_concurrency` | ^0.3.0 |
| Reactive Streams | `rxdart` | ^0.28.0 |
| Permisos | `permission_handler` | ^12.0.1 |
| Almacenamiento seguro | `flutter_secure_storage` | ^9.2.2 |
| Preferencias | `shared_preferences` | ^2.2.3 |
| JWT | `dart_jsonwebtoken` | ^3.3.1 |
| Auth biométrica | `local_auth` | ^3.0.0 |
| Connectivity | `connectivity_plus` | ^7.0.0 |
| Logging | `logger` | ^2.6.0 |
| Animaciones | `animate_do`, `lottie` | latest |
| Skeletons/Shimmer | `skeletonizer`, `shimmer` | latest |
| Toasts | `toastification` | ^3.0.3 |
| Asset Generation | `flutter_gen_runner` | ^5.12.0 |
| Testing Mocks | `mocktail` | ^1.0.4 |
| SDK mínimo | Dart `>=3.3.0 <4.0.0` | — |

---

## 2. Arquitectura de Carpetas

```
lib/
├── main.dart                         # Entry point: setupInjector(), BlocObserver, runApp()
├── app/
│   └── {nombre_del_proyecto}_app.dart                 # MaterialApp: theme, routes, onGenerateRoute
├── gen/
│   └── assets.gen.dart               # Auto-generado por flutter_gen
├── config/
│   ├── config.dart                   # Barrel export
│   ├── routes/
│   │   └── routes.dart               # Clase Routes con static constants + onGenerateRoute()
│   ├── theme/
│   │   ├── app_colors.dart           # Paleta de colores de la app
│   │   ├── app_theme.dart            # ThemeData
│   │   └── theme.dart                # Barrel export
│   └── utils/                        # Utilidades de configuración
│
├── core/
│   ├── core.dart                     # Barrel export global del core
│   ├── components/                   # Widgets reutilizables globales
│   ├── constants/                     # ApiConstants (baseUrl, tokens, timeouts)
│   ├── contracts/                     # Interfaces abstractas (solo archivos .dart)
│   │   └── toast_contract.dart       # Interfaz abstracta de Toast
│   ├── events/                        # ConnectivityEventBus (Singleton, RxDart)
│   │   └── connectivity_event_bus.dart
│   ├── foundation/
│   │   ├── foundation.dart           # Barrel export
│   │   ├── bloc_observer/
│   │   │   └── app_bloc_observer.dart
│   │   ├── data_state/
│   │   │   ├── data_state.dart       # Sealed class DataState<T> con Freezed + Extensions
│   │   │   └── data_state.freezed.dart
│   │   └── use_case/
│   │       └── use_case.dart        # Los 4 tipos de UseCase (ver sección 4)
│   ├── helpers/
│   │   ├── helpers.dart              # Barrel export
│   │   ├── biometrics_helper.dart    # Singleton
│   │   ├── connectivity_helper.dart  # Singleton
│   │   ├── image_selector_helper.dart
│   │   ├── location_helper.dart      # Singleton
│   │   ├── modals_helper.dart
│   │   ├── permission_helper.dart    # Singleton (PermissionsHelper)
│   │   ├── secure_storage_helper.dart # Singleton
│   │   ├── shared_preference_helper.dart # Singleton
│   │   └── toast_helper.dart        # Implementación de ToastContract
│   ├── network/
│   │   ├── network.dart              # Barrel export
│   │   ├── clients/
│   │   │   ├── clients.dart
│   │   │   ├── client.dart          # Dio → Middleware API (Ejemplo de uso)
│   │   │   └── client_direct.dart    # Dio → Backend directo (Ejemplo de uso)
│   │   ├── interceptors/
│   │   │   ├── interceptors.dart
│   │   │   ├── connectivity_interceptor.dart
│   │   │   ├── token_interceptor.dart
│   │   │   └── logger_interceptor.dart
│   │   └── response_model/
│   │   │       └── response_model.dart   # Wrapper genérico de respuesta API
│   └── utils/
│       ├── utils.dart
│       ├── formatters/
│       ├── types/
│       │   └── typedef.dart          # DataResult<T>, DataResultVoid
│       └── validators/
│
├── injector/
│   ├── injector.dart                 # setupInjector() — registra todo en GetIt
│   └── dependencies/
│       ├── dependencies.dart         # Barrel export
│       ├── auth_dependencies.dart
│       ├── asignations_dependencies.dart
│       ├── catalog_dependencies.dart
│       ├── gestions_dependencies.dart
│       ├── location_dependencies.dart
│       ├── mapping_services_dependencies.dart
│       ├── payments_dependencies.dart
│       └── route_dependencies.dart
│
└── features/
    ├── features.dart                 # Barrel export global de features
    ├── auth/
    ├── home/
    ├── overlay/
    ├── catalogs/
    ├── location/
    ├── route/
    ├── mapping_services/
    ├── gestions/
    ├── asignations/
    └── payments/
        ├── payments.dart
        ├── data/
        ├── domain/
        └── presentation/

assets/
├── icons/
├── images/
└── lotties/
    └── loading_animated.json
```

---

## 3. Core Foundation

### 3.1 DataState<T> con Freezed

```dart
@freezed
abstract class DataState<T> with _$DataState<T> {
  const factory DataState.success(T data) = DataSuccess<T>;

  const factory DataState.dioError({
    required DioException error,
    String? userMessage,
    String? technicalMessage,
    required String module,
    required String file,
    required String line,
    required String stackTrace,
  }) = DataDioError<T>;

  const factory DataState.generalError({
    required String userMessage,
    required String technicalMessage,
    required String module,
    required String file,
    required String line,
    required String stackTrace,
  }) = DataGeneralError<T>;
}
```

### 3.2 DataStateConvenience + propagateError

**EXTENSIÓN sobre `DataState<T>`** que incluye mapeo de errores y `propagateError`. No es una clase separada.

```dart
/// Mapeo de errores DioException → mensajes user-friendly
/// 400 → "Los datos enviados no son válidos"
/// 401 → "Tus credenciales han expirado"
/// 403 → "No tienes permisos"
/// 404 → "No pudimos encontrar lo que buscas"
/// 500/502/503 → "El servidor está teniendo problemas"
/// Timeout → "La conexión tardó demasiado"

extension DataStateConvenience<T> on DataState<T> {
  // Verificadores de tipo
  bool get isSuccess      => maybeWhen(orElse: () => false);
  bool get isDioError     => maybeWhen(orElse: () => false);
  bool get isGeneralError  => maybeWhen(orElse: () => false);
  bool get isError         => isDioError || isGeneralError;

  // Accesores
  T? get dataOrNull;
  String? get userMessageOrNull;
  String? get technicalMessageOrNull;

  /// Map fromDioException → mensaje amigable
  static String _mapDioExceptionToMessage(DioException error) { ... }
}

/// propagateError<R>()
///
/// Convierte un DataState<A> en DataState<B> preservando el error original.
/// Sin generar logs duplicados, solo cambia el tipo genérico.
///
/// ⚠️ Lanza StateError si se llama sobre un DataState.success
extension DataStatePropagateError<T> on DataState<T> {
  DataState<R> propagateError<R>() {
    return maybeWhen(
      success: (_) => throw StateError('propagateError solo válido en estados de error'),
      dioError: (error, userMessage, technicalMessage, module, file, line, stackTrace) =>
        DataState<R>.dioError(
          error: error,
          userMessage: userMessage ?? _mapDioExceptionToMessage(error),
          technicalMessage: technicalMessage ?? error.message,
          module: module,
          file: file,
          line: line,
          stackTrace: stackTrace,
        ),
      generalError: (userMessage, technicalMessage, module, file, line, stackTrace) =>
        DataState<R>.generalError(
          userMessage: userMessage,
          technicalMessage: technicalMessage,
          module: module,
          file: file,
          line: line,
          stackTrace: stackTrace,
        ),
      orElse: () => throw StateError('Estado no reconocido'),
    );
  }
}
```

### 3.3 DataResult<T> y DataResultVoid

```dart
typedef DataResult<T> = Future<DataState<T>>;
typedef DataResultVoid = Future<DataState<VoidSuccess>>;

/// Clase especial para éxito sin datos
class VoidSuccess { const VoidSuccess(); }
```

---

## 4. Los 4 Tipos de UseCase

El proyecto define **4 abstract classes** para cubrir todos los escenarios de uso:

```
┌─────────────────────────────────────────────────────────────────┐
│                        UseCase                                  │
├───────────────────────┬─────────────────────────────────────────┤
│   CON PARÁMETROS      │         SIN PARÁMETROS                 │
│                       │                                         │
│  UseCaseWithParams<T> │    UseCaseWithoutParams<T>             │
│  UseCaseVoid<Params>  │    UseCaseVoidWithoutParams            │
└───────────────────────┴─────────────────────────────────────────┘
```

### 4.1 Implementación Base

```dart
// ============================================================================
// USE CASE CON PARÁMETROS
// ============================================================================

/// UseCase que recibe parámetros y retorna un valor
abstract class UseCaseWithParams<Type, Params> {
  const UseCaseWithParams();

  DataResult<Type> call(Params params);
}

/// UseCase que recibe parámetros pero retorna void
abstract class UseCaseVoid<Params> {
  const UseCaseVoid();

  DataResultVoid call(Params params);
}

// ============================================================================
// USE CASE SIN PARÁMETROS
// ============================================================================

/// UseCase sin parámetros que retorna un valor
abstract class UseCaseWithoutParams<Type> {
  const UseCaseWithoutParams();

  DataResult<Type> call();
}

/// UseCase sin parámetros que retorna void
abstract class UseCaseVoidWithoutParams {
  const UseCaseVoidWithoutParams();

  DataResultVoid call();
}
```

### 4.2 Cuándo Usar Cada Uno

| Tipo | Cuándo Usar | Ejemplo |
|------|-------------|---------|
| `UseCaseWithParams<T>` | Operaciones que requieren datos de entrada | `GetUserById(Params)`, `LoginUser(LoginParams)` |
| `UseCaseWithoutParams<T>` | Operaciones que no requieren parámetros | `GetCurrentUser()`, `GetDashboard()` |
| `UseCaseVoid<Params>` | Acciones que no retornan datos | `LogoutUser(userId)`, `DeleteItem(itemId)` |
| `UseCaseVoidWithoutParams` | Acciones sin entrada ni retorno | `ClearCache()`, `SyncData()` |

### 4.3 Ejemplos de Uso

#### UseCaseWithParams (más común)

```dart
// Domain: Definición
class GetUserByIdUseCase extends UseCaseWithParams<User, UserIdParams> {
  final UserRepository _repository;

  GetUserByIdUseCase(this._repository);

  @override
  DataResult<User> call(UserIdParams params) async {
    return _repository.getUserById(params.id);
  }
}

// Domain: Params
class UserIdParams {
  final String id;
  const UserIdParams({required this.id});
}

// Data: Llamada
final result = await getUserByIdUseCase(const UserIdParams(id: '123'));

if (result.isSuccess) {
  final user = result.dataOrNull;
}
```

#### UseCaseWithoutParams

```dart
// Domain: Definición
class GetCurrentUserUseCase extends UseCaseWithoutParams<User> {
  final UserRepository _repository;

  GetCurrentUserUseCase(this._repository);

  @override
  DataResult<User> call() async {
    return _repository.getCurrentUser();
  }
}

// Data: Llamada
final result = await getCurrentUserUseCase();

if (result.isError) {
  return result.propagateError<SomeOtherType>();
}
```

#### UseCaseVoid (acciones sin retorno)

```dart
// Domain: Definición
class LogoutUserUseCase extends UseCaseVoid<UserIdParams> {
  final AuthRepository _repository;

  LogoutUserUseCase(this._repository);

  @override
  DataResultVoid call(UserIdParams params) async {
    return _repository.logout(params.id);
  }
}

// Data: Llamada
final result = await logoutUserUseCase(const UserIdParams(id: '123'));

if (result.isSuccess) {
  getIt<ToastContract>().showSuccess(message: 'Sesión cerrada');
}
```

#### UseCaseVoidWithoutParams

```dart
// Domain: Definición
class ClearCacheUseCase extends UseCaseVoidWithoutParams {
  final CacheRepository _repository;

  ClearCacheUseCase(this._repository);

  @override
  DataResultVoid call() async {
    return _repository.clearCache();
  }
}

// Data: Llamada
await clearCacheUseCase();
```

### 4.4 Params con Freezed (Opcional pero Recomendado)

Para casos complejos, usar `@freezed` en los params:

```dart
@freezed
class LoginParams with _$LoginParams {
  const factory LoginParams({
    required String email,
    required String password,
    bool? rememberMe,
  }) = _LoginParams;
}

// Uso
class LoginUseCase extends UseCaseWithParams<User, LoginParams> {
  // ...
  
  DataResult<User> call(LoginParams params) async {
    // params.email, params.password, params.rememberMe
  }
}

await loginUseCase(const LoginParams(
  email: 'test@test.com',
  password: '123456',
  rememberMe: true,
));
```

---

## 5. ConnectivityEventBus

**Singleton que expone streams de eventos para usar fuera del árbol de widgets.**

```dart
/// Ejemplo de uso:
/// - Desde el cliente HTTP (interceptores)
/// - Desde servicios en segundo plano
/// - Para compartir estado entre widgets no relacionados

class ConnectivityEventBus {
  final _connectivityStream = BehaviorSubject<bool>.seeded(true);
  
  static final ConnectivityEventBus instance = ConnectivityEventBus._();
  
  Stream<bool> get connectivityStream => _connectivityStream.stream;
  bool get isConnected => _connectivityStream.value;
  
  void updateStatus(bool isConnected) {
    _connectivityStream.add(isConnected);
  }
}
```

---

## 6. Feature Pattern

### Estructura

```
features/{name}/
├── {name}.dart              # Barrel export
├── data/
│   ├── data.dart
│   ├── data_sources/
│   ├── models/
│   └── repositories/
├── domain/
│   ├── domain.dart
│   ├── entities/
│   ├── repositories/
│   └── use_cases/
│       ├── get_user_by_id_use_case.dart    # UseCaseWithParams<T>
│       ├── get_current_user_use_case.dart   # UseCaseWithoutParams<T>
│       ├── logout_user_use_case.dart       # UseCaseVoid<Params>
│       └── clear_cache_use_case.dart        # UseCaseVoidWithoutParams
└── presentation/
    ├── presentation.dart
    └── bloc/
        └── {name}_bloc/
```

### Domain Layer

- **Entity**: POJO puro con `factory Entity.mock()`
- **Repository Interface**: `abstract class` con métodos que retornan `DataResult<T>`
- **UseCase**: Puede ser cualquiera de los 4 tipos:
  - `UseCaseWithParams<T>` - Con parámetros y retorno
  - `UseCaseWithoutParams<T>` - Sin parámetros, con retorno
  - `UseCaseVoid<Params>` - Con parámetros, sin retorno
  - `UseCaseVoidWithoutParams` - Sin parámetros ni retorno

### Data Layer

> ⚠️ **IMPORTANTE**: Los DataSources DEBEN implementarse con **Retrofit**.
> - Usar `@RestApi()` y anotaciones `@POST`, `@GET`, `@PUT`, `@DELETE`, etc.
> - Incluir la línea `part '{feature_name}_data_source.g.dart';` al inicio del archivo (después de los imports)
> - Esto permite que build_runner genere el archivo `.g.dart` necesario para la compilación

**Ejemplo de DataSource con Retrofit:**
```dart
import 'package:dio/dio.dart';
import 'package:retrofit/retrofit.dart';
import 'package:stoki/features/auth/data/models/auth_model.dart';

part 'auth_data_source.g.dart';

@RestApi()
abstract class AuthDataSource {
  factory AuthDataSource(Dio dio) = _AuthDataSource;

  @POST('/auth/login')
  Future<AuthModel> login(
    @Field('email') String email,
    @Field('password') String password,
  );
}
```

- **Model**: `fromJson`, `toJson`, `toEntity()`
- **RepositoryImpl**: Implementa la interfaz, usa `DataStateConvenience`

### Presentation Layer

- **Event**: `@freezed` con factories por cada acción
- **State**: `@freezed` con estados `initial`, `loading`, `error`, `loaded`
- **StateX Extension**: Verificadores (`isLoading`, `isError`) y accesores cross-state
- **Bloc**: Handlers que emiten estados preservando datos

---

## 7. Inyección de Dependencias

```dart
void setupInjector() {
  // Singletons globales
  getIt.registerSingleton<SharedPreferenceHelper>(SharedPreferenceHelper.instance);
  getIt.registerSingleton<ToastContract>(ToastHelper.instance);
  getIt.registerSingleton<ConnectivityEventBus>(ConnectivityEventBus.instance);

  // Clientes HTTP
  getIt.registerSingleton<AppClient>(AppClient(...));

  // Features
  payments_dependencies();
}
```

Por feature:
```dart
void payments_dependencies() {
  // Data Sources
  getIt.registerFactory<PaymentsDataSource>(
    () => PaymentsDataSource(getIt<AppClient>().dio),
  );
  
  // Repositories
  getIt.registerFactory<PaymentsRepository>(
    () => PaymentsRepositoryImpl(getIt<PaymentsDataSource>()),
  );
  
  // UseCases
  getIt.registerFactory<GetPaymentLinkUseCase>(
    () => GetPaymentLinkUseCase(getIt<PaymentsRepository>()),
  );
  getIt.registerFactory<CancelPaymentUseCase>(
    () => CancelPaymentUseCase(getIt<PaymentsRepository>()),
  );
  
  // BLoCs
  getIt.registerLazySingleton<PaymentBloc>(
    () => PaymentBloc(
      getPaymentLinkUseCase: getIt<GetPaymentLinkUseCase>(),
      cancelPaymentUseCase: getIt<CancelPaymentUseCase>(),
    ),
  );
}
```

---

## 8. Main Entry Point

```dart
void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await initializeDateFormatting('es', null);
  setupInjector();
  await getIt<SharedPreferenceHelper>().init();
  SystemChrome.setPreferredOrientations([DeviceOrientation.portraitUp]);
  runApp(const App());
}
```

---

## 9. App Root

```dart
class App extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final clampedScale = MediaQuery.of(context).textScaler.scale(1.0).clamp(1.0, 1.3);

    return MediaQuery(
      data: MediaQuery.of(context).copyWith(textScaler: TextScaler.linear(clampedScale)),
      child: ToastificationWrapper(
        child: MaterialApp(
          theme: AppTheme.themeLight,
          initialRoute: Routes.initialRoute,
          routes: Routes.routes,
          onGenerateRoute: Routes.onGenerateRoute,
        ),
      ),
    );
  }
}
```

> **Nota:** No se usa ConnectivityBloc en App. En su lugar, usar `ConnectivityEventBus.instance.connectivityStream` con `StreamBuilder` o `context.watch<ConnectivityEventBus>`.

---

## 10. Checklist Nuevo Feature

```
□ 1. Crear carpeta lib/features/{feature_name}/
□ 2. Domain: 
   □ entity (con factory .mock())
   □ repository interface (métodos retornan DataResult<T>)
   □ use cases (seleccionar el tipo correcto de UseCase)
□ 3. Data: 
   □ data source (@RestApi con Retrofit) ← OBLIGATORIO
   □ incluir: part '{feature_name}_data_source.g.dart'; ← OBLIGATORIO
   □ models (fromJson/toJson/toEntity)
   □ repository impl (usa DataStateConvenience)
□ 4. Presentation: 
   □ bloc/event/state (@freezed)
   □ StateX extension (verificadores)
□ 5. dependencies/{name}_dependencies.dart
□ 6. Registrar en setupInjector()
□ 7. flutter pub run build_runner build --delete-conflicting-outputs
□ 8. Agregar rutas
```

---

## 11. Code Generation (OBLIGATORIO)

> ⚠️ **IMPORTANTE**: Los DataSources usan Retrofit que requiere generación de código. Este paso es **OBLIGATORIO** después de crear o modificar DataSources.

```bash
# Una vez
flutter pub run build_runner build --delete-conflicting-outputs

# Watch mode (recomendado durante desarrollo)
flutter pub run build_runner watch --delete-conflicting-outputs
```

### ¿Por qué es necesario?

1. Los DataSources usan `@RestApi` y `@POST`, `@GET`, etc. de Retrofit
2. Deben incluir la línea `part '{feature_name}_data_source.g.dart';`
3. Esto genera automáticamente el archivo `*.g.dart` con las implementaciones
4. Sin este paso, el código NO compilará

### Errores comunes si no se ejecuta

```
Error: Target of URI hasn't been generated: 'feature/data/datasources/feature_data_source.g.dart'
Error: The name '_FeatureDataSource' isn't a type and can't be used in a redirected constructor.
```

---

## 12. ToastContract

Interfaz abstracta en `core/contracts/toast_contract.dart` implementada por `ToastHelper`.

Uso:
```dart
getIt<ToastContract>().showSuccess(message: 'Operación exitosa');
getIt<ToastContract>().showError(message: state.userMessageOrNull ?? 'Error');
```

Requisito: `ToastificationWrapper` debe envolver el `MaterialApp`.

---

## 13. Reglas de Limpieza de Estructura

Antes de generar código, verificar:

1. **No crear carpetas innecesarias**: Si una carpeta solo tendrá un archivo, considerar ponerlo directamente en el nivel superior
2. **Eliminar carpetas vacías**: Después de generar, verificar que no haya carpetas sin archivos
3. **Barrel exports**: Cada carpeta debe tener un archivo `.dart` que re-exporte su contenido

Ejemplo de carpeta innecesaria:
```
❌ contracts/
    └── toast_contract/
        └── toast_contract.dart

✅ contracts/
    └── toast_contract.dart
```

---

## 14. Resumen de Tipos de UseCase

```
┌────────────────────────────────────────────────────────────────────────────┐
│                           GUÍA RÁPIDA DE USECASE                           │
├────────────────────────────────────────────────────────────────────────────┤
│                                                                            │
│  ¿Retorna valor?        │  ¿Requiere parámetros?                          │
│         │               │         │                                        │
│         ▼               │         ▼                                        │
│    ┌─────────┐         │    ┌─────────┐                                   │
│    │   SÍ    │         │    │   SÍ    │  → UseCaseWithParams<T>          │
│    └─────────┘         │    └─────────┘     ej: GetUserById(id)            │
│                        │                                               │
│         │              │         │                                        │
│         ▼              │         ▼                                        │
│    ┌─────────┐         │    ┌─────────┐                                   │
│    │   NO    │         │    │   SÍ    │  → UseCaseVoid<Params>           │
│    └─────────┘         │    └─────────┘     ej: DeleteUser(id)             │
│                        │                                               │
│         │              │         │                                        │
│         ▼              │         ▼                                        │
│    ┌─────────┐         │    ┌─────────┐                                   │
│    │   SÍ    │         │    │   NO    │  → UseCaseWithoutParams<T>        │
│    └─────────┘         │    └─────────┘     ej: GetCurrentUser()          │
│                        │                                               │
│         │              │         │                                        │
│         ▼              │         ▼                                        │
│    ┌─────────┐         │    ┌─────────┐                                   │
│    │   NO    │         │    │   NO    │  → UseCaseVoidWithoutParams      │
│    └─────────┘         │    └─────────┘     ej: ClearCache()              │
│                        │                                               │
└────────────────────────────────────────────────────────────────────────────┘
```