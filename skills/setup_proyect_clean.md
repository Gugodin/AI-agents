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
- **DataState & Factory**: Sistema de estados y mapeo de errores `fromDioException`.
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
