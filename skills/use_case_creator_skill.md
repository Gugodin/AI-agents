# Skill: Creador de Casos de Uso Quirúrgico (v2.2)

## 🎯 Objetivo
Añadir una nueva funcionalidad (caso de uso) a un feature existente. El agente debe actualizar todas las capas (Domain, Data y DI) de forma precisa, respetando el código previo y siguiendo los estándares de `BLUEPRINT.md`.

## ⚡ Disparador Interactivo (Plantilla de Recolección)
Si el usuario escribe **"Generaré un caso de uso"** o **"Añadir nuevo caso de uso"**, el agente DEBE responder EXACTAMENTE:

> "Claro, para generar un feature óptimo necesito los siguientes datos:
> 
> - **Feature**: [Nombre del módulo existente]
> - **Caso de Uso**: [Nombre] | Path: [Ruta] | Método: [GET/POST]
> - **BaseUrl**: https://learn.microsoft.com/es-es/intune/configmgr/core/clients/deploy/about-client-settings
> - **Body/Params**: [JSON de parámetros]
> - **Observaciones**: [Lógica especial: Ej. Monto * 100, Split de strings, campos 'SIEMPRE', etc.]
> - **Response**: [JSON esperado]"

## 🏗️ Pasos de Ejecución Quirúrgica

### 1. Capa Domain
- **Repository Interface**: Añadir la firma del nuevo método al repositorio abstracto del feature.
- **UseCase Class**: Crear el archivo del nuevo caso de uso extendiendo de `UseCaseWithParams` o `UseCaseWithoutParams`.
- **Params**: Generar una clase de parámetros dedicada para este caso de uso.

### 2. Capa Data
- **DataSource (Retrofit)**: Añadir el nuevo método con su anotación HTTP. Si **BaseUrl** tiene valor, generar un nuevo DataSource específico o usar el atributo `baseUrl` en la anotación si el proyecto lo permite.
- **Model**: Generar un nuevo `Model` basado en el **Response** proporcionado. Debe incluir `fromJson`, `toJson` y el método **`toEntity()`**.
- **RepositoryImpl**: Implementar el nuevo método usando obligatoriamente `DataStateFactory` para capturar `DioException` y excepciones generales con un `idError` único.

### 3. Persistencia Opcional
- Si las **Observaciones** indican que el caso de uso debe guardar o leer datos locales, el agente debe proponer la creación de un nuevo **Mixin** en `SharedPreferenceHelper` (ej: `FeatureNameMixin`) para manejar esa persistencia de forma limpia.

### 4. Inyección de Dependencias (DI)
- Registrar el nuevo `UseCase` como `factory` en el archivo `{feature}_dependencies.dart`.
- Si se creó un nuevo `DataSource` por un `BaseUrl` distinto, registrarlo también en este archivo.

## 📝 Reglas de Lógica Especial
- **Fidelidad al JSON**: El mapeo inicial debe ser idéntico al **Response**. Las transformaciones complejas (splits, cálculos) se aplican solo si están en **Observaciones**.
- **Hard-Coding**: Parámetros marcados como "SIEMPRE" se queman en el Data/Repository, evitando pedirlos en el UseCase.
- **Reutilización de Parámetros**: Si varios campos de la API usan el mismo dato, el UseCase solo debe recibir un parámetro y el Repository se encarga de distribuirlo.
- **Barrel Exports**: Asegurar que los nuevos archivos (UseCase, Model, Entity) sean exportados en sus respectivos archivos barrel.