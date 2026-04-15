---
name: blueprint-foundation
description: Reglas de oro para la base de datos (DataState) y logica de negocio (UseCases) en AppCobranza
---

# Skill: Foundation - El ADN de la Arquitectura

Este documento define las estructuras de datos y contratos de ejecucion obligatorios para todo el proyecto. Ningun agente puede desviarse de estas definiciones bajo ninguna circunstancia.

## 1. DataState (Inmutabilidad con Freezed)
Es OBLIGATORIO el uso de @freezed para el manejo de estados de datos. No se permiten sealed classes nativas fuera de Freezed para evitar inconsistencias estructurales.

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

## 1.1 DataStateFactory (Mapeo, Logging e IDs de Error)
La DataStateFactory es una extension que automatiza la creacion de estados de error, garantizando que todo fallo sea rastreable y amigable para el usuario final.

### Reglas de Mensajes e Identificadores (IDs)
1. **Unicidad**: Cada llamada en el repositorio debe asignar un idError unico (ej: 'AUTH01', 'USER05').
2. **Formato de Usuario**: El mensaje final para el usuario (userMessage) DEBE concatenar el ID al final con el formato: "Mensaje de error (ERR_idError)".
3. **Logging Obligatorio**: Todo error debe disparar un print en consola (debugPrint) con un formato visual claro que separe la informacion de Red (Dio) de los errores Generales.

### Implementacion de la Fabrica (Extension)
```dart
extension DataStateFactory<T> on DataState<T> {
  static DataState<T> fromDioException<T>(
    DioException e, {
    required String module,
    required String file,
    required String line,
    required String stackTrace,
    required String idError,
  }) {
    final statusCode = e.response?.statusCode;
    String userMessage;
    String technicalMessage = 'DioException: ${e.type} - Status: $statusCode - ${e.message}';

    switch (statusCode) {
      case 404: userMessage = 'No pudimos encontrar lo que buscas'; break;
      case 500: case 502: case 503:
        userMessage = 'El servidor esta teniendo problemas. Intentalo en unos minutos'; break;
      case 401: userMessage = 'Tus credenciales han expirado. Vuelve a iniciar sesion'; break;
      case 403: userMessage = 'No tienes permisos para realizar esta accion'; break;
      case 400: userMessage = 'Los datos enviados no son validos'; break;
      default:
        userMessage = 'Hay un problema con el servicio. Intentalo mas tarde';
    }

    userMessage = "$userMessage (ERR_$idError)";

    _logDioError(e, userMessage, technicalMessage, module, file, line);

    return DataState<T>.dioError(
      error: e,
      userMessage: userMessage,
      technicalMessage: technicalMessage,
      module: module,
      file: file,
      line: line,
      stackTrace: stackTrace,
    );
  }

  static DataState<T> fromException<T>({
    required String message,
    required String module,
    required String file,
    required String line,
    required String stackTrace,
    required String idError,
  }) {
    String userMessage = 'Algo salio mal en la aplicacion. Nuestro equipo fue notificado (ERR_$idError)';

    _logGeneralError(message, userMessage, module, file, line, stackTrace);

    return DataState<T>.generalError(
      userMessage: userMessage,
      technicalMessage: message,
      module: module,
      file: file,
      line: line,
      stackTrace: stackTrace,
    );
  }

  static void _logDioError(DioException error, String userMessage, String techMsg, String module, String file, String line) {
    debugPrint('''
[ERROR DE RED EN $module]
--------------------------------------------------------------------------------
Tipo:       DioException (${error.type})
Codigo:     ${error.response?.statusCode ?? 'N/A'}
URL:        ${error.requestOptions.path}
Archivo:    $file:$line
Usuario:    $userMessage
Tecnico:    $techMsg
--------------------------------------------------------------------------------
    ''');
  }

  static void _logGeneralError(String techMsg, String userMessage, String module, String file, String line, String stack) {
    debugPrint('''
[ERROR GENERAL EN $module]
--------------------------------------------------------------------------------
Archivo:    $file:$line
Usuario:    $userMessage
Tecnico:    $techMsg
Stacktrace: $stack
--------------------------------------------------------------------------------
    ''');
  }
}
```
### Ejemplo de Uso en RepositoryImpl

```dart
try {
  // Logica de peticion...
} on DioException catch (e, stackTrace) {
  return DataStateFactory.fromDioException(e,
      module: 'AuthRepositoryImpl',
      file: 'auth_repository_impl.dart',
      line: '89',
      idError: 'AUTH01',
      stackTrace: stackTrace.toString());
} catch (e, stackTrace) {
  return DataStateFactory.fromException(
      message: e.toString(),
      module: 'AuthRepositoryImpl',
      file: 'auth_repository_impl.dart',
      line: '96',
      idError: 'AUTH02',
      stackTrace: stackTrace.toString());
}
```

## 1.2 Extension de Conveniencia y Propagacion (DataStateConvenience)
Esta extension añade metodos de utilidad para simplificar la lectura de estados en la UI (BLoCs) y permitir la transferencia de errores entre capas sin generar ruido tecnico innecesario.

### Metodos de Verificacion y Acceso
* isSuccess / isError: Verificadores rapidos de estado.
* dataOrNull: Obtiene el valor de exito o null.
* userMessageOrNull: Extrae el mensaje para el usuario si existe un error.

### Propagacion Quirurgica (propagateError)
El metodo propagateError<R>() permite re-mapear un error de un tipo a otro (ej. de DataState<UserEntity> a DataState<HomeEntity>). 
REGLA DE ORO: Solo debe usarse cuando el estado es un error. Si se llama en un exito, lanzara un StateError.

```dart
extension DataStateConvenience<T> on DataState<T> {
  bool get isSuccess => this is DataSuccess<T>;
  bool get isError => this is DataDioError<T> || this is DataGeneralError<T>;
  
  T? get dataOrNull => isSuccess ? (this as DataSuccess<T>).data : null;

  String? get userMessageOrNull => when(
        success: (_) => null,
        dioError: (_, userMsg, __, ___, ____, _____, ______) => userMsg,
        generalError: (userMsg, _, __, ___, ____, _____) => userMsg,
      );

  DataState<R> propagateError<R>() {
    return when(
      success: (_) => throw StateError('propagateError() llamado en un exito.'),
      dioError: (error, userMessage, technicalMessage, module, file, line, stackTrace) =>
          DataState.dioError(
        error: error,
        userMessage: userMessage,
        technicalMessage: technicalMessage,
        module: module,
        file: file,
        line: line,
        stackTrace: stackTrace,
      ),
      generalError: (userMessage, technicalMessage, module, file, line, stackTrace) =>
          DataState.generalError(
        userMessage: userMessage,
        technicalMessage: technicalMessage,
        module: module,
        file: file,
        line: line,
        stackTrace: stackTrace,
      ),
    );
  }
}
```

### Ejemplo de Uso: UseCase Orquestador
Este patron es obligatorio cuando un UseCase depende de otros (Catalogs, Sync, Dashboards). Evita duplicar logs ya que el error original ya fue registrado por la DataStateFactory en el repositorio.

```dart
class GetAllCatalogsUseCase extends UseCaseWithoutParams<AllCatalogsEntity> {
  final GetStatesUseCase _getStatesUseCase;
  final GetReasonsUseCase _getReasonsUseCase;

  const GetAllCatalogsUseCase(this._getStatesUseCase, this._getReasonsUseCase);

  @override
  DataResult<AllCatalogsEntity> call() async {
    // 1. Ejecutar sub-proceso
    final statesResult = await _getStatesUseCase.call();

    // 2. Validar y propagar si hay error
    if (statesResult.isError) {
      return statesResult.propagateError<AllCatalogsEntity>();
    }

    // 3. Continuar si es exitoso
    final states = statesResult.dataOrNull!;
    
    // ... repetir para otros procesos y combinar
    return DataState.success(AllCatalogsEntity(states: states));
  }
}
```

## 2. Los 4 Tipos de UseCase
Para mantener la consistencia en la capa de Domain, se deben implementar exclusivamente estos 4 contratos abstractos:

| Tipo | Definicion | Uso sugerido |
|---|---|---|
| UseCaseWithParams<T, P> | Recibe parametros, retorna un valor | GetUserById, LoginUser |
| UseCaseWithoutParams<T> | Sin parametros, retorna un valor | GetCurrentUser, GetDashboard |
| UseCaseVoid<P> | Recibe parametros, retorna void | LogoutUser, DeleteItem |
| UseCaseVoidWithoutParams | Sin parametros ni retorno | ClearCache, SyncData |

### 2.1 Implementacion Base
Todos los UseCases deben seguir estas firmas de contrato para asegurar que el retorno sea siempre un DataResult:

```dart
abstract class UseCaseWithParams<Type, Params> {
  const UseCaseWithParams();
  DataResult<Type> call(Params params);
}

abstract class UseCaseVoid<Params> {
  const UseCaseVoid();
  DataResultVoid call(Params params);
}

abstract class UseCaseWithoutParams<Type> {
  const UseCaseWithoutParams();
  DataResult<Type> call();
}

abstract class UseCaseVoidWithoutParams {
  const UseCaseVoidWithoutParams();
  DataResultVoid call();
}
```

## 3. Typedefs Globales (DataResult)
Estandarizacion de las firmas de metodos en Repositorios y UseCases para asegurar un manejo de estados asincrono y consistente.

```dart
typedef DataResult<T> = Future<DataState<T>>;
typedef DataResultVoid = Future<DataState<VoidSuccess>>;

/// Clase especial para representar un exito sin datos de retorno
class VoidSuccess { const VoidSuccess(); }
```

## Reglas de Verificacion para el Auditor
Criterios de aceptacion obligatorios que el Agente Auditor debe validar antes de dar por finalizada cualquier tarea de codificacion:

1. Generacion de Codigo: Si el archivo utiliza @freezed pero no incluye la linea "part 'nombre_archivo.freezed.dart'", se debe marcar como ERROR CRITICO.
2. Contratos de Domain: Todo UseCase nuevo debe extender obligatoriamente de uno de los 4 contratos definidos en este Skill (UseCaseWithParams, UseCaseWithoutParams, UseCaseVoid, UseCaseVoidWithoutParams).
3. Trazabilidad de Errores: Todo estado de error (dioError o generalError) debe capturar y enviar los parametros module, file, line y stackTrace. No se permite omitir el origen tecnico del fallo.
4. Naming Convention: Se debe respetar el uso de PascalCase para clases y snake_case para los archivos de la parte generada.