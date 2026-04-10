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
│   │   │   ├── exitus_client.dart    # Dio → Middleware API
│   │   │   └── exitus_client_direct.dart # Dio → Backend directo
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

### 4.1 ExitusClient (Dio configurado)

Archivo: `lib/core/network/clients/exitus_client.dart`

```dart
class ExitusClient {
  late final Dio _dio;

  ExitusClient(ConnectivityEventBus bus, ConnectivityHelper helper) {
    _dio = Dio();
    _dio.options.baseUrl = ApiConstants.baseUrl;
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

Existe `ExitusClientDirect` con la misma estructura pero apuntando al backend directo.

### 4.2 Interceptores

| Interceptor | Responsabilidad |
|---|---|
| `ConnectivityInterceptor` | Verifica conexión antes de cada request; emite evento al EventBus si no hay red |
| `TokenInterceptor` | Inyecta `access-token` en el body de cada request |
| `LoggerInterceptor` | Log de requests/responses en desarrollo |

### 4.3 `ConnectivityEventBus` (Singleton)

Basado en RxDart. Emite eventos `ConnectivityEventType.noConnection` para que los BLoCs globales reaccionen.

---

## 5. Feature de Referencia: `payments`

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
```dart
@RestApi(baseUrl: 'https://api.example.com/api/LinkDePago')
abstract class PaymentsDataSource {
  factory PaymentsDataSource(Dio dio, {String? baseUrl}) = _PaymentsDataSource;

  @POST('/GenerarLink')
  Future<ResponseModel> generatePaymentLink({
    @Field('nombre_cliente') required String nombreCliente,
    // ... otros @Field / @Query / @Header
    @Header('Authorization') required String basicAuth,
  });

  @GET('/ObtenerPagos')
  Future<ResponseModel> getPaymentsFromReference({ @Query('referencia') required String referencia, ... });
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

**State** (`part of 'payment_bloc.dart'`):
```dart
@freezed
class PaymentState with _$PaymentState {
  const factory PaymentState.initial() = _Initial;
  const factory PaymentState.loading({
    required bool isGeneratingLink,
    required bool isFetchingPayments,
    List<PaymentEntity>? payments,         // Datos preservados durante loading
    PaymentGeneratedEntity? paymentGenerated,
  }) = _Loading;
  const factory PaymentState.error({
    required String message,
    List<PaymentEntity>? payments,
  }) = _Error;
  const factory PaymentState.loaded({
    List<PaymentEntity>? payments,
    PaymentGeneratedEntity? paymentGenerated,
  }) = _Loaded;
}

// Extension para accesores cómodos en la UI
extension PaymentStateX on PaymentState {
  bool get isLoading => maybeMap(loading: (_) => true, orElse: () => false);
  List<PaymentEntity>? get payments => mapOrNull(
    loaded: (s) => s.payments,
    loading: (s) => s.payments,   // Preservar datos en loading
    error: (s) => s.payments,
  );
}
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

## 6. Inyección de Dependencias (GetIt)

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

  // 2. Clientes HTTP
  getIt.registerSingleton<ExitusClient>(ExitusClient(getIt<ConnectivityEventBus>(), getIt<ConnectivityHelper>()));
  getIt.registerSingleton<ExitusClientDirect>(ExitusClientDirect(...));

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
  getIt.registerFactory<PaymentsDataSource>(
    () => PaymentsDataSource(getIt<ExitusClient>().dio),
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

## 7. Rutas

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

## 8. Barrel Exports — Convención

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

## 9. Reglas de Code Generation

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

## 10. Checklist para Crear un Nuevo Feature

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

## 11. Patrones Clave

### Preservar datos durante loading
Los estados `loading` y `error` del BLoC llevan los mismos campos opcionales que `loaded`, permitiendo que la UI muestre datos previos con skeleton/shimmer mientras se actualizan.

### Singletons con `.instance`
Los Helpers (ConnectivityHelper, SharedPreferenceHelper, etc.) implementan el patrón Singleton clásico con `static final instance = HelperClass._()` para garantizar una sola instancia sin GetIt.

### DataStateFactory
Factory estática que mapea automáticamente `DioException` a mensajes user-friendly según el código HTTP (401, 403, 404, 500, timeout, etc.) y loguea con Logger.

### ToastContract
Interfaz en `core/contracts/` implementada por `ToastHelper` en `core/helpers/`. Se registra como `ToastContract` en GetIt, permitiendo mockear en tests.
