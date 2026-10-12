---
name: dependency-manager
description: Reglas para la gestion de dependencias, ciclo de vida de objetos y configuracion del Service Locator (GetIt).
---

# Skill: Dependency Manager - El Tejido Conectivo

Este manual define como registrar y proveer las dependencias en toda la aplicacion. Un manejo correcto asegura que los componentes sean testeables, desacoplados y eficientes en memoria.

## 1. Centralizacion con GetIt
Utilizamos **GetIt** como nuestro Service Locator oficial. Todas las dependencias de una feature deben estar registradas antes de ser consumidas por la UI o por otros componentes.

### 1.1 El Directorio Injector
Toda la configuracion de inyeccion reside en `lib/injector/`. 
* **injector.dart**: El archivo principal que inicializa el contenedor global (`getIt`).
* **feature_injector.dart**: Se requiere separar la inyeccion por modulos o features para evitar que un solo archivo crezca indefinidamente.

### 1.2 Jerarquia de Registro (Orden de Capas)
Para evitar errores de "dependencia no encontrada", el registro debe seguir un orden logico de abajo hacia arriba:
1. **Core/External**: Dio, SharedPreferences, SecureStorage, Helpers.
2. **DataSources**: Requieren el cliente de red (Dio).
3. **Repositories**: Requieren los DataSources y Helpers.
4. **UseCases**: Requieren los Repositories.
5. **BLoCs**: Requieren los UseCases.

### 1.3 Ciclo de Vida: Factory vs Singleton
* **registerFactory**: Se usa para **Casos de uso**, **DataSources**, **Repositories** y **BLoCs de feature**. Cada vez que se solicita, se crea una instancia nueva. Esto garantiza la limpieza de estados al cerrar pantallas.
* **registerSingleton**: Se usa EXCLUSIVAMENTE para **Helpers** (ConnectivityHelper, LocationHelper, etc.), **Clientes HTTP** (ExitusClient) y **BLoCs globales** (OverlayBloc, ConnectivityBloc). Los BLoCs de feature NUNCA deben ser Singleton.

### 1.4 Ejemplo de Implementacion de archivo principal
```dart
final GetIt getIt = GetIt.instance;

void setupInjector() {
  // EVENT BUS Y HELPERS
  getIt.registerSingleton<ConnectivityEventBus>(ConnectivityEventBus.instance);

  getIt.registerSingleton<ConnectivityHelper>(ConnectivityHelper.instance);
  getIt.registerSingleton<LocationHelper>(LocationHelper.instance);
  getIt.registerSingleton<BiometryicsHelper>(BiometryicsHelper.instance);
  getIt.registerSingleton<PermissionsHelper>(PermissionsHelper.instance);
  getIt.registerSingleton<SharedPreferenceHelper>(
      SharedPreferenceHelper.instance);
  getIt.registerSingleton<SecureStorageHelper>(SecureStorageHelper.instance);
  getIt.registerSingleton<ToastContract>(ToastHelper.instance);
  getIt.registerSingleton<ImageSelectorHelper>(ImageSelectorHelper.instance);

  // CLIENTES HTTP

  // ExitusClient apuntando al middleware
  getIt.registerSingleton<ExitusClient>(
      ExitusClient(getIt<ConnectivityEventBus>(), getIt<ConnectivityHelper>()));

  // ExitusClientDirect apuntando a la BACKEND directo
  getIt.registerSingleton<ExitusClientDirect>(ExitusClientDirect(
      getIt<ConnectivityEventBus>(), getIt<ConnectivityHelper>()));

  // Blocs globales
  getIt.registerSingleton<OverlayBloc>(OverlayBloc());
  getIt.registerSingleton<HomeOrchestrationBloc>(HomeOrchestrationBloc());
  getIt.registerSingleton<ConnectivityBloc>(ConnectivityBloc());

  // DEPENDENCIAS DE FEATURES
  auth_dependencies();
  location_dependencies();
  catalog_dependencies();
  asignations_dependencies();
  mapping_services_dependencies();
  route_dependencies();
  gestions_dependencies();
  payments_dependencies();
}
``` 

#### 1.5 Ejemplo de Implementacion de archivo de feature
```dart
void auth_dependencies() {
  // Data sources
  getIt.registerFactory<AuthDataSource>(
      () => AuthDataSource(getIt<ExitusClient>().dio));

  // Repositorios implementados con ExitusClient
  getIt.registerFactory<AuthRepository>(
    () => AuthRepositoryImpl(
      remoteDataSource: getIt<AuthDataSource>(),
      locationHelper: getIt<LocationHelper>(),
      sharedPreferenceHelper: getIt<SharedPreferenceHelper>(),
      secureStorageHelper: getIt<SecureStorageHelper>(),
      biometricsHelper: getIt<BiometryicsHelper>(),
    ),
  );

  // Casos de uso
  getIt.registerFactory<LoginUseCase>(
    () => LoginUseCase(getIt<AuthRepository>()),
  );
  getIt.registerFactory<BiometricLoginUseCase>(
    () => BiometricLoginUseCase(getIt<AuthRepository>()),
  );
  getIt.registerFactory<LogoutUseCase>(
    () => LogoutUseCase(getIt<AuthRepository>()),
  );

  // Bloc auth (registerFactory para BLoCs de feature)
  getIt.registerFactory<AuthBloc>(() => AuthBloc(
    loginUseCase: getIt<LoginUseCase>(),
    biometricLoginUseCase: getIt<BiometricLoginUseCase>(),
    logoutUseCase: getIt<LogoutUseCase>(),
  ));
}
```

### 2. Reglas de Verificacion para el Auditor (Dependency Edition)
El Auditor debe ser extremadamente vigilante con el ciclo de vida de los objetos para evitar fugas de memoria o errores de estado compartido.

## 2.1 Control de Instanciacion

Cero New/Constructores Manuales: El Auditor debe rechazar cualquier codigo en las capas de Domain, Data o Presentation que instancie una dependencia manualmente (ej: final repo = AuthRepositoryImpl(...)). Todo debe venir de la inyeccion.

Uso de Interfaces: Se debe verificar que se registre la Interfaz y se provea la Implementacion (ej: getIt.registerLazySingleton<Repository>(() => RepositoryImpl())).

## 2.2 Validacion de BLoCs

BLoCs de Feature como Factory: Es un ERROR CRITICO registrar un BLoC de feature como Singleton. El Auditor debe obligar al uso de registerFactory para asegurar la limpieza de estados al cerrar pantallas.

Excepcion para BLoCs Globales: Los BLoCs que viven durante todo el ciclo de vida de la app (OverlayBloc, ConnectivityBloc, HomeOrchestrationBloc) pueden registrarse como Singleton en el archivo principal del injector. El Auditor debe verificar que estos BLoCs globales esten justificados y documentados.

## 2.3 Orden y Limpieza

Dependencias Circulares: Si el Auditor detecta que el Objeto A necesita al B, y el B al A, debe ordenar una refactorizacion inmediata (posiblemente falta un UseCase intermedio).

Registro Unico: Verificar que no existan registros duplicados para el mismo tipo, lo cual causaria una excepcion en tiempo de ejecucion de GetIt.

## 2.4 Acceso en UI

Solo via Context o GetIt: En la UI, el acceso debe ser preferiblemente a traves de getIt<T>() dentro del MultiBlocProvider en la raiz, como se definio en el UI Specialist.