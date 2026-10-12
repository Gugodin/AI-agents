---
name: network-standards
description: Estandares de comunicacion para APIs (Retrofit/ResponseModel) y Servicios externos (Firebase/Packages)
---

# Skill: Network Standards - Comunicacion y Servicios de Datos

Este documento establece como se deben gestionar las peticiones de datos, diferenciando entre APIs REST estandarizadas y servicios basados en paquetes de Flutter.

## 1. Clasificacion de DataSources
Existen dos formas obligatorias de implementar un DataSource dependiendo del origen de los datos:

### 1.1 REST DataSources (Retrofit)
Se utiliza para APIs que siguen el protocolo HTTP. Es OBLIGATORIO usar Retrofit para estos casos.
* **Con ResponseModel**: Si la API es interna o sigue el estandar de la organizacion, el metodo de Retrofit debe retornar un `Future<ResponseModel>` o `Future<Map<String, dynamic>>` para ser procesado por el wrapper.
* **Sin ResponseModel**: Si es una API externa con una estructura unica (si se utilizara GoogleMapsAPI por ejemplo), se retorna directamente el `Future<GoogleMapsAPIModel>`.

### 1.2 Service DataSources (Firebase / Paquetes)
Se utiliza cuando la fuente de datos es un SDK o paquete (ej: Firebase Firestore, Local Auth, Bluetooth).
* **Implementacion**: Se debe crear una clase (generalmente un Singleton registrado en GetIt) que encapsule el cliente del SDK.
* **Estructura**: Las peticiones deben estar divididas por metodos claros que representen la accion (ej: `getUserStream`, `saveDocument`).
* **Manejo de Errores**: Aunque no usen DioException, deben ser mapeados a `DataState.generalError` para mantener la consistencia en el Repositorio.

## 2. El Estandar ResponseModel
Para APIs conocidas, se utiliza el `ResponseModel` como un wrapper que unifica las diferentes llaves de respuesta (data, status, mensaje) en un solo objeto predecible.

### 2.1 Definicion de la Clase
```dart
class ResponseModel {
  final bool error;
  final String message;
  final dynamic data;
  final String code;
  final String? description;

  ResponseModel({
    required this.error,
    required this.message,
    required this.data,
    required this.code,
    required this.description,
  });

  factory ResponseModel.fromJson(Map<String, dynamic> json) {
    bool isError = json['error'] ?? json['description_status'] != 'SUCCESS';
    if (json['status'] != null) {
      isError = json['status'] != 1;
    }

    return ResponseModel(
      error: isError,
      message: json['message'] ?? json['description_status'] ?? json['mensaje'] ?? '',
      data: json['data_info'] ?? json['catalogo'] ?? json['data'] ?? json['result'],
      code: json['code'] ?? json['code_status']?.toString() ?? '',
      description: json['description'] ?? json['description_status'],
    );
  }
}
```

## 3. DataStateFactory - El Corazon de la Consistencia
El DataStateFactory es una clase o conjunto de metodos estaticos encargados de transformar las excepciones tecnicas y las respuestas de la API en estados de DataState estandarizados.

### 3.1 Mapeo de DioException
Se debe implementar un mapeo de codigos de error HTTP a mensajes legibles para el usuario:
* 400: "Los datos enviados no son validos".
* 401: "Tus credenciales han expirado".
* 403: "No tienes permisos para realizar esta accion".
* 404: "No pudimos encontrar lo que buscas".
* 500, 502, 503: "El servidor esta teniendo problemas, intenta mas tarde".
* Default/Timeout: "La conexion tardo demasiado o es inestable".

## 4. Implementacion en RepositoryImpl
El Repositorio es el responsable de decidir si una respuesta exitosa a nivel de red (HTTP 200) es realmente un exito de negocio usando el ResponseModel.

### 4.1 Flujo de Trabajo con ResponseModel
1. El Repositorio llama al DataSource.
2. El DataSource retorna un ResponseModel.
3. El Repositorio evalua la propiedad 'error' del ResponseModel.
4. Si 'error' es false, se transforma el 'data' a una Entidad y se retorna DataState.success.
5. Si 'error' es true, se extrae el 'message' y se retorna DataState.generalError.

### 4.2 Ejemplo de RepositoryImpl (Implementacion con Referencia Legal)
Para la implementacion de este metodo, es OBLIGATORIO seguir el contrato de la DataStateFactory definido en: 
> Ver: [00-core-rules/blueprint-foundation/SKILL.md > Punto 1.1]

```dart
@override
DataResult<UserEntity> login(LoginParams params) async {
  try {
    final response = await _remoteDataSource.login(params.username, params.password);

    // Validacion de negocio (Ver logica en network-standards > Punto 4.1)
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
    // Uso obligatorio de la fabrica (Ver blueprint-foundation > Punto 1.1)
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

## 5. Reglas de Verificacion para el Auditor (Network Edition)
Criterios obligatorios que el Agente Auditor debe validar para garantizar la calidad del manejo de datos, red y trazabilidad de fallos.

1. Uso de Retrofit: Si el DataSource consume una API REST, debe usar obligatoriamente @RestApi y la linea "part 'nombre_archivo.g.dart'". De lo contrario, se marca como ERROR.

2. Logica de ResponseModel: En Repositorios que consumen APIs conocidas, el Auditor debe verificar que se valide la propiedad 'error' del ResponseModel. Solo si 'error' es false se puede proceder al mapeo de la entidad y retornar DataState.success.

3. Trazabilidad con ID Unico: Es OBLIGATORIO que cada llamada a DataStateFactory (fromDioException o fromException) incluya un parametro 'idError' unico y descriptivo (ej: 'AUTH01', 'AUTH02'). El Auditor debe rechazar cualquier implementacion que omita este ID o que lo repita en diferentes metodos del mismo Repositorio.

4. Mensajes al Usuario: El Auditor debe confirmar que no se esten "quemando" (hardcoding) mensajes de error genericos dentro del try-catch si el ResponseModel ya provee un mensaje. Se debe priorizar el mensaje de la API y asegurar que el ID de error sea visible para el usuario final.

5. DataSources de Servicio: Para integraciones con SDKs (ej: Firebase, Bluetooth), el Auditor debe confirmar que no se use Retrofit y que la logica de peticiones este encapsulada en una clase de servicio debidamente registrada en el inyector de dependencias.

6. Prevencion de Logs Duplicados: El Auditor debe verificar que en UseCases orquestadores se utilice 'propagateError()' en lugar de volver a llamar a la DataStateFactory. Esto asegura que el log del error se genere una sola vez en el origen (Repositorio).