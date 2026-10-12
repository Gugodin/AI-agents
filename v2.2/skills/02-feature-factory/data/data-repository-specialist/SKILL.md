---
name: data-repository-specialist
description: Reglas para la implementacion concreta de Repositorios (RepositoryImpl) en la capa de Data.
---

# Skill: Data Repository Specialist - La Implementacion de la Infraestructura

## 1. Implementacion de Repositorios (RepositoryImpl)
El `RepositoryImpl` es el responsable de ejecutar la logica de infraestructura. Debe coordinar multiples dependencias para cumplir con los contratos del dominio, manejando errores de forma explicita y granular.

### 1.1 Inyeccion de Dependencias Total (Constructor)
Para garantizar el desacoplamiento y facilitar las pruebas unitarias, queda ESTRICTAMENTE PROHIBIDO instanciar helpers o servicios dentro del repositorio. Todas las dependencias deben ser inyectadas por constructor:
* **DataSources**: Interfaces de Retrofit/Local.
* **Helpers**: LocationHelper, BiometricsHelper, etc.
* **Persistencia**: SharedPreferenceHelper, SecureStorageHelper.

### 1.2 Estructura de Metodos (Try-Catch)
Cada operacion debe estar envuelta en un bloque `try-catch` para asegurar que ningun error inesperado rompa el flujo de la app. El manejo de estados de error debe delegarse a la factoria global:

1. **Bloque Try**: Ejecuta la logica (obtener coordenadas, llamar al DataSource, guardar en local).
2. **On DioException**: Captura errores de red y los mapea usando `DataStateFactory.fromDioException`. Para entender la estructura del objeto de error resultante, consultar la **Seccion 1.1 del data-state-standard/SKILL.md**.
3. **Catch General**: Captura cualquier otra excepcion y la mapea con `DataStateFactory.fromException`.

### 1.3 Validacion de ResponseModel
Al recibir la respuesta del `DataSource`, se debe validar la propiedad `.error` del `ResponseModel`. Este modelo de respuesta sigue el estandar de API definido en la **Seccion 2.1 del network-standards/SKILL.md**, asegurando que la data cruda este envuelta en una estructura predecible.

### 1.4 Ejemplo de Implementacion Estandar
```dart
class AuthRepositoryImpl implements AuthRepository {
  final AuthDataSource _remoteDataSource;
  final SharedPreferenceHelper _sharedPreferenceHelper;

  const AuthRepositoryImpl({
    required AuthDataSource remoteDataSource,
    required SharedPreferenceHelper sharedPreferenceHelper,
  })  : _remoteDataSource = remoteDataSource,
        _sharedPreferenceHelper = sharedPreferenceHelper;

  @override
  DataResult<UserEntity> login({required String user, required String pass}) async {
    try {
      final response = await _remoteDataSource.login(user, pass);

      if (response.error) {
        return DataState.generalError(
          technicalMessage: response.message,
          userMessage: 'Credenciales invalidas',
          // ... mapeo de campos de rastreo segun data-state-standard 1.1
        );
      }

      await _sharedPreferenceHelper.setUser(UserModel.fromJson(response.data));
      return DataState.success(UserModel.fromJson(response.data));

    } on DioException catch (e, stackTrace) {
      // Uso de DataStateFactory segun data-state-standard 1.1
      return DataStateFactory.fromDioException(e, module: 'AuthRepo', idError: 'AUTH01', stackTrace: stackTrace.toString());
    } catch (e, stackTrace) {
      return DataStateFactory.fromException(message: e.toString(), module: 'AuthRepo', idError: 'AUTH02', stackTrace: stackTrace.toString());
    }
  }
}
```

## 2. Reglas de Verificacion para el Auditor (Data Repository Edition)

### 2.1 Calidad del Repositorio
1. **Inyeccion Obligatoria**: Es un ERROR FATAL si el repositorio instancia un Helper o DataSource internamente (ej: `final h = LocationHelper();`). Todas las dependencias deben ser `final` y recibidas por constructor.
2. **Cobertura de Try-Catch**: El Auditor debe verificar que TODOS los metodos del repositorio esten envueltos en un bloque `try-catch`.
3. **Uso de Factorias**: Confirmar que los errores sean mapeados exclusivamente a traves de `DataStateFactory.fromDioException` o `fromException`. No se permiten retornos manuales de `DataState.error` sin pasar por la factoria.

### 2.2 Protocolo de Red
1. **Validacion de ResponseModel**: El Auditor debe asegurar que el codigo verifique siempre el campo `response.error`. Si no se gestiona el error proveniente de la API antes de intentar mapear la data, la revision falla.
2. **Trazabilidad (IDs de Error)**: Cada mapeo de error en el `catch` debe contar con un `idError` unico y descriptivo (ej: `AUTH01`, `INV02`) para facilitar el debugging en produccion.
