# Skill: Setup Proyecto Clean

## 🎯 Objetivo
Inicializar un proyecto de Flutter desde cero siguiendo estrictamente el "molde" definido en el `BLUEPRINT.md`. Este skill automatiza la creación de la base arquitectónica para que el desarrollador pueda empezar directamente con la lógica de negocio.

## 🔗 Vinculación con Blueprint
Para cada componente generado, el agente DEBE consultar los estándares de implementación en:
- **Estructura de Carpetas**: Ver Sección 2 de `BLUEPRINT.md`.
- **Manejo de Estados**: Ver Sección 3.1 (`DataState`) y 3.2 (`DataResult`).
- **Casos de Uso**: Ver Sección 3.3 (`UseCase`).
- **Red y Clientes**: Ver Sección 4 (`Network Layer`).

## ⚡ Comando de Activación: "Inicializar Proyecto"
Cuando el usuario ejecute este comando, el agente debe realizar las siguientes tareas en orden:

### 1. Configuración de Entorno
- Generar el archivo `pubspec.yaml` incluyendo todas las dependencias del stack tecnológico (Bloc, GetIt, Dio, Retrofit, Freezed, etc.).
- Configurar el archivo `analysis_options.yaml` con reglas de linter estrictas.

### 2. Creación de la Estructura de Directorios
- Crear el árbol de carpetas completo: `app/`, `config/`, `core/`, `features/`, `injector/` y `assets/`.
- Generar todos los archivos **Barrel Exports** (`.dart`) iniciales para cada carpeta.

### 3. Implementación del Core Foundation
- Escribir los archivos base en `lib/core/foundation/`:
    - `DataState<T>` (Sealed class con Freezed).
    - `DataStateFactory` (Lógica de mapeo de errores de red y generales).
- Escribir las interfaces de `UseCase` y el alias `DataResult`.

### 4. Capa de Red y Utilidades
- Crear el `ApiClient` base (Dio) con los interceptores de Conectividad, Token y Logger.
- Implementar el `ResponseModel` para el parseo genérico de respuestas.

### 5. Configuración de Inyección (DI)
- Crear el archivo `lib/injector/injector.dart` con la función `setupInjector()` registrando los Singletons globales (Helpers, API Clients, EventBus).

## 📝 Reglas de Ejecución
- **Nomenclatura**: Usar nombres genéricos (ej. `ApiClient` en lugar de nombres de empresas específicas).
- **Consistencia**: No omitir ningún archivo mencionado en el Blueprint.
- **Confirmación**: Al finalizar, mostrar un resumen de la estructura creada y los comandos de `build_runner` necesarios para generar el código inicial.