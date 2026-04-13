---
name: use-case-creator
description: Añadir nueva funcionalidad (caso de uso) a un feature existente en Clean Architecture
---

# Skill: Creador de Casos de Uso Quirúrgico (v2.3)

## 🎯 Objetivo
Añadir una nueva funcionalidad (caso de uso) a un feature existente. El agente debe actualizar todas las capas (Domain, Data y DI) de forma precisa, respetando el código previo y siguiendo los estándares del blueprint.

## ⚡ Disparador Interactivo (Plantilla de Recolección)
Si el usuario escribe **"Generaré un caso de uso"** o **"Añadir nuevo caso de uso"**, el agente DEBE responder EXACTAMENTE:

> "Claro, para añadir un caso de uso necesito los siguientes datos:
> 
> - **Feature**: [Nombre del módulo existente]
> - **Caso de Uso**: [Nombre] | Path: [Ruta] | Método: [GET/POST]
> - **BaseUrl**: https://learn.microsoft.com/es-es/intune/configmgr/core/clients/deploy/about-client-settings
> - **Body/Params**: [JSON de parámetros]
> - **Observaciones**: [Lógica especial: Ej. Monto * 100, Split de strings, campos 'SIEMPRE', etc.]
> - **Response**: [JSON esperado]
> - **Implementación**: ¿Deseas que genere el **cascaron** (sin implementación) o el caso de uso **completo** (con lógica implementada)?"

## 🏗️ Pasos de Ejecución Quirúrgica

### 1. Capa Domain
- **Repository Interface**: Añadir la firma del nuevo método al repositorio abstracto del feature.
- **UseCase Class**: Crear el archivo del nuevo caso de uso extendiendo de `UseCaseWithParams`, `UseCaseWithoutParams`, `UseCaseVoid` o `UseCaseVoidWithoutParams`.
- **Params**: Los parámetros deben definirse **DENTRO del mismo archivo** del UseCase, justo después de la clase del UseCase. NO crear archivos separados para los params.

  **Estructura correcta:**
  ```dart
  class GetUserByIdUseCase extends UseCaseWithParams<User, GetUserByIdParams> {
    final UserRepository _repository;

    GetUserByIdUseCase(this._repository);

    @override
    DataResult<User> call(GetUserByIdParams params) async {
      return await _repository.getUserById(params.id);
    }
  }

  class GetUserByIdParams {
    final String id;

    const GetUserByIdParams({required this.id});
  }
  ```

### 2. Capa Data
- **DataSource (Retrofit)**: 
    - Añadir el nuevo método con su anotación HTTP (`@GET`, `@POST`, etc.)
    - Incluir `part 'nombre_data_source.g.dart';`
    - Si **BaseUrl** tiene valor, usar `@RestApi(baseUrl: "...")`
- **Model**: Generar un nuevo `Model` basado en el **Response** proporcionado. Debe incluir `fromJson`, `toJson` y el método **`toEntity()`**.
- **RepositoryImpl**: Implementar el nuevo método usando las extensiones de `DataState` (`isSuccess`, `isDioError`, `dataOrNull`, `userMessageOrNull`)

### 3. Persistencia Opcional
- Si las **Observaciones** indican que el caso de uso debe guardar o leer datos locales, el agente debe proponer la creación de un nuevo **Mixin** en `SharedPreferenceHelper` (ej: `FeatureNameMixin`) para manejar esa persistencia de forma limpia.

### 4. Inyección de Dependencias (DI)
- Registrar el nuevo `UseCase` como `factory` en el archivo `{feature}_dependencies.dart`.
- Si se creó un nuevo `DataSource`, registrarlo también.

## ⚠️ IMPORTANTE - Code Generation

> Si se creó o modificó un DataSource con Retrofit, **DEBE** ejecutarse:
> ```bash
> flutter pub run build_runner build --delete-conflicting-outputs
> ```
> Sin este paso, el código NO compilará.

## 📝 Reglas de Lógica Especial
- **Fidelidad al JSON**: El mapeo inicial debe ser idéntico al **Response**. Las transformaciones complejas (splits, cálculos) se aplican solo si están en **Observaciones**.
- **Hard-Coding**: Parámetros marcados como "SIEMPRE" se queman en el Data/Repository, evitando pedirlos en el UseCase.
- **Reutilización de Parámetros**: Si varios campos de la API usan el mismo dato, el UseCase solo debe recibir un parámetro y el Repository se encarga de distribuirlo.
- **Barrel Exports**: Asegurar que los nuevos archivos (UseCase, Model, Entity) sean exportados en sus respectivos archivos barrel.

## 🏗️ Modo Cascaron (Skeleton)

Si el usuario indica que desea generar **"solo el cascaron"** o **"sin implementación"**, el agente debe:

1. **Crear la estructura completa** del caso de uso (Domain, Data)
2. **NO implementar la lógica interna** - dejar los métodos vacíos o con `throw UnimplementedError()`
3. **SÍ incluir**:
   - UseCase con la estructura correcta (clase + params en mismo archivo)
   - DataSource con Retrofit (anotaciones @Get, @Post, etc.)
   - Model con `fromJson`, `toJson`, `toEntity()`
   - Repository interface con firma del método
   - RepositoryImpl con estructura pero sin lógica

**Ejemplo de UseCase en modo cascaron:**
```dart
class GetUserByIdUseCase extends UseCaseWithParams<User, GetUserByIdParams> {
  final UserRepository _repository;

  GetUserByIdUseCase(this._repository);

  @override
  DataResult<User> call(GetUserByIdParams params) async {
    // TODO: Implementar getUserById
    throw UnimplementedError('GetUserByIdUseCase - Implementar lógica');
  }
}

class GetUserByIdParams {
  final String id;

  const GetUserByIdParams({required this.id});
}
```

**El agente debe preguntarle al usuario:** "¿Deseas que genere el cascaron completo (sin implementación) o el caso de uso completo (con lógica implementada)?"

## ⚠️ Recordatorio Post-Creación

Después de añadir un caso de uso que implique modificar un DataSource con Retrofit, el agente DEBE informar al usuario:

> "✅ Caso de uso añadido exitosamente. 
> ⚠️ IMPORTANTE: Si creaste o modificate un DataSource con Retrofit, necesitas ejecutar:
> `flutter pub run build_runner build --delete-conflicting-outputs`
> 
> Esto regenerará los archivos `.g.dart` necesarios."