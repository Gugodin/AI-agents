---
name: use-case-standard
description: Reglas de oro para la implementacion de Logica de Negocio y Orquestacion de servicios
---

# Skill: Use Case Standard - Logica de Negocio

Este documento establece los contratos obligatorios para la capa de Domain. Los UseCases son los unicos responsables de orquestar repositorios y otros UseCases para cumplir con una accion de negocio.

## 1. Los 4 Tipos de UseCase
Para mantener la consistencia, se deben implementar exclusivamente estos 4 contratos abstractos segun la necesidad de parametros y retorno:

| Tipo | Definicion | Uso sugerido |
|---|---|---|
| UseCaseWithParams<T, P> | Recibe parametros, retorna un valor | GetUserById, LoginUser |
| UseCaseWithoutParams<T> | Sin parametros, retorna un valor | GetCurrentUser, GetDashboard |
| UseCaseVoid<P> | Recibe parametros, retorna void | LogoutUser, DeleteItem |
| UseCaseVoidWithoutParams | Sin parametros ni retorno | ClearCache, SyncData |

## 1.1 Implementacion Base
Todos los UseCases deben seguir estas firmas para asegurar que el retorno sea siempre un DataResult (typedef estandar):

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

## 2. Orquestacion y Propagacion de Errores
Los UseCases orquestadores son aquellos que consumen multiples sub-procesos (otros UseCases) para consolidar informacion en una sola entidad de respuesta.

### Regla de Propagacion (propagateError)
Es OBLIGATORIO utilizar el metodo `propagateError<R>()` cuando un sub-proceso falla. Esto garantiza que:
* No se dupliquen logs de error en consola, ya que el log original se genero en el Repositorio.
* Se preserve la trazabilidad original (module, file, line, idError) para el equipo de soporte.
* El flujo de ejecucion se detenga inmediatamente ante el primer fallo detectado.

### Ejemplo de Uso: GetAllCatalogsUseCase
```dart
class GetAllCatalogsUseCase extends UseCaseWithoutParams<AllCatalogsEntity> {
  final GetStatesUseCase _getStatesUseCase;
  final GetReasonsUseCase _getReasonsUseCase;

  const GetAllCatalogsUseCase(this._getStatesUseCase, this._getReasonsUseCase);

  @override
  DataResult<AllCatalogsEntity> call() async {
    // 1. Ejecutar primer sub-proceso
    final statesResult = await _getStatesUseCase.call();

    // 2. Validar y propagar si hay error
    if (statesResult.isError) {
      return statesResult.propagateError<AllCatalogsEntity>();
    }

    // 3. Extraer datos con seguridad
    final states = statesResult.dataOrNull!;
    
    // ... Logica continua para otros procesos
    return DataState.success(AllCatalogsEntity(states: states));
  }
}
```

## 3. Reglas de Verificacion para el Auditor (UseCase Edition)

1. Contratos de Domain: Todo UseCase nuevo debe extender obligatoriamente de uno de los 4 contratos base (WithParams, WithoutParams, Void, VoidWithoutParams).
2. Manejo de Errores: Se prohibe el uso de bloques try-catch dentro de un UseCase para capturar errores de Repositorios u otros UseCases. Se debe usar estrictamente isError y propagateError().
3. Pureza de Domain: El Auditor debe confirmar que el UseCase no tenga dependencias de paquetes de Flutter (como material.dart) ni de modelos de datos (Models); solo debe conocer Entidades y Repositorios.