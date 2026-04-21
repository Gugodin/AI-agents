---
name: data-state-standard
description: Reglas obligatorias para el manejo de estados inmutables, trazabilidad de errores y tipos globales
---

# Skill: Data State Standard - El ADN de los Datos

Este documento define la estructura inmutable de los datos y el manejo de estados de la aplicacion. Es el estandar de oro para cualquier retorno de informacion en la capa de Data y Domain.

## 1. DataState (Inmutabilidad con Freezed)
⚠️ INYECCION OBLIGATORIA PARA SUB-AGENTES: Este bloque debe copiarse integramente si vas a pedir a otro agente que cree o modifique este archivo.

Es OBLIGATORIO el uso de @freezed para el manejo de estados de datos. No se permiten sealed classes nativas fuera de Freezed para garantizar la consistencia en el pattern matching y la inmutabilidad.

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
⚠️ INYECCION OBLIGATORIA PARA SUB-AGENTES: Si vas a delegar la creacion del DataStateFactory, debes pasar esta extension completa al sub-agente.

La DataStateFactory es una extension que automatiza la creacion de estados de error, garantizando que todo fallo sea rastreable y amigable para el usuario final.

### Reglas de Mensajes e Identificadores (IDs)
1. Unicidad: Cada llamada en el repositorio debe asignar un idError unico (ej: 'AUTH01', 'USER05') para identificar el punto exacto del fallo.
2. Formato de Usuario: El mensaje final para el usuario (userMessage) DEBE concatenar el ID al final con el formato: "Mensaje de error (ERR_idError)".
3. Logging Obligatorio: Todo error debe disparar un print en consola (debugPrint) con un formato visual claro que separe la informacion de Red (Dio) de los errores Generales.

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
@override
DataResult<UserEntity> login(LoginParams params) async {
  try {
    final response = await _remoteDataSource.login(params.username, params.password);

    if (response.error) {
      return DataState.generalError(
        userMessage: '${response.message} (ERR_AUTH01)',
        technicalMessage: response.message,
        module: 'AuthRepositoryImpl',
        file: 'auth_repository_impl.dart',
        line: '45',
        stackTrace: 'Business Logic Error',
      );
    }

    return DataState.success(UserModel.fromJson(response.data).toEntity());
    
  } on DioException catch (e, stackTrace) {
    return DataStateFactory.fromDioException(
      e,
      module: 'AuthRepositoryImpl',
      file: 'auth_repository_impl.dart',
      line: '55',
      idError: 'AUTH02',
      stackTrace: stackTrace.toString(),
    );
  } catch (e, stackTrace) {
    return DataStateFactory.fromException(
      message: e.toString(),
      module: 'AuthRepositoryImpl',
      file: 'auth_repository_impl.dart',
      line: '62',
      idError: 'AUTH03',
      stackTrace: stackTrace.toString(),
    );
  }
}
```

## 1.2 Extension de Conveniencia y Propagacion (DataStateConvenience)
⚠️ INYECCION OBLIGATORIA PARA SUB-AGENTES: Si vas a pedir la creacion de este archivo a un sub-agente, debes inyectar este bloque obligatoriamente.

Esta extension añade metodos de utilidad para simplificar la lectura de estados en la UI (BLoCs) y permitir la transferencia de errores entre capas sin generar ruido tecnico innecesario.

### Metodos de Verificacion y Acceso
* isSuccess / isError: Verificadores rapidos de estado.
* dataOrNull: Obtiene el valor de exito o null.
* userMessageOrNull: Extrae el mensaje para el usuario si existe un error.

### Propagacion Quirurgica (propagateError)
El metodo propagateError<R>() permite re-mapear un error de un tipo a otro (ej. de DataState<UserEntity> a DataState<HomeEntity>) preservando la trazabilidad original.

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

## 2. Typedefs Globales (DataResult)
⚠️ INYECCION OBLIGATORIA PARA SUB-AGENTES: Estos Typedefs y el VoidSuccess son inmutables. Pasalos al sub-agente explicitamente.

Estandarizacion de las firmas de metodos en Repositorios y UseCases para asegurar un manejo de estados asincrono y consistente.

```dart
typedef DataResult<T> = Future<DataState<T>>;
typedef DataResultVoid = Future<DataState<VoidSuccess>>;

/// Clase especial para representar un exito sin datos de retorno
class VoidSuccess { const VoidSuccess(); }
```

## 3. Reglas de Verificacion para el Auditor (Data State Edition)

1. Generacion de Codigo: El Auditor debe verificar que todo archivo de DataState incluya la linea "part 'nombre_archivo.freezed.dart'".
2. Trazabilidad: Todo estado de error (dioError o generalError) debe capturar y enviar obligatoriamente module, file, line, stackTrace e idError.
3. IDs de Error: El Auditor debe rechazar cualquier Repository que use IDs de error genericos o repetidos.