---
name: data-state-standard
description: Reglas obligatorias de uso, trazabilidad de errores y protocolos de manejo de estados en la capa de Data y Domain
---

# Skill: Data State Standard - Protocolos de Uso y Trazabilidad

Este documento define las **reglas de uso** y los **protocolos de trazabilidad** para el sistema de estados de la arquitectura. Las definiciones de código (DataState, DataStateFactory, DataStateConvenience, Typedefs) residen en `blueprint-foundation` como fuente única.

> **Referencia obligatoria**: Para las implementaciones de código, consultar `blueprint-foundation > Secciones 1, 1.1, 1.2 y 3`.

## 1. Principios de Trazabilidad de Errores

### 1.1 IDs de Error Unicos
Cada punto de fallo en un `RepositoryImpl` debe tener un identificador unico que permita rastrear el error exacto en produccion:

* **Formato**: `[FEATURE][NUMERO]` (ej: `AUTH01`, `USER05`, `INV03`)
* **Unicidad**: Dentro del mismo repositorio, NUNCA se repite un ID
* **Descriptividad**: El prefijo debe identificar la feature (AUTH, USER, INV, CAT, etc.)
* **Secuencialidad**: Los IDs deben ser correlativos dentro de cada feature para facilitar el debugging

### 1.2 Formato de Mensajes al Usuario
Todo `userMessage` que llega a la UI debe concatenar el ID de error al final:
```
"Mensaje descriptivo para el usuario (ERR_[ID])"
```

Ejemplos:
* `"No pudimos encontrar lo que buscas (ERR_AUTH01)"`
* `"Tus credenciales han expirado (ERR_LOGIN03)"`

### 1.3 Logging Obligatorio
Todo error debe disparar un `debugPrint` con formato visual que separe:
* **Errores de Red (Dio)**: Tipo, codigo HTTP, URL, archivo, mensaje usuario, mensaje tecnico
* **Errores Generales**: Archivo, mensaje usuario, mensaje tecnico, stacktrace

## 2. Protocolo de Uso en RepositoryImpl

### 2.1 Estructura Obligatoria de Try-Catch
Todo metodo async en un `RepositoryImpl` debe seguir esta estructura:

```
try {
  // 1. Ejecutar logica (llamar DataSource, helpers, etc.)
  // 2. Validar ResponseModel.error si aplica
  // 3. Retornar DataState.success con la entidad mapeada
} on DioException catch (e, stackTrace) {
  // Usar DataStateFactory.fromDioException con idError UNICO
} catch (e, stackTrace) {
  // Usar DataStateFactory.fromException con idError UNICO
}
```

### 2.2 Validacion de ResponseModel
Cuando el DataSource retorna un `ResponseModel`, el repositorio DEBE:
1. Verificar la propiedad `.error` del ResponseModel
2. Si `error == true`: Retornar `DataState.generalError` con el mensaje de la API
3. Si `error == false`: Transformar `.data` a la Entidad correspondiente y retornar `DataState.success`

### 2.3 Campos Obligatorios en Cada Llamada a DataStateFactory
Toda invocacion a `DataStateFactory.fromDioException` o `fromException` debe incluir:

| Campo | Descripcion | Ejemplo |
|-------|-------------|---------|
| `module` | Nombre de la clase del repositorio | `'AuthRepositoryImpl'` |
| `file` | Nombre del archivo | `'auth_repository_impl.dart'` |
| `line` | Numero de linea del catch | `'89'` |
| `stackTrace` | Stacktrace como String | `stackTrace.toString()` |
| `idError` | ID unico del error | `'AUTH01'` |

## 3. Protocolo de Propagacion en UseCases

### 3.1 Regla de No-Duplicacion de Logs
Cuando un UseCase orquestador consume otros UseCases, NUNCA debe volver a llamar a `DataStateFactory`. El log original ya fue generado en el repositorio.

### 3.2 Uso de propagateError
Si un sub-proceso falla (`result.isError`), se DEBE usar `propagateError<T>()` para transferir el error preservando la trazabilidad original:

```dart
final result = await _subUseCase.call();
if (result.isError) {
  return result.propagateError<AllCatalogsEntity>();
}
final data = result.dataOrNull!;
```

### 3.3 Prohibicion de Try-Catch en UseCases
Los UseCases NUNCA deben tener bloques `try-catch`. Si hay un error, debe venir del Repositorio ya mapeado en un `DataState`.

## 4. Protocolo de Consumo en BLoC

### 4.1 Mapeo Exhaustivo con .when()
El BLoC debe usar `.when()` (no `.maybeWhen()`) para procesar el `DataResult`, manejando explicitamente:
1. **success**: Emitir estado con la data de la entidad
2. **dioError**: Emitir estado de error con detalles de red
3. **generalError**: Emitir estado de error para fallos logicos

### 4.2 Uso de Extensiones en UI
La UI NUNCA debe hacer casting manual (`state is SuccessState`). Siempre usar los getters de la extension del estado (`state.isLoading`, `state.items`, `state.errorMessage`).

## 5. Reglas de Verificacion para el Auditor (Data State Edition)

### 5.1 Trazabilidad Completa
1. **Generacion de Codigo**: Verificar que `data_state.dart` incluya `part 'data_state.freezed.dart'` (definicion en `blueprint-foundation`)
2. **Campos Obligatorios**: Todo estado de error debe capturar `module`, `file`, `line`, `stackTrace` e `idError`
3. **IDs Unicos**: Rechazar cualquier Repository con IDs genericos (`ERROR01`) o repetidos dentro del mismo archivo

### 5.2 Protocolo de Repository
1. **Cobertura Try-Catch**: Todo metodo async debe estar envuelto en try-catch
2. **Uso de Factory**: Los errores se mapean SOLO a traves de `DataStateFactory`, nunca con `DataState.error` manual
3. **Validacion ResponseModel**: Siempre verificar `.error` antes de mapear data a entidad

### 5.3 Protocolo de UseCase
1. **Sin Try-Catch**: Los UseCases no deben tener bloques try-catch
2. **Propagacion**: Usar `propagateError()` en lugar de re-crear errores
3. **Verificacion Previa**: Siempre validar `.isError` antes de acceder a `.dataOrNull!`

### 5.4 Protocolo de BLoC
1. **Exhaustividad**: Usar `.when()` en lugar de `.maybeWhen()` para el DataResult
2. **Extensiones**: La UI debe usar getters de extension, no casting manual
