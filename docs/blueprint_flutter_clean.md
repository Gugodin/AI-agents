---
name: blueprint-flutter-clean
description: Blueprint de arquitectura Flutter Clean Architecture - Estructura completa del proyecto AppCobranza con DataState, Freezed, BLoC, GetIt y más
---

# Blueprint: Arquitectura Flutter — AppCobranza (Exitus)

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

### 3.2 DataStateFactory

Mapeo de errores con mensajes user-friendly:

```dart
// 400 → "Los datos enviados no son válidos"
// 401 → "Tus credenciales han expirado"
// 403 → "No tienes permisos"
// 404 → "No pudimos encontrar lo que buscas"
// 500/502/503 → "El servidor está teniendo problemas"
// Timeout → "La conexión tardó demasiado"
```

### 3.3 DataResult<T>

```dart
typedef DataResult<T> = Future<DataState<T>>;
typedef DataResultVoid = Future<DataState<VoidSuccess>>;
class VoidSuccess { const VoidSuccess(); }
```

### 3.4 UseCase

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

---

## 4. Feature Pattern

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
└── presentation/
    ├── presentation.dart
    └── bloc/
        └── {name}_bloc/
```

### Domain Layer

- **Entity**: POJO puro con `factory Entity.mock()`
- **Repository Interface**: `abstract class` con métodos que retornan `DataResult<T>`
- **UseCase**: Extiende `UseCaseWithParams` o `UseCaseWithoutParams`

### Data Layer

- **Model**: `fromJson`, `toJson`, `toEntity()`
- **DataSource**: `@RestApi` con Retrofit
- **RepositoryImpl**: Implementa la interfaz, usa `DataStateFactory`

### Presentation Layer

- **Event**: `@freezed` con factories por cada acción
- **State**: `@freezed` con estados `initial`, `loading`, `error`, `loaded`
- **StateX Extension**: Verificadores (`isLoading`, `isError`) y accesores cross-state
- **Bloc**: Handlers que emiten estados preservando datos

---

## 5. Inyección de Dependencias

```dart
void setupInjector() {
  // Singletons globales
  getIt.registerSingleton<SharedPreferenceHelper>(SharedPreferenceHelper.instance);
  getIt.registerSingleton<ToastContract>(ToastHelper.instance);

  // Clientes HTTP
  getIt.registerSingleton<AppClient>(AppClient(...));

  // BLoCs globales
  getIt.registerLazySingleton<ConnectivityBloc>(ConnectivityBloc());

  // Features
  payments_dependencies();
}
```

Por feature:
```dart
void payments_dependencies() {
  getIt.registerFactory<PaymentsDataSource>(() => PaymentsDataSource(getIt<AppClient>().dio));
  getIt.registerFactory<PaymentsRepository>(() => PaymentsRepositoryImpl(...));
  getIt.registerFactory<GeneratePaymentLinkUseCase>(() => GeneratePaymentLinkUseCase(...));
  getIt.registerLazySingleton<PaymentBloc>(() => PaymentBloc(...));
}
```

---

## 6. Main Entry Point

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

## 7. App Root

```dart
class App extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final clampedScale = MediaQuery.of(context).textScaler.scale(1.0).clamp(1.0, 1.3);

    return MediaQuery(
      data: MediaQuery.of(context).copyWith(textScaler: TextScaler.linear(clampedScale)),
      child: ToastificationWrapper(
        child: MultiBlocProvider(
          providers: [
            BlocProvider<ConnectivityBloc>(create: (_) => getIt<ConnectivityBloc>()),
          ],
          child: MaterialApp(
            theme: AppTheme.themeLight,
            initialRoute: Routes.initialRoute,
            routes: Routes.routes,
            onGenerateRoute: Routes.onGenerateRoute,
          ),
        ),
      ),
    );
  }
}
```

---

## 8. Checklist Nuevo Feature

```
□ 1. Crear carpeta lib/features/{feature_name}/
□ 2. Domain: entity, repository interface, use cases + params
□ 3. Data: data source (@RestApi), models (fromJson/toJson/toEntity), repository impl
□ 4. Presentation: bloc/event/state + StateX extension
□ 5. dependencies/{name}_dependencies.dart
□ 6. Registrar en setupInjector()
□ 7. flutter pub run build_runner build --delete-conflicting-outputs
□ 8. Agregar rutas
```

---

## 9. Code Generation

```bash
# Una vez
flutter pub run build_runner build --delete-conflicting-outputs

# Watch mode
flutter pub run build_runner watch --delete-conflicting-outputs
```

---

## 10. ToastContract

Interfaz abstracta en `core/contracts/toast_contract/` implementada por `ToastHelper`.

Uso:
```dart
getIt<ToastContract>().showSuccess(message: 'Operación exitosa');
getIt<ToastContract>().showError(message: state.errorMessage);
```

Requisito: `ToastificationWrapper` debe envolver el `MaterialApp`.
