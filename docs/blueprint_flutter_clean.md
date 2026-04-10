# Blueprint: Arquitectura Flutter — AppCobranza (Exitus)

> Documento de especificación técnica para replicar el boilerplate de este proyecto desde cero.
> Generado el 5 de abril de 2026.

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
│   └── segi_app.dart                 # MaterialApp: theme, routes, onGenerateRoute
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
│   ├── constants/                    # ApiConstants (baseUrl, tokens, timeouts)
│   ├── contracts/
│   │   ├── contracts.dart
│   │   └── toast_contract/           # Interfaz abstracta de Toast
│   ├── events/                       # ConnectivityEventBus (Singleton, RxDart)
│   ├── foundation/
│   │   ├── foundation.dart           # Barrel export
│   │   ├── bloc_observer/
│   │   │   └── app_bloc_observer.dart
│   │   ├── data_state/
│   │   │   ├── data_state.dart       # Sealed class DataState<T> con Freezed
│   │   │   └── data_state.freezed.dart
│   │   └── use_case/
│   │       └── use_case.dart         # UseCaseWithParams + UseCaseWithoutParams
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
│   │   │   ├── client.dart    # Dio → Middleware API (Ejemplo de uso)
│   │   │   └── client_direct.dart # Dio → Backend directo (Ejemplo de uso)
│   │   ├── interceptors/
│   │   │   ├── interceptors.dart
│   │   │   ├── connectivity_interceptor.dart
│   │   │   ├── token_interceptor.dart
│   │   │   └── logger_interceptor.dart
│   │   └── response_model/
│   │       └── response_model.dart   # Wrapper genérico de respuesta API
│   └── utils/
│       ├── utils.dart
│       ├── formatters/
│       ├── types/
│       │   └── typedef.dart          # DataResult<T> = Future<DataState<T>>
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
    └── payments/                     # Feature de referencia (ver Sección 5)
        ├── payments.dart             # Barrel export del feature
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
        └── presentation/
            ├── presentation.dart
            └── bloc/

assets/
├── icons/
├── images/
└── lotties/
    └── loading_animated.json
```

---

## 3. Core Foundation

### 3.1 `DataState<T>` — Union Type con Freezed

Archivo: `lib/core/foundation/data_state/data_state.dart`

Reemplaza al patrón `Either<Failure, T>` de dartz con estados más expresivos.

#### Sealed class `DataState<T>`

```dart
@freezed
abstract class DataState<T> with _$DataState<T> {
  // ✅ Operación exitosa — contiene el dato resultante
  const factory DataState.success(T data) = DataSuccess<T>;

  // 🌐 Error de red/HTTP (Dio) — todos los campos required salvo los mensajes
  const factory DataState.dioError({
    required DioException error,
    String? userMessage,
    String? technicalMessage,
    required String module,
    required String file,
    required String line,
    required String stackTrace,
  }) = DataDioError<T>;

  // 🚨 Error general (parsing, casting, lógica de negocio)
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

**Uso en BLoC — pattern matching con `when()`:**

```dart
result.when(
  success: (data) => emit(PaymentState.loaded(payments: data)),
  dioError: (error, userMessage, technicalMessage, module, file, line, stackTrace) =>
      emit(PaymentState.error(message: userMessage!)),
  generalError: (userMessage, technicalMessage, module, file, line, stackTrace) =>
      emit(PaymentState.error(message: userMessage)),
);
```

**Uso alternativo — switch expressions (Dart 3.0+):**

```dart
switch (result) {
  case DataSuccess(:final data):
    emit(PaymentState.loaded(payments: data));
  case DataDioError(:final userMessage):
    emit(PaymentState.error(message: userMessage ?? 'Error de red'));
  case DataGeneralError(:final userMessage):
    emit(PaymentState.error(message: userMessage));
}
```

---

#### Extension `DataStateFactory` — Creación inteligente con logging automático

Esta extension sobre `DataState<T>` provee dos **métodos de fábrica estáticos** que automatizan la construcción de los estados de error. Es la forma recomendada de crear errores en los `RepositoryImpl`.

##### `DataStateFactory.fromDioException<T>()`

Para cuando se captura una `DioException`. Realiza automáticamente:

1. **Mapeo de código HTTP → mensaje user-friendly** según la tabla:

| Código HTTP / Tipo Dio | Mensaje al usuario |
|---|---|
| `400` | "Los datos enviados no son válidos" |
| `401` | "Tus credenciales han expirado. Vuelve a iniciar sesión" |
| `403` | "No tienes permisos para realizar esta acción" |
| `404` | "No pudimos encontrar lo que buscas" |
| `500 / 502 / 503` | "El servidor está teniendo problemas. Inténtalo en unos minutos" |
| `408` | "La conexión tardó demasiado. Revisa tu internet" |
| `connectionTimeout / sendTimeout / receiveTimeout` | "La conexión tardó demasiado. Revisa tu internet" |
| `connectionError` | "El servidor está teniendo problemas o no hay conexión a internet..." |
| `badCertificate` | "Problema de seguridad en la conexión" |
| `cancel` | "Operación cancelada" |
| Otros | "Hay un problema con el servicio. Inténtalo más tarde" |

2. **Sufijo de error** con `idError`: el mensaje final siempre termina en `(ERR_<idError>)` para facilitar el soporte técnico al usuario.

3. **Logging automático** en consola con `debugPrint`:
```
🔴 ERROR DE RED EN PaymentsRepositoryImpl 🌐
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📍 Tipo:       DioException (badResponse)
📍 Código:     404
📍 URL:        /ObtenerPagos
📍 Método:     GET
📍 Módulo:     PaymentsRepositoryImpl
📍 Archivo:    payments_repository_impl.dart:57
📍 Usuario:    No pudimos encontrar lo que buscas (ERR_PAY01)
📍 Técnico:    DioException: badResponse - Status: 404 - ...
📍 Response:   {"error": true, ...}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**Firma:**

```dart
static DataState<T> fromDioException<T>(
  DioException e, {
  required String module,
  required String file,
  required String line,
  required String stackTrace,
  required String idError,   // ← Código único de error, ej: 'PAY01'
})
```

**Uso en RepositoryImpl:**

```dart
} on DioException catch (e, stackTrace) {
  return DataStateFactory.fromDioException(
    e,
    module: 'PaymentsRepositoryImpl',
    file: 'payments_repository_impl.dart',
    line: '57',
    idError: 'PAY01',
    stackTrace: stackTrace.toString(),
  );
}
```

---

##### `DataStateFactory.fromException<T>()`

Para cuando se captura un `catch (e, stackTrace)` genérico (errores de parsing, casting, lógica). Realiza automáticamente:

1. **Mensaje user-friendly fijo**: `"Algo salió mal en la aplicación. Nuestro equipo fue notificado (ERR_<idError>)"`.
2. **Logging automático** en consola:
```
🔴 ERROR GENERAL EN PaymentsRepositoryImpl 🚨
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📍 Tipo:       GeneralError
📍 Módulo:     PaymentsRepositoryImpl
📍 Archivo:    payments_repository_impl.dart:63
📍 Usuario:    Algo salió mal en la aplicación... (ERR_PAY02)
📍 Técnico:    FormatException: ...
📍 Stacktrace: ...
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**Firma:**

```dart
static DataState<T> fromException<T>({
  required String message,     // e.toString()
  required String module,
  required String file,
  required String line,
  required String stackTrace,
  required String idError,
})
```

**Uso en RepositoryImpl:**

```dart
} catch (e, stackTrace) {
  return DataStateFactory.fromException(
    message: e.toString(),
    module: 'PaymentsRepositoryImpl',
    file: 'payments_repository_impl.dart',
    line: '63',
    idError: 'PAY02',
    stackTrace: stackTrace.toString(),
  );
}
```

---

#### Extension `DataStateConvenience` — Accesores rápidos

Segunda extension sobre `DataState<T>` para uso directo en la UI o en UseCases sin pattern matching completo:

```dart
extension DataStateConvenience<T> on DataState<T> {
  bool get isSuccess    => this is DataSuccess<T>;
  bool get isError      => this is DataDioError<T> || this is DataGeneralError<T>;
  bool get isDioError   => this is DataDioError<T>;
  bool get isGeneralError => this is DataGeneralError<T>;

  // Obtiene el dato si es success, null si no
  T? get dataOrNull => isSuccess ? (this as DataSuccess<T>).data : null;

  // Obtiene el mensaje de usuario si es error, null si es success
  String? get userMessageOrNull => when(
    success: (_) => null,
    dioError: (_, userMsg, ...) => userMsg,
    generalError: (userMsg, ...) => userMsg,
  );

  // Re-mapea un error de DataState<A> → DataState<B> sin duplicar logs
  // Solo válido en estados de error. Lanza StateError si se llama en success.
  DataState<R> propagateError<R>();
}
```

**Uso de `propagateError` en UseCases compuestos:**

```dart
// Cuando un UseCase orquesta múltiples repos y necesita pasar el error "hacia arriba"
final asignationResult = await _asignationRepository.getDetail(id);
if (asignationResult.isError) {
  return asignationResult.propagateError<OtherEntity>();
}
final detail = asignationResult.dataOrNull!;
```

### 3.2 `DataResult<T>` — Type Alias

Archivo: `lib/core/utils/types/typedef.dart`

```dart
typedef DataResult<T> = Future<DataState<T>>;
typedef DataResultVoid = DataResult<VoidSuccess>;

class VoidSuccess { const VoidSuccess(); }
```

### 3.3 `UseCaseWithParams` / `UseCaseWithoutParams`

Archivo: `lib/core/foundation/use_case/use_case.dart`

```dart
abstract class UseCaseWithParams<Type, Params> {
  const UseCaseWithParams();
  DataResult<Type> call(Params params);
}

abstract class UseCaseWithoutParams<Type> {
  const UseCaseWithoutParams();
  DataResult<Type> call();
}
```

### 3.4 `ResponseModel` — Wrapper genérico de API

Archivo: `lib/core/network/response_model/response_model.dart`

Normaliza respuestas de múltiples formatos de API:

```dart
class ResponseModel {
  final bool error;
  final String message;
  final dynamic data;    // data_info | catalogo | data | result
  final String code;
  final String? description;

  factory ResponseModel.fromJson(Map<String, dynamic> json) {
    // Soporta múltiples formatos: error/status/description_status
    bool isError = json['error'] ?? json['description_status'] != 'SUCCESS';
    if (json['status'] != null) isError = json['status'] != 1;
    return ResponseModel(
      error: isError,
      message: json['message'] ?? json['description_status'] ?? json['mensaje'] ?? '',
      data: json['data_info'] ?? json['catalogo'] ?? json['data'] ?? json['result'],
      ...
    );
  }
}
```

---

## 4. Capa de Red (Network Layer)

### 4.1 Clientes HTTP (Dio configurado)

Archivo: `lib/core/network/clients/client.dart`

El proyecto puede tener **uno o más clientes Dio**, cada uno apuntando a un origen distinto (middleware, backend directo, API externa, etc.). Todos comparten la misma estructura base.

```dart
class AppClient {
  late final Dio _dio;

  AppClient(ConnectivityEventBus bus, ConnectivityHelper helper) {
    _dio = Dio();
    _dio.options.baseUrl = ApiConstants.baseUrl;   // URL principal del proyecto
    _dio.options.sendTimeout = const Duration(seconds: ApiConstants.connectionTimeoutSeconds);
    _dio.options.receiveTimeout = const Duration(seconds: ApiConstants.receiveTimeoutSeconds);
    _dio.interceptors.addAll([
      ConnectivityInterceptor(helper, bus),
      TokenInterceptor(),
      LoggerInterceptor(),
    ]);
  }

  Dio get dio => _dio;
}
```

Si el proyecto requiere apuntar a un segundo origen (ej: backend directo, API de terceros) se crea `AppClientDirect` con la misma estructura pero con su propia `baseUrl` en `ApiConstants`. Ambos clientes se registran como `Singleton` en GetIt y se inyectan donde se necesiten.

### 4.2 Interceptores

| Interceptor | Responsabilidad |
|---|---|
| `ConnectivityInterceptor` | Verifica conexión antes de cada request; emite evento al EventBus si no hay red |
| `TokenInterceptor` | Inyecta `access-token` en el body de cada request |
| `LoggerInterceptor` | Log de requests/responses en desarrollo |

### 4.3 `ConnectivityEventBus` (Singleton)

Archivo: `lib/core/events/connectivity_event_bus.dart`

Implementado con `StreamController.broadcast()` de Dart (no requiere RxDart). Permite que cualquier parte de la app escuche eventos de conectividad sin acoplamiento directo.

```dart
enum ConnectivityEventType { noConnection }

class ConnectivityEventBus {
  static ConnectivityEventBus get instance => _instance ??= ConnectivityEventBus._internal();
  static ConnectivityEventBus? _instance;
  ConnectivityEventBus._internal();

  final _controller = StreamController<ConnectivityEventType>.broadcast();

  Stream<ConnectivityEventType> get stream => _controller.stream;

  void emit(ConnectivityEventType event) {
    if (!_controller.isClosed) _controller.add(event);
  }

  void dispose() => _controller.close();
}
```

El `ConnectivityInterceptor` llama a `emit(ConnectivityEventType.noConnection)` y el `ConnectivityBloc` (global) escucha el stream para reaccionar en la UI.

---

## 5. App Entry Point (`main.dart` + `segi_app.dart`)

### 5.1 `main.dart` — Bootstrapping

Archivo: `lib/main.dart`

Orden de inicialización obligatorio:

```dart
void main() async {
  WidgetsFlutterBinding.ensureInitialized();

  // Inicializar locale para formateo de fechas
  await initializeDateFormatting('es', null); // ajustar al locale del proyecto

  // Activar el observador global de BLoCs (solo en debug)
  // Bloc.observer = const AppBlocObserver();

  // Inicializar inyección de dependencias
  setupInjector();

  // SharedPreferences requiere init() async antes de usarse
  await getIt<SharedPreferenceHelper>().init();

  // Forzar orientación vertical (ajustar según el proyecto)
  SystemChrome.setPreferredOrientations([
    DeviceOrientation.portraitUp,
    DeviceOrientation.portraitDown,
  ]);

  runApp(const App());
}
```

> **Importante:** `SharedPreferenceHelper.init()` debe llamarse antes de `runApp()` porque el helper usa `late SharedPreferences _preferences` que se popula en ese método. Cualquier acceso previo causará un `LateInitializationError`.

### 5.2 `segi_app.dart` — App root

Archivo: `lib/app/app.dart`

Patrones clave que deben estar presentes en el widget raíz:

```dart
class App extends StatelessWidget {
  const App({super.key});

  @override
  Widget build(BuildContext context) {
    return Builder(builder: (context) {
      // Clamp del texto para evitar errores con date pickers
      // cuando el sistema tiene textScaleFactor > 1.3
      final rawScale = MediaQuery.of(context).textScaler.scale(1.0);
      final clampedScale = rawScale.clamp(1.0, 1.3);

      return MediaQuery(
        data: MediaQuery.of(context).copyWith(
          textScaler: TextScaler.linear(clampedScale),
        ),
        // ToastificationWrapper es OBLIGATORIO para que ToastHelper funcione.
        // Debe envolver todo el árbol, encima del MaterialApp.
        child: ToastificationWrapper(
          child: MultiBlocProvider(
            providers: [
              // BLoCs globales (singleton en GetIt, provisto aquí para toda la app)
              BlocProvider<ConnectivityBloc>(create: (_) => getIt<ConnectivityBloc>()),
              BlocProvider<OverlayBloc>(create: (_) => getIt<OverlayBloc>()),
              // ... otros BLoCs globales del proyecto
            ],
            child: MaterialApp(
              title: AppConstants.appName,
              debugShowCheckedModeBanner: false,
              theme: AppTheme.themeLight,
              initialRoute: Routes.initialRoute,
              routes: Routes.routes,
              onGenerateRoute: Routes.onGenerateRoute,
            ),
          ),
        ),
      );
    });
  }
}
```

> **`ToastificationWrapper`:** Sin este widget en el árbol, cualquier llamada a `toastification.show()` lanza una excepción en runtime. Debe estar por encima del `MaterialApp`.

> **`MultiBlocProvider` en raíz:** Los BLoCs globales (Connectivity, Overlay, Auth, etc.) se proveen aquí. Los BLoCs de feature se proveen localmente en sus pantallas.

### 5.3 `AppBlocObserver` — Observador global de BLoCs

Archivo: `lib/core/foundation/bloc_observer/app_bloc_observer.dart`

Observador de debug que loguea el ciclo de vida de los BLoCs. Se activa descomentando `Bloc.observer = const AppBlocObserver()` en `main.dart`.

**Patrón recomendado:** restringir el logging a BLoCs específicos para no saturar la consola:

```dart
class AppBlocObserver extends BlocObserver {
  const AppBlocObserver();

  @override
  void onCreate(BlocBase<dynamic> bloc) {
    super.onCreate(bloc);
    if (bloc is! TargetBloc) return; // filtrar por BLoC específico
    if (kDebugMode) debugPrint('🟢 onCreate -- ${bloc.runtimeType}');
  }

  @override
  void onEvent(Bloc<dynamic, dynamic> bloc, Object? event) {
    super.onEvent(bloc, event);
    if (bloc is! TargetBloc) return;
    if (kDebugMode) debugPrint('📩 onEvent -- ${bloc.runtimeType}\n   event: $event');
  }

  @override
  void onChange(BlocBase<dynamic> bloc, Change<dynamic> change) {
    super.onChange(bloc, change);
    if (bloc is! TargetBloc) return;
    if (kDebugMode) debugPrint(
      '🔄 onChange -- ${bloc.runtimeType}\n'
      '   current: ${change.currentState.runtimeType}\n'
      '   next:    ${change.nextState.runtimeType}',
    );
  }
}
```

---

## 6. Feature de Referencia: `payments`

Esta sección documenta la arquitectura completa de un feature. **Todos los features siguen este mismo patrón.**

### 5.1 Estructura de carpetas del feature

```
features/payments/
├── payments.dart          # Barrel: export data.dart, domain.dart, presentation.dart
├── data/
│   ├── data.dart
│   ├── data_sources/
│   │   ├── payments_data_source.dart     # @RestApi Retrofit
│   │   └── payments_data_source.g.dart   # Auto-generado
│   ├── models/
│   │   ├── payment_model.dart
│   │   ├── payment_detail_model.dart
│   │   └── payment_generated_model.dart
│   └── repositories/
│       └── payments_repository_impl.dart
├── domain/
│   ├── domain.dart
│   ├── entities/
│   │   ├── payment_entity.dart
│   │   ├── payment_detail_entity.dart
│   │   ├── payment_generated_entity.dart
│   │   └── payment_status.dart           # Enum con fromPaso() / toPaso()
│   ├── repositories/
│   │   └── payments_repository.dart      # Interfaz abstracta
│   └── use_cases/
│       ├── use_cases.dart
│       ├── generate_payment_link_use_case.dart
│       ├── get_payment_details_use_case.dart
│       └── get_payments_from_reference_use_case.dart
└── presentation/
    ├── presentation.dart
    └── bloc/
        └── payment_bloc/
            ├── payment_bloc.dart
            ├── payment_event.dart
            ├── payment_state.dart
            └── payment_bloc.freezed.dart
```

### 5.2 Capa Domain

**Entity** (POJO puro, sin dependencias de Flutter/Dart externas):
```dart
class PaymentEntity {
  final int idLinkPagoConekta;
  final String referencia;
  // ... todos los campos
  const PaymentEntity({required ...});
  factory PaymentEntity.mock() { ... } // Para pruebas y Skeletonizer
}
```

**Repository Interface** (contrato abstracto):
```dart
abstract class PaymentsRepository {
  DataResult<PaymentGeneratedEntity> generatePaymentLink({ ... });
  DataResult<List<PaymentEntity>> getPaymentsFromReference({ required String referencia });
  DataResult<PaymentDetailEntity> getPaymentDetails({ required String idLinkConekta });
}
```

**UseCase** (orquesta el repositorio):
```dart
class GeneratePaymentLinkUseCase
    extends UseCaseWithParams<PaymentGeneratedEntity, GeneratePaymentLinkParams> {
  final PaymentsRepository _repository;
  const GeneratePaymentLinkUseCase(this._repository);

  @override
  DataResult<PaymentGeneratedEntity> call(GeneratePaymentLinkParams params) async {
    return await _repository.generatePaymentLink( ... );
  }
}

// Params siempre van en una clase dedicada
class GeneratePaymentLinkParams {
  final String nombreCliente;
  final String referencia;
  // ...
  const GeneratePaymentLinkParams({ required ... });
}
```

### 5.3 Capa Data

**DataSource** (Retrofit `@RestApi`):

> **Nota sobre `baseUrl`:** El atributo `baseUrl` en `@RestApi` **no siempre es necesario**. La URL base normalmente la define el cliente Dio (`AppClient`) que se inyecta al construir el DataSource. Solo se especifica `baseUrl` directamente en el `@RestApi` cuando el DataSource apunta a una API externa cuyas peticiones no deben pasar por ninguno de los clientes del proyecto (ej: una API de terceros con su propia URL y autenticación). En ese caso se crea además un `AppClientDirect` dedicado o se construye el `Dio` manualmente.

```dart
// ✅ Caso normal: baseUrl viene del AppClient, NO se declara en @RestApi
@RestApi()
abstract class FeatureDataSource {
  factory FeatureDataSource(Dio dio, {String? baseUrl}) = _FeatureDataSource;

  @GET('/endpoint')
  Future<ResponseModel> getItems();
}

// ⚠️ Caso especial: API externa ajena a nuestros clientes
@RestApi(baseUrl: 'https://api.tercero.com/v1')
abstract class ExternalDataSource {
  factory ExternalDataSource(Dio dio, {String? baseUrl}) = _ExternalDataSource;

  @POST('/recurso')
  Future<ResponseModel> createItem({
    @Field('campo') required String campo,
    @Header('Authorization') required String auth,
  });

  @GET('/recursos')
  Future<ResponseModel> getItems({ @Query('filtro') required String filtro });
}
```

**Model** (extiende/mapea la Entity, contiene `fromJson` y `toJson`):
```dart
class PaymentModel {
  // Mismas propiedades que PaymentEntity
  factory PaymentModel.fromJson(Map<String, dynamic> json) { ... }
  Map<String, dynamic> toJson() { ... }
  PaymentEntity toEntity() => PaymentEntity( ... ); // Conversión a domain
}
```

**RepositoryImpl** (implementa la interfaz del dominio):
```dart
class PaymentsRepositoryImpl implements PaymentsRepository {
  late final PaymentsDataSource _dataSource;
  late final SharedPreferenceHelper _sharedPreferenceHelper;

  @override
  DataResult<List<PaymentEntity>> getPaymentsFromReference({ ... }) async {
    try {
      final response = await _dataSource.getPaymentsFromReference( ... );
      if (!response.error && response.data != null) {
        final list = (response.data as List).map((e) => PaymentModel.fromJson(e).toEntity()).toList();
        return DataState.success(list);
      } else {
        return DataState.generalError( userMessage: '...', ... );
      }
    } on DioException catch (e, stack) {
      return DataStateFactory.fromDioException(e, module: '...', file: '...', line: '...', stackTrace: stack.toString());
    } catch (e, stack) {
      return DataStateFactory.fromException(module: '...', ...);
    }
  }
}
```

### 5.4 Capa Presentation — BLoC con Freezed

**Event** (`part of 'payment_bloc.dart'`):
```dart
@freezed
class PaymentEvent with _$PaymentEvent {
  const factory PaymentEvent.generatePaymentLink({ required String referencia, ... }) = _GeneratePaymentLink;
  const factory PaymentEvent.getPaymentsFromReference({ required String referencia }) = _GetPaymentsFromReference;
}
```

**State** (`part of 'feature_bloc.dart'`):
```dart
@freezed
class FeatureState with _$FeatureState {
  const factory FeatureState.initial() = _Initial;
  const factory FeatureState.loading({
    required bool isFetchingList,        // Flag granular por operación
    required bool isExecutingAction,
    List<ItemEntity>? items,             // Datos preservados durante loading
    ItemDetailEntity? detail,
  }) = _Loading;
  const factory FeatureState.error({
    required String message,
    List<ItemEntity>? items,             // Datos preservados en error
    ItemDetailEntity? detail,
  }) = _Error;
  const factory FeatureState.loaded({
    List<ItemEntity>? items,
    ItemDetailEntity? detail,
  }) = _Loaded;
}
```

**Extension `FeatureStateX`** — se define en el mismo archivo `feature_state.dart` y cumple dos roles:

**1. Verificadores de tipo de estado** (usados en la capa de presentación para mostrar/ocultar widgets):
```dart
extension FeatureStateX on FeatureState {
  // ─── Verificadores de estado ─────────────────────────────────────────
  bool get isLoading => maybeMap(
    loading: (_) => true,
    orElse: () => false,
  );

  bool get isFetchingList => maybeMap(
    loading: (s) => s.isFetchingList,
    orElse: () => false,
  );

  bool get isExecutingAction => maybeMap(
    loading: (s) => s.isExecutingAction,
    orElse: () => false,
  );

  bool get isError => maybeMap(
    error: (_) => true,
    orElse: () => false,
  );

  String get errorMessage => mapOrNull(error: (s) => s.message) ?? '';

  // ─── Accesores de datos cross-state ──────────────────────────────────
  // Retornan el dato sin importar en qué estado se encuentre el BLoC,
  // permitiendo que la UI siempre tenga acceso al último valor conocido.
  List<ItemEntity>? get items => mapOrNull(
    loaded: (s) => s.items,
    loading: (s) => s.items,   // ← preservado durante loading
    error: (s) => s.items,     // ← preservado en error
  );

  ItemDetailEntity? get detail => mapOrNull(
    loaded: (s) => s.detail,
    loading: (s) => s.detail,
    error: (s) => s.detail,
  );
}
```

**2. Helpers internos del BLoC** — con estos accesores el BLoC puede leer el estado actual de forma limpia al emitir nuevos estados, sin necesidad de un `switch/when` completo:
```dart
// Dentro de un handler del BLoC, al emitir loading se preservan los datos actuales:
emit(FeatureState.loading(
  isFetchingList: true,
  isExecutingAction: false,
  items: state.items,    // ← usa el accessor cross-state
  detail: state.detail,
));

// Al emitir error también se preservan los datos:
emit(FeatureState.error(
  message: userMessage,
  items: state.items,
  detail: state.detail,
));
```

**Uso en la capa de presentación (Widget/BlocBuilder):**
```dart
BlocBuilder<FeatureBloc, FeatureState>(
  builder: (context, state) {
    if (state.isFetchingList) return const SkeletonListView();
    if (state.isError) return ErrorCard(message: state.errorMessage);

    final items = state.items;   // ← siempre disponible
    if (items == null || items.isEmpty) return const EmptyState();

    return ItemListView(
      items: items,
      isActionLoading: state.isExecutingAction, // spinner granular
    );
  },
)
```

**BLoC**:
```dart
class PaymentBloc extends Bloc<PaymentEvent, PaymentState> {
  final GeneratePaymentLinkUseCase _generatePaymentLinkUseCase;
  final GetPaymentsFromReference _getPaymentsFromReferenceUseCase;

  PaymentBloc({ required ... }) : super(const PaymentState.initial()) {
    on<_GeneratePaymentLink>(_onGeneratePaymentLink);
    on<_GetPaymentsFromReference>(_onGetPaymentsFromReference);
  }

  Future<void> _onGeneratePaymentLink(_GeneratePaymentLink event, Emitter<PaymentState> emit) async {
    // 1. Emitir loading preservando datos anteriores
    emit(PaymentState.loading(isGeneratingLink: true, payments: state.payments, ...));

    // 2. Ejecutar use case
    final result = await _generatePaymentLinkUseCase(GeneratePaymentLinkParams( ... ));

    // 3. Pattern match sobre DataState
    result.when(
      success: (data) => emit(PaymentState.loaded(paymentGenerated: data, payments: state.payments)),
      dioError: (error, userMessage, ...) => emit(PaymentState.error(message: userMessage!, ...)),
      generalError: (userMessage, ...) => emit(PaymentState.error(message: userMessage, ...)),
    );
  }
}
```

---

## 7. Inyección de Dependencias (GetIt)

### 6.1 `injector.dart` — Punto de entrada

```dart
final GetIt getIt = GetIt.instance;

void setupInjector() {
  // 1. Singletons globales (Helpers, EventBus)
  getIt.registerSingleton<ConnectivityEventBus>(ConnectivityEventBus.instance);
  getIt.registerSingleton<ConnectivityHelper>(ConnectivityHelper.instance);
  getIt.registerSingleton<SharedPreferenceHelper>(SharedPreferenceHelper.instance);
  getIt.registerSingleton<SecureStorageHelper>(SecureStorageHelper.instance);
  getIt.registerSingleton<ToastContract>(ToastHelper.instance);

  // 2. Clientes HTTP (nombrarlos según el proyecto, ej: AppClient / AppClientDirect)
  getIt.registerSingleton<AppClient>(AppClient(getIt<ConnectivityEventBus>(), getIt<ConnectivityHelper>()));
  getIt.registerSingleton<AppClientDirect>(AppClientDirect(getIt<ConnectivityEventBus>(), getIt<ConnectivityHelper>()));

  // 3. BLoCs globales
  getIt.registerSingleton<OverlayBloc>(OverlayBloc());
  getIt.registerSingleton<ConnectivityBloc>(ConnectivityBloc());

  // 4. Dependencias por feature (modularizadas)
  auth_dependencies();
  payments_dependencies();
  // ...
}
```

### 6.2 Patrón por feature (`payments_dependencies.dart`)

```dart
void payments_dependencies() {
  // DataSource: registerFactory (nueva instancia cada vez)
  // Inyectar el cliente que corresponda según el origen de la API
  getIt.registerFactory<PaymentsDataSource>(
    () => PaymentsDataSource(getIt<AppClient>().dio),
  );

  // Repository: registerFactory
  getIt.registerFactory<PaymentsRepository>(
    () => PaymentsRepositoryImpl(
      dataSource: getIt<PaymentsDataSource>(),
      sharedPreferenceHelper: getIt<SharedPreferenceHelper>(),
    ),
  );

  // Use Cases: registerFactory
  getIt.registerFactory<GeneratePaymentLinkUseCase>(
    () => GeneratePaymentLinkUseCase(getIt<PaymentsRepository>()),
  );

  // BLoC: registerLazySingleton (instancia única creada al primer uso)
  getIt.registerLazySingleton<PaymentBloc>(
    () => PaymentBloc(
      generatePaymentLinkUseCase: getIt<GeneratePaymentLinkUseCase>(),
      getPaymentsFromReferenceUseCase: getIt<GetPaymentsFromReference>(),
      getPaymentDetailsUseCase: getIt<GetPaymentDetails>(),
    ),
  );
}
```

**Regla de registro:**
- `registerSingleton` → Helpers globales, clientes HTTP, EventBus
- `registerLazySingleton` → BLoCs de feature
- `registerFactory` → DataSources, Repositories, UseCases (sin estado)

---

## 8. Rutas

Archivo: `lib/config/routes/routes.dart`

```dart
class Routes {
  static String initialRoute = "/";
  static const String paymentLink = "/payment-link";
  static const String paymentHistory = "/payment-history";
  // ...

  // Rutas simples (sin args)
  static Map<String, Widget Function(BuildContext)> routes = {
    "/": (_) => const LoginScreen(),
    "/home": (_) => HomeScreen(),
  };

  // Rutas con argumentos tipados
  static Route<dynamic>? onGenerateRoute(RouteSettings settings) {
    switch (settings.name) {
      case paymentLink:
        final extra = settings.arguments as Map<String, dynamic>;
        return MaterialPageRoute(
          builder: (_) => PaymentLinkScreen(
            reference: extra['referenceId'] as String,
            nameClient: extra['nameClient'] as String,
          ),
        );
      default:
        return null;
    }
  }
}
```

Uso en `MaterialApp`:
```dart
MaterialApp(
  initialRoute: Routes.initialRoute,
  routes: Routes.routes,
  onGenerateRoute: Routes.onGenerateRoute,
  theme: AppTheme.theme,
)
```

Navegación con args:
```dart
Navigator.pushNamed(context, Routes.paymentLink, arguments: {
  'referenceId': '123',
  'nameClient': 'Juan Pérez',
});
```

---

## 9. Barrel Exports — Convención

Cada carpeta tiene un archivo `nombre_carpeta.dart` que re-exporta todo su contenido:

```dart
// lib/features/payments/payments.dart
export 'data/data.dart';
export 'domain/domain.dart';
export 'presentation/presentation.dart';

// lib/features/payments/domain/domain.dart
export 'entities/entities.dart';
export 'repositories/repositories.dart';
export 'use_cases/use_cases.dart';
```

Esto permite importar todo un feature con una sola línea:
```dart
import 'package:app_gestion/features/payments/payments.dart';
```

---

## 10. Reglas de Code Generation

Ejecutar siempre después de crear/modificar archivos con `@freezed`, `@RestApi`, `@JsonSerializable`:

```bash
flutter pub run build_runner build --delete-conflicting-outputs
```

Para desarrollo continuo:
```bash
flutter pub run build_runner watch --delete-conflicting-outputs
```

Archivos generados automáticamente (no editar manualmente):
- `*.freezed.dart` — Freezed union types
- `*.g.dart` — Retrofit DataSources y JSON serialization

---

## 11. Constantes del Proyecto

### 11.1 `ApiConstants` — Red y entornos

Archivo: `lib/core/constants/api_constants.dart`

Template con soporte multi-entorno. Ajustar URLs y tokens según el proyecto:

```dart
class ApiConstants {
  // Modo debug (usa kDebugMode de Flutter)
  static const bool debugMode = kDebugMode;

  // Timeouts de red (segundos)
  static const int connectionTimeoutSeconds = 30;
  static const int receiveTimeoutSeconds    = 30;

  // URLs por entorno
  static const String _urlProd   = 'https://api.miproyecto.com/v1';
  static const String _urlQA     = 'https://api-qa.miproyecto.com/v1';
  static const String _urlDirect = 'https://backend.miproyecto.com/api/v1'; // acceso directo

  // URL activa (cambiar según el entorno de compilación)
  static const String baseUrl       = _urlProd;
  static const String baseUrlDirect = _urlDirect;

  // Tokens por entorno
  static const String _tokenProd = 'TOKEN_PRODUCCION';
  static const String _tokenTest = 'TOKEN_QA';
  static const String apiToken   = debugMode ? _tokenTest : _tokenProd;
}
```

### 11.2 `AppConstants` — Constantes de UI y lógica

Archivo: `lib/core/constants/app_constants.dart`

Template de constantes no relacionadas a red:

```dart
class AppConstants {
  // Información de la app
  static const String appName    = 'NombreApp';
  static const String appVersion = '1.0.0';

  // Claves de SharedPreferences (usadas en SharedPreferenceHelper)
  static const String userIdKey    = 'user-id';
  static const String userNameKey  = 'user-name';
  static const String userTokenKey = 'user-token';
  // ... agregar las claves necesarias por proyecto

  // Paginación
  static const int defaultPageSize = 20;

  // Animaciones
  static const int defaultAnimationDurationMs = 300;
  static const int loadingAnimationDurationMs = 500;

  // Validación
  static const int minPasswordLength  = 6;
  static const int maxUsernameLength  = 50;

  // UI
  static const double defaultBorderRadius = 8.0;
}
```

### 11.3 `SharedPreferenceHelper` — Patrón de Mixins

Archivo: `lib/core/helpers/shared_preference_helper.dart`

La clase principal extiende una base privada y agrega funcionalidad mediante **Mixins** por dominio. Esto evita que la clase crezca indefinidamente:

```dart
class SharedPreferenceHelper extends _SharedPreferenceCacheBase
    with UserSessionMixin, CatalogsMixin, FeatureAMixin {
  static final SharedPreferenceHelper _instance = SharedPreferenceHelper._internal();
  static SharedPreferenceHelper get instance => _instance;
  factory SharedPreferenceHelper() => _instance;
  SharedPreferenceHelper._internal();

  late SharedPreferences _preferences;

  // Debe llamarse en main() antes de runApp()
  Future<bool> init() async {
    _preferences = await SharedPreferences.getInstance();
    return true;
  }
}

// Cada Mixin encapsula sus propias claves y métodos
mixin UserSessionMixin on _SharedPreferenceCacheBase {
  Future<void> saveUserId(String id) => setString(AppConstants.userIdKey, id);
  String? getUserId() => getString(AppConstants.userIdKey);
}

mixin CatalogsMixin on _SharedPreferenceCacheBase {
  // métodos de caché de catálogos
}
```

---

## 12. Checklist para Crear un Nuevo Feature

```
□ 1. Crear carpeta lib/features/{feature_name}/
□ 2. Crear barrel: {feature_name}.dart
□ 3. Domain:
    □ entity: {name}_entity.dart (POJO + mock())
    □ repository interface: {name}_repository.dart (abstract class)
    □ use case(s): {action}_{name}_use_case.dart + Params class
    □ domain.dart (barrel)
□ 4. Data:
    □ data source: {name}_data_source.dart (@RestApi Retrofit)
    □ model: {name}_model.dart (fromJson + toJson + toEntity())
    □ repository impl: {name}_repository_impl.dart (implements + try/catch DataState)
    □ data.dart (barrel)
□ 5. Presentation:
    □ bloc/{name}_bloc/
        □ {name}_event.dart (@freezed, part of)
        □ {name}_state.dart (@freezed, part of + StateX extension)
        □ {name}_bloc.dart (imports parts, handlers)
    □ presentation.dart (barrel)
□ 6. Injector:
    □ Crear lib/injector/dependencies/{name}_dependencies.dart
    □ Registrar en setupInjector() en injector.dart
    □ Agregar export en dependencies/dependencies.dart
□ 7. Ejecutar build_runner
□ 8. Agregar rutas en config/routes/routes.dart si aplica
```

---

## 13. Patrones Clave

### Preservar datos durante loading
Los estados `loading` y `error` del BLoC llevan los mismos campos opcionales que `loaded`, permitiendo que la UI muestre datos previos con skeleton/shimmer mientras se actualizan.

### Singletons con `.instance`
Los Helpers (ConnectivityHelper, SharedPreferenceHelper, etc.) implementan el patrón Singleton clásico con `static final instance = HelperClass._()` para garantizar una sola instancia sin GetIt.

### DataStateFactory
Factory estática que mapea automáticamente `DioException` a mensajes user-friendly según el código HTTP (401, 403, 404, 500, timeout, etc.) y loguea con Logger.

### ToastContract

Interfaz en `core/contracts/toast_contract/toast_contract.dart` implementada por `ToastHelper` en `core/helpers/`. Se registra como `ToastContract` en GetIt, permitiendo mockear en tests.

**Implementación:** `ToastHelper` usa internamente el paquete **`toastification`** para mostrar las notificaciones en pantalla. Define además dos enums propios que la interfaz expone: `ToastPosition` (top/center/bottom) y `ToastDuration` (short/medium/long/custom).

**Requisito crítico:** Para que `toastification` funcione en runtime, el widget **`ToastificationWrapper`** debe envolver el árbol de widgets en `segi_app.dart`, por encima del `MaterialApp`. Sin él, cualquier llamada a `showSuccess()` / `showError()` lanzará una excepción:

```dart
// ✅ Correcto
ToastificationWrapper(
  child: MaterialApp( ... ),
)

// ❌ Incorrecto — toastification no encontrará el contexto
MaterialApp(
  home: ToastificationWrapper( ... ), // demasiado abajo en el árbol
)
```

Uso desde features:
```dart
getIt<ToastContract>().showSuccess(message: 'Operación exitosa');
getIt<ToastContract>().showError(message: state.errorMessage);
```
