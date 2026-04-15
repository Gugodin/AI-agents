---
name: setup-proyect
description: Inicializar estructura completa, Core Foundation y configuración nativa de un proyecto Flutter
---

# Skill: Setup Proyecto Clean (v2.3)

## 🎯 Objetivo
Inicializar la estructura completa, el Core Foundation y la configuración nativa de un proyecto Flutter. Este skill asegura un arranque profesional, flexible y libre de errores de inicialización tanto en Dart como en las capas nativas.

## ⚡ Disparador Interactivo: "Inicializar Proyecto"
Si el usuario solicita arrancar un proyecto nuevo, el agente DEBE responder solicitando estos datos:

> "¡Excelente! Vamos a preparar la base de tu nuevo proyecto. Para que todo quede configurado correctamente, por favor indícame:
> 
> - **Nombre del Proyecto**: [ej: app_gestion (en snake_case para imports)]
> - **Organización**: [ej: com.tuempresa (para el Bundle ID/Package Name)]
> - **Entornos**: [¿Solo Prod o también QA/Direct?]
> - **Módulos Core Opcionales**: [¿Deseas incluir Toastification, SharedPreferences, Biometría o Connectivity ahora mismo?]
> 
> Una vez recibidos, generaré el código Dart y las configuraciones nativas necesarias."

## 🏗️ Pasos de Ejecución

### 1. Configuración de Entorno (pubspec.yaml)
Generar el `pubspec.yaml` con el nombre del proyecto y las dependencias base (Bloc, GetIt, Dio, Retrofit, Freezed). Añadir los módulos opcionales seleccionados por el usuario.

### 2. Ciclo de Vida Crítico (main.dart)
Generar el `main.dart` con el orden de arranque obligatorio:
1. `WidgetsFlutterBinding.ensureInitialized();`
2. `await initializeDateFormatting();`
3. `setupInjector();`
4. `await getIt<SharedPreferenceHelper>().init();` (Si hay persistencia).
5. `SystemChrome.setPreferredOrientations([...]);`
6. `runApp(const App());`

### 3. Estructura de Raíz (app/app.dart)
Configurar el widget `App` incluyendo:
- **ToastificationWrapper**: Envolviendo al `MaterialApp` (si se seleccionó el módulo).
- **MediaQuery Clamp**: Implementar el `textScaler` con clamp (1.0 a 1.3).
- **MultiBlocProvider**: Proveer BLoCs globales registrados en GetIt.

### 4. Core Foundation & Helpers
- **DataState & Factory**: Sistema de estados con `@freezed sealed class` Y **OBLIGATORIO** incluir `DataStateFactory` con:
  - `fromDioException(DioException, ...)` - factory estático para errores HTTP
  - `fromException(Exception, ...)` - factory estático para errores genéricos
  - `success(T data)` - factory estático para éxito
  - Include mapping de DioException a mensajes user-friendly en el factory
- **Extensions Obligatorias**:
  - `DataStateX` - verificadores (`isSuccess`, `isError`, etc.) y accesores (`dataOrNull`, `userMessageOrNull`)
  - `DataStatePropagateError` - método `propagateError<R>()` para cambiar tipo preservando error
- **SharedPreferenceHelper**: Base con soporte para **Mixins** por dominio.
- **ApiClient**: Configuración de `Dio` con interceptores.

### 5. Configuración Nativa (Android & iOS)
Según los módulos opcionales elegidos, el agente debe proporcionar:
- **Android**: 
    - Permisos en `AndroidManifest.xml` (Internet, Biometrics, Location).
    - Ajuste de `minSdkVersion` en `build.gradle`.
    - Cambio a `FlutterFragmentActivity` en `MainActivity.kt` si se usa biometría.
- **iOS**:
    - Llaves de privacidad en `Info.plist` (Location, FaceID, etc.) con descripciones profesionales.
    - Configuración de plataforma en el `Podfile`.

## 📝 Reglas de Oro
- **Consistencia de Imports**: Todos los archivos generados deben usar el **Nombre del Proyecto** proporcionado para los imports internos (`import 'package:nombre/...'`).
- **Modularidad**: Tratar las configuraciones nativas como opcionales; solo generarlas si el paquete correspondiente está en el pubspec.
- **Barrel Exports**: Mantener la convención de archivos `.dart` que re-exportan el contenido de sus carpetas.

## 🔍 VERIFICACIÓN FINAL (OBLIGATORIA)
Antes de reportar "completado", el agente DEBE ejecutar esta verificación. Si algo falta, debe crearlo antes de finalizar.

### Checklist de Componentes Obligatorios

#### 1. DataState (lib/core/foundation/data_state/data_state.dart)
- [ ] `@freezed sealed class DataState<T>` con:
  - [ ] `DataState.success(T data)`
  - [ ] `DataState.dioError({required DioException error, ...})`
  - [ ] `DataState.generalError({required String userMessage, ...})`
- [ ] `DataStateFactory` con:
  - [ ] `fromDioException(DioException, ...)` - incluye mapping a mensajes user-friendly
  - [ ] `fromException(Exception, ...)`
  - [ ] `success(T data)`
- [ ] Extension `DataStateX`:
  - [ ] `isSuccess`, `isDioError`, `isGeneralError`, `isError`
  - [ ] `dataOrNull`, `userMessageOrNull`, `technicalMessageOrNull`
- [ ] Extension `DataStatePropagateError`:
  - [ ] `propagateError<R>()` - cambia tipo preservando error

#### 2. UseCases (lib/core/foundation/use_case/use_case.dart)
- [ ] `UseCaseWithParams<T, Params>`
- [ ] `UseCaseVoid<Params>`
- [ ] `UseCaseWithoutParams<T>`
- [ ] `UseCaseVoidWithoutParams`

#### 3. Helpers (lib/core/helpers/)
- [ ] `SharedPreferenceHelper` con soporte para mixins por dominio
- [ ] `SecureStorageHelper`
- [ ] `ToastHelper` (implementando `ToastContract`)
- [ ] `PermissionHelper`

#### 4. Core
- [ ] `ConnectivityEventBus` (Singleton con stream)
- [ ] `AppBlocObserver`
- [ ] `AppClient` con interceptores (Logger, Token, Connectivity)
- [ ] `DirectClient`

#### 5. Injector (lib/injector/injector.dart)
- [ ] `setupInjector()` con:
  - [ ] Singleton de SharedPreferenceHelper
  - [ ] Singleton de SecureStorageHelper
  - [ ] Singleton de PermissionHelper
  - [ ] Singleton de ToastContract
  - [ ] Singleton de ConnectivityEventBus
  - [ ] AppClient y DirectClient
  - [ ] `await getIt<SharedPreferenceHelper>().init()`
  - [ ] `await getIt<ConnectivityEventBus>().init()`

#### 6. App y main.dart
- [ ] `StokiApp` (o nombre del proyecto) con:
  - [ ] `ToastificationWrapper`
  - [ ] MediaQuery textScaler clamp (1.0 a 1.3)
- [ ] `main.dart` con orden correcto:
  - [ ] `WidgetsFlutterBinding.ensureInitialized()`
  - [ ] `setupInjector()`
  - [ ] `Bloc.observer = AppBlocObserver()`
  - [ ] `runApp()`

#### 7. Config (lib/config/)
- [ ] `AppTheme` con `AppColors`
- [ ] `Routes` con `onGenerateRoute`

#### 8. Configuración Nativa
- [ ] Android:
  - [ ] `AndroidManifest.xml` con permisos (INTERNET, ACCESS_NETWORK_STATE, etc.)
  - [ ] `minSdk = 23` en build.gradle.kts
- [ ] iOS:
  - [ ] `Info.plist` con permisos de privacidad (Location, Camera, FaceID, etc.)
  - [ ] Orientación portrait only

#### 9. Build
- [ ] `flutter pub get` exitoso
- [ ] `flutter pub run build_runner build --delete-conflicting-outputs` exitoso
- [ ] `flutter analyze` con 0 errores

### 📋 Prompt de Finalización
Al terminar, el agente DEBE mostrar:

```
✅ Proyecto [nombre] configurado exitosamente.

Resumen de verificación:
- DataState: ✓ Completo
- UseCases: ✓ Completo  
- Helpers: ✓ Completo
- Core: ✓ Completo
- Injector: ✓ Completo
- App: ✓ Completo
- Config: ✓ Completo
- Nativo: ✓ Completo
- Build: ✓ Sin errores

¿Deseas que inicie con la primera feature? [Auth] [Home] [Otra]
```
