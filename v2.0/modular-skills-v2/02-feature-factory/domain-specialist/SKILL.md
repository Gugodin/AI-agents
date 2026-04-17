---
name: domain-specialist
description: Reglas para la creacion de Entidades inmutables y Contratos de Repositorio en la capa de Domain.
---

# Skill: Domain Specialist - El Corazon del Negocio

## 1. Definicion de Entidades (Entities)
Las entidades son los modelos de datos fundamentales que representan la logica de negocio. Al ser el "corazon" del sistema, deben estar escritas en **Dart Nativo** para asegurar que el dominio sea agnostico, ligero y facil de testear.

### 1.1 Inmutabilidad y Estructura
Para mantener la integridad de los datos a lo largo de la aplicacion, se deben seguir estas reglas de construccion:
* **Propiedades Finales**: Todas las variables miembro deben ser marcadas como `final`.
* **Constructor Const**: Se debe utilizar un constructor `const` para permitir la instanciacion eficiente y asegurar que el objeto no cambie tras su creacion.
* **Metodos Opcionales**: Se permite la creacion manual de metodos como `copyWith` o `toString` si la funcionalidad de la feature lo requiere, evitando siempre el uso de generadores de codigo externos en esta capa.

### 1.2 Ubicacion y Nomenclatura
* **Directorio**: `lib/features/[feature_name]/domain/entities/`.
* **Nombre de Archivo**: `[nombre]_entity.dart` (ej: `product_entity.dart`).
* **Nombre de Clase**: `[Nombre]Entity` (ej: `ProductEntity`).

### 1.3 Reglas de Pureza
* **Cero Dependencias**: No se permite importar `package:flutter/material.dart`, `package:dio` ni ninguna libreria externa de infraestructura. A menos que sea estrictamente necesario para la logica de negocio, se deben evitar incluso las dependencias internas del proyecto. Como por ejemplo,
si una entidad debe de guardar la latitud y longitud de un lugar, se pueden usar `double` en lugar de una clase `Location` que dependa de otra capa.
* **Sin Serializacion**: Las entidades NO deben contener metodos `fromJson` o `toJson`. Esa responsabilidad pertenece a los `Models` en la capa de Data.

### 1.4 Ejemplo de Implementacion
```dart
class UserEntity {
  final String id;
  final String name;
  final String email;

  const UserEntity({
    required this.id,
    required this.name,
    required this.email,
  });
}
 ```

## 2. Contratos de Repositorio (Interfaces)
El repositorio en el dominio es una clase abstracta que define **qué** puede hacer la feature, sin preocuparse por el **cómo** se implementa (si es via REST, GraphQL o base de datos local).

### 2.1 Estandarizacion de Retornos (DataResult)
Para mantener la consistencia con el sistema de estados y errores de la arquitectura Segi, es OBLIGATORIO que todos los metodos utilicen los `typedefs` globales:

* **DataResult<T>**: Se usa para operaciones asincronas que devuelven una Entidad o lista de ellas.
* **DataResultVoid**: Se usa para acciones que no devuelven datos, como eliminar un registro o cerrar sesion.

### 2.2 Ubicacion y Nomenclatura
* **Directorio**: `lib/features/[feature_name]/domain/repositories/`.
* **Nombre de Archivo**: `[nombre]_repository.dart` (ej: `auth_repository.dart`).
* **Nombre de Clase**: `[Nombre]Repository` (ej: `AuthRepository`).

### 2.3 Ejemplo de Implementacion
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