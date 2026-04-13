---
name: feature-creator
description: Generador de features completo (Domain, Data, Presentation) siguiendo el estándar de Clean Architecture para Flutter
---

# Skill: Generador de Features Flexible (v2.2)

## 🎯 Objetivo
Generar un feature completo (Domain, Data, Presentation) siguiendo el estándar de Clean Architecture. La prioridad es la fidelidad al JSON proporcionado, dejando las transformaciones complejas para casos específicos.

## ⚡ Disparador Interactivo (Plantilla de Recolección)
Si el usuario escribe **"Generaré un feature"** o **"Generaremos un nuevo feature"**, el agente DEBE responder EXACTAMENTE:

> "Claro, para generar un feature óptimo necesito los siguientes datos:
> 
> - **Feature**: [Nombre del módulo]
> - **Caso de Uso**: [Nombre] | Path: [Ruta] | Método: [GET/POST]
> - **BaseUrl**: https://learn.microsoft.com/es-es/intune/configmgr/core/clients/deploy/about-client-settings
> - **Body/Params**: [JSON de parámetros]
> - **Observaciones**: [Lógica especial: Ej. Monto * 100, Split de strings, campos 'SIEMPRE', etc.]
> - **Response**: [JSON esperado]"

## 🏗️ Reglas Arquitectónicas

### 1. Capa de Datos (Data)
- **Mapeo de Modelos (Estándar)**: El `Model` debe generarse basándose estrictamente en la estructura del JSON proporcionado en el campo **Response**. Debe incluir obligatoriamente:
    - Métodos de serialización: `fromJson` y `toJson`.
    - Método de transformación: **`toEntity()`**, el cual debe mapear cada propiedad del modelo a su correspondiente en la `Entity` del dominio.
- **Transformaciones**: Solo si se especifica en **Observaciones**, el agente debe aplicar lógica adicional (como splits o cálculos) dentro del `Model` o el `RepositoryImpl`.
- **Retrofit**: Si el campo **BaseUrl** de la plantilla tiene valor, úsalo en `@RestApi(baseUrl: "...")`. Si está vacío, no declares `baseUrl` en la anotación.
- **Manejo de Errores**: Uso obligatorio de `DataStateFactory` con un `idError` único en los bloques catch del `RepositoryImpl`.

### 2. Capa de Dominio (Domain)
- **Entities**: POJOS puros con un `factory Entity.mock()` para facilitar el uso de Skeletons en la UI.
- **UseCases**: Deben extender de `UseCaseWithParams` o `UseCaseWithoutParams` usando `DataResult<T>`.

### 3. Capa de Presentación (BLoC & UI)
- **Estados Granulares (Recomendado)**: Si el feature maneja múltiples acciones, usar booleanos específicos (ej: `isFetchingList`, `isExecutingAction`). Para features simples, se permite un `isLoading` único.
- **Preservación de Datos**: Los estados `loading` y `error` deben incluir los campos de datos (ej: `List<Entity>? items`) para mostrar información previa o Skeletons mientras se actualiza la UI.
- **Extensión FeatureStateX**: Generar siempre la extensión para facilitar el acceso a datos y estados desde la UI (getters `isLoading`, `isError`, `items`, etc.).

### 4. Inyección de Dependencias
- Registrar capas de Datos y Dominio como `factory`.
- Registrar el BLoC como `lazySingleton`.
- Crear el archivo `{feature}_dependencies.dart` y asegurar su exportación.

## 📝 Reglas de Lógica Especial
- **Hard-Coding**: Parámetros marcados como "SIEMPRE" se implementan directamente en la capa de datos.
- **Flexibilidad**: El agente debe interpretar las "Observaciones" como órdenes prioritarias que sobresalen del mapeo estándar del JSON.
