---
name: data-source-addition
description: Reglas para la creacion de DataSources remotos utilizando Retrofit y el modelo de respuesta estandarizado.
---

# Skill: Data Source Addition - El Sensor de Datos

Este manual define como implementar la comunicacion con APIs externas. El DataSource es una interfaz tecnica que utiliza Retrofit para mapear endpoints de red a funciones de Dart.

## 1. Definicion del Remote DataSource (Retrofit)
Para asegurar un codigo limpio y facil de mantener, se utiliza el generador de codigo **Retrofit**. Esto evita escribir codigo repetitivo de Dio y centraliza la definicion de los endpoints.

### 1.1 Estructura y Anotaciones
Cada DataSource debe ser una clase abstracta anotada con `@RestApi()`. Se deben utilizar las anotaciones estandar de Retrofit (`@GET`, `@POST`, `@PUT`, `@DELETE`, `@Body`, `@Path`, etc.) para definir la interaccion con la API.

### 1.2 Ubicacion y Nomenclatura
* **Directorio**: `lib/features/[feature_name]/data/data_sources/`.
* **Archivo**: `[nombre]_data_source.dart` (ej: `auth_data_source.dart`).
* **Clase**: `[Nombre]DataSource` (ej: `AuthDataSource`).

### 1.3 Integracion con el ResponseModel
Es OBLIGATORIO que todos los metodos del DataSource retornen un `Future<ResponseModel<dynamic>>`. Esta estructura asegura que, sin importar el endpoint, el Repositorio siempre reciba un contenedor predecible con los campos `error`, `message` y `data`.

### 1.4 Ejemplo de Implementacion Base
```dart
import 'package:dio/dio.dart';
import 'package:retrofit/retrofit.dart';
import '../../../../core/network/response_model.dart';

part 'auth_data_source.g.dart';

@RestApi()
abstract class AuthDataSource {
  factory AuthDataSource(Dio dio, {String baseUrl}) = _AuthDataSource;

  @POST('/auth/login')
  Future<ResponseModel<dynamic>> login({
    @Field("username") required String username,
    @Field("password") required String password,
    @Field("latitud") required String latitud,
    @Field("longitud") required String longitud,
    @Field("so") required String so,
  });

  @GET('/auth/logout')
  Future<ResponseModel<dynamic>> logout();
}
```

## 2. DataSources de Proveedores Externos (SDKs/Local)
Existen casos donde la fuente de datos no es una API REST (ej: Firebase, SQLite, Bluetooth). En estos escenarios, no se utiliza Retrofit, sino que se crea una clase manual que encapsula el SDK del proveedor.

### 2.1 Consistencia de Interfaz
Para que el Repositorio sea agnostico a la fuente, estos DataSources manuales DEBEN retornar tambien un `Future<ResponseModel>`. Nosotros mismos construiremos el objeto `ResponseModel` basandonos en lo que el SDK nos devuelva.

### 2.2 Patron Singleton y Gestion
Aunque la clase se defina de forma estandar, su ciclo de vida como **Singleton** se gestionara preferiblemente a traves del `injector/` del proyecto para asegurar una unica instancia del cliente (ej: Firestore) en toda la app.

### 2.3 Ejemplo de Implementacion (Firebase Firestore)
Este ejemplo muestra como transformar una peticion de un SDK en un `ResponseModel` compatible con nuestra arquitectura.

```dart
import 'package:cloud_firestore/cloud_firestore.dart';
import '../../../../core/network/response_model.dart';

class UserRemoteDataSource {
  final FirebaseFirestore _firestore;

  const UserRemoteDataSource(this._firestore);

  /// Obtiene todos los usuarios desde Firestore y los envuelve en un ResponseModel
  Future<ResponseModel> getAllUsers() async {
    try {
      final querySnapshot = await _firestore.collection('users').get();
      
      // Transformamos los documentos en una lista de Mapas
      final data = querySnapshot.docs.map((doc) => doc.data()).toList();

      // Retornamos el formato estandarizado
      return ResponseModel(
        error: false,
        message: 'Usuarios obtenidos con exito',
        data: data,
      );
    } catch (e) {
      // Si el SDK falla, lo notificamos a traves del ResponseModel
      return ResponseModel(
        error: true,
        message: 'Error en Firestore: ${e.toString()}',
        data: null,
      );
    }
  }
}
```
### 3 Reglas de Verificacion para el Auditor (DataSource Edition)
1. Tipo de Retorno: El Auditor debe rechazar cualquier metodo en un DataSource que NO retorne un Future<ResponseModel>. La estandarizacion de la respuesta es innegociable.

2. Aislamiento de Errores: En DataSources manuales (Punto 2), el Auditor debe verificar que exista un try-catch interno que asegure que el ResponseModel siempre se entregue con error: true en caso de falla del SDK.

3. Cero Logica de Negocio: El DataSource solo debe entregar la "data cruda" (Mapas, Listas). El Auditor debe marcar como ERROR si detecta que el DataSource intenta instanciar una Entity o un Model.