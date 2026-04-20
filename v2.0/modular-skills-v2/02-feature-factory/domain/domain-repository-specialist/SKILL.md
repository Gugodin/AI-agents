---
name: domain-repository-specialist
description: Reglas para la creacion de Contratos de Repositorio (interfaces abstractas) en la capa de Domain.
---

# Skill: Domain Repository Specialist - Los Contratos del Negocio

## 1. Contratos de Repositorio (Interfaces)
El repositorio en el dominio es una clase abstracta que define **que** puede hacer la feature, sin preocuparse por el **como** se implementa (si es via REST, GraphQL o base de datos local).

### 1.1 Estandarizacion de Retornos (DataResult)
Para mantener la consistencia con el sistema de estados y errores de la arquitectura Segi, es OBLIGATORIO que todos los metodos utilicen los `typedefs` globales:

* **DataResult<T>**: Se usa para operaciones asincronas que devuelven una Entidad o lista de ellas.
* **DataResultVoid**: Se usa para acciones que no devuelven datos, como eliminar un registro o cerrar sesion.

### 1.2 Ubicacion y Nomenclatura
* **Directorio**: `lib/features/[feature_name]/domain/repositories/`.
* **Nombre de Archivo**: `[nombre]_repository.dart` (ej: `auth_repository.dart`).
* **Nombre de Clase**: `[Nombre]Repository` (ej: `AuthRepository`).

### 1.3 Ejemplo de Implementacion
```dart
import '../../../../core/foundation/typedefs.dart';
import '../entities/user_entity.dart';

abstract class AuthRepository {
  /// Realiza el inicio de sesion y retorna la entidad del usuario
  DataResult<UserEntity> login({
    required String username,
    required String password,
  });

  /// Cierra la sesion activa en el dispositivo
  DataResultVoid logout();
}
```

### 2. Reglas de Verificacion para el Auditor (Repository Edition)
El Auditor debe garantizar que el repositorio sea una interfaz pura y que cumpla estrictamente con los estándares de retorno de la arquitectura.

## 2.1 Pureza de la Interfaz

Prohibición de Implementación: El Auditor debe rechazar cualquier repositorio que contenga cuerpos de métodos (bloques { ... }). Todo debe ser estrictamente abstract.
Cero Lógica Técnica: Si el contrato incluye parámetros que revelen la implementación (ej: un Map<String, dynamic> como parámetro o un DioException en la firma), la auditoría falla.

## 2.2 Blindaje de Tipos (Typedefs)

Uso Obligatorio de DataResult: El Auditor debe verificar que TODOS los métodos asíncronos retornen DataResult<T> o DataResultVoid. Queda estrictamente prohibido el uso de Future<Entity>, Future<Either> o Future<T>.

Importación de Foundation: Se debe validar que el archivo importe correctamente core/foundation/typedefs.dart para el uso de los tipos de retorno estandarizados.

## 2.3 Control de Dependencias (Entities vs Models)

Regla de Oro de Imports: El Auditor debe revisar la sección de imports. Si encuentra una referencia a un archivo que termine en _model.dart, el contrato es inválido. Un repositorio de dominio solo puede conocer Entidades.

Independencia de DataSources: No se permite que el repositorio dependa de interfaces de DataSources o clientes de red en esta capa.

## 2.4 Documentación y Nomenclatura

Documentación de Intención: Se recomienda que cada método tenga un comentario de documentación (///) que explique el propósito del contrato desde el punto de vista del negocio.

Nomenclatura: El Auditor debe confirmar que la clase sea una abstract class y que el nombre termine exactamente en Repository.