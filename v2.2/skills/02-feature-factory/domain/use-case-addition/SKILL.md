---
name: use-case-addition
description: Catalogo de implementacion de Casos de Uso. Define los 4 patrones base y la logica de orquestacion compleja.
---

# Skill: Use Case Addition - Patrones de Implementacion

Este manual define como dar vida a las reglas de negocio. Los Use Cases son los unicos autorizados para dictar el flujo de datos entre los Repositorios y la Interfaz.

## 1. Los 4 Patrones Fundamentales
Todo Caso de Uso debe extender obligatoriamente uno de estos 4 contratos para asegurar que el retorno sea siempre un `DataState` consistente y tipado.

### 1.1 UseCaseWithParams<T, P> (Con parametros y retorno)
Se utiliza para operaciones que requieren datos de entrada (filtros, credenciales, IDs) y devuelven una entidad.

```dart
// Ejemplo: Autenticacion de usuario con parametros dedicados
class LoginUseCase extends UseCaseWithParams<UserEntity, LoginParams> {
  final AuthRepository _authRepository;

  const LoginUseCase(this._authRepository);

  @override
  DataResult<UserEntity> call(LoginParams params) async {
    return await _authRepository.login(
      username: params.username,
      password: params.password,
      rememberMe: params.rememberMe,
    );
  }
}

class LoginParams {
  final String username;
  final String password;
  final bool rememberMe;

  const LoginParams({
    required this.rememberMe,
    required this.username,
    required this.password,
  });
}
```
### 1.2 UseCaseWithoutParams<T> (Sin parametros, con retorno)
Ideal para obtener informacion del estado actual o configuraciones que no requieren filtros adicionales.

```dart
// Ejemplo: Obtener perfil del usuario autenticado
class GetCurrentUserUseCase extends UseCaseWithoutParams<UserEntity> {
  final AuthRepository _authRepository;

  const GetCurrentUserUseCase(this._authRepository);

  @override
  DataResult<UserEntity> call() async {
    return await _authRepository.getProfile();
  }
}
```

### UseCaseVoid<P> (Con parametros, sin retorno de datos)
Se usa para acciones que modifican el estado (Update/Delete) donde solo nos interesa saber si la operacion fue exitosa.

```dart
// Ejemplo: Actualizar el stock de un producto especifico
class UpdateStockUseCase extends UseCaseVoid<UpdateStockParams> {
  final InventoryRepository _repository;

  const UpdateStockUseCase(this._repository);

  @override
  DataResultVoid call(UpdateStockParams params) async {
    return await _repository.updateQuantity(
      id: params.productId,
      newQuantity: params.quantity,
    );
  }
}

class UpdateStockParams {
  final String productId;
  final int quantity;
  const UpdateStockParams({required this.productId, required this.quantity});
}
```

### UseCaseVoidWithoutParams (Sin parametros ni retorno)
Utilizado para acciones globales de limpieza o procesos internos que no requieren entrada ni salida.

```dart
// Ejemplo: Cerrar sesion de forma global
class LogoutUseCase extends UseCaseVoidWithoutParams {
  final AuthRepository _authRepository;

  const LogoutUseCase(this._authRepository);

  @override
  DataResultVoid call() async {
    return await _authRepository.logout();
  }
}
```

## 2. El Patron Orquestador (Logica Compleja)
Un Caso de Uso Orquestador es aquel que consume multiples sub-procesos (otros Use Cases) para consolidar una respuesta unica. Su mision es asegurar la integridad del flujo y la trazabilidad de errores.

### 2.1 Regla de Propagacion Quirurgica
Es OBLIGATORIO validar el estado de cada sub-proceso inmediatamente despues de su ejecucion. Se debe usar el metodo `.propagateError<T>()` para retornar el error original hacia el BLoC, preservando el ID de error y la trazabilidad original del Repositorio.

### 2.2 Ejemplo: GetAllCatalogsUseCase (Orquestador Masivo)
Este ejemplo ilustra como combinar multiples fuentes de datos en una sola Entidad consolidada de forma segura.

```dart
class GetAllCatalogsUseCase extends UseCaseWithoutParams<AllCatalogsEntity> {
  final GetStatesUseCase _getStatesUseCase;
  final GetCatalogStatusUseCase _getCatalogStatusUseCase;
  final GetReasonsUseCase _getReasonsUseCase;

  const GetAllCatalogsUseCase(
    this._getStatesUseCase,
    this._getCatalogStatusUseCase,
    this._getReasonsUseCase,
  );

  @override
  DataResult<AllCatalogsEntity> call() async {
    // 1. Ejecutar y validar Estados
    final statesResult = await _getStatesUseCase.call();
    if (statesResult.isError) return statesResult.propagateError<AllCatalogsEntity>();
    final states = statesResult.dataOrNull!;

    // 2. Ejecutar y validar Estatus de Catalogo
    final statusResult = await _getCatalogStatusUseCase.call();
    if (statusResult.isError) return statusResult.propagateError<AllCatalogsEntity>();
    final catalogStatus = statusResult.dataOrNull!;

    // 3. Ejecutar y validar Motivos
    final reasonsResult = await _getReasonsUseCase.call();
    if (reasonsResult.isError) return reasonsResult.propagateError<AllCatalogsEntity>();
    final reasons = reasonsResult.dataOrNull!;

    // 4. Consolidar exito final
    return DataState.success(AllCatalogsEntity(
      states: states,
      catalogStatus: catalogStatus,
      reasons: reasons,
    ));
  }
}
```

## Reglas de Verificacion para el Auditor (UseCase Edition)
El Auditor debe rechazar cualquier Caso de Uso que viole estos principios de pureza y flujo:

1. **Sin Logica de Infraestructura**: No se permite el uso de `try-catch`. Si hay un error, debe venir del Repositorio ya mapeado en un `DataState`.
2. **Uso de DataOrNull**: Queda prohibido acceder a `.data` de un resultado sin haber verificado previamente `.isError`. El uso de `dataOrNull!` solo se permite tras la validacion de error.
3. **Exclusividad de DataState**: Verificar que todos los metodos `call` retornen estrictamente `DataResult<T>` o `DataResultVoid`. El Auditor debe marcar como ERROR FATAL cualquier intento de usar `Future<Either>`, `Future<T>` o `Result<T>`. Solo se permite el `DataState` personalizado de la arquitectura.
4. **Independencia de UI**: El Auditor debe confirmar que el archivo NO tenga imports de Flutter (material/cupertino). Solo se permiten imports de `core/` y carpetas de `domain/`.