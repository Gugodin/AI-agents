---
name: data-model-specialist
description: Reglas para la creacion de Models (DTOs) y su serializacion en la capa de Data.
---

# Skill: Data Model Specialist - Los Puentes de Datos

## 1. Modelos de Datos (Models / DTOs)
Los modelos son las representaciones tecnicas de los datos que recibimos o enviamos al exterior. Deben ser implementados en **Dart Nativo** para mantener la consistencia con las entidades y evitar la sobrecarga de generadores de codigo.

### 1.1 Extension de Entidades
Para asegurar que la capa de Data sea compatible con el Dominio, se deben seguir estas reglas estructurales:
* **Herencia**: Todo `Model` debe extender (`extends`) a su correspondiente `Entity`.
* **Uso de super**: El constructor del modelo debe pasar los valores al constructor de la entidad utilizando la palabra clave `super`.

### 1.2 Serializacion Nativa (fromJson)
La responsabilidad de transformar el JSON crudo recae exclusivamente en el Modelo mediante metodos manuales:
* **Factory fromJson**: Es obligatorio incluir un constructor factory que reciba un `Map<String, dynamic>`.
* **Defensa contra Nulos**: Se deben proveer valores por defecto o manejar la nulidad durante el mapeo para evitar que la aplicacion truene por errores de parsing.

### 1.3 Ubicacion y Nomenclatura
* **Directorio**: `lib/features/[feature_name]/data/models/`.
* **Nombre de Archivo**: `[nombre]_model.dart` (ej: `user_model.dart`).
* **Nombre de Clase**: `[Nombre]Model` (ej: `UserModel`).

### 1.4 Ejemplo de Implementacion (Dart Nativo)
```dart
import '../../domain/entities/user_entity.dart';

class UserModel extends UserEntity {
  const UserModel({
    required super.id,
    required super.name,
    required super.email,
  });

  factory UserModel.fromJson(Map<String, dynamic> json) {
    return UserModel(
      id: json['id'] as String? ?? '',
      name: json['name'] as String? ?? 'Sin nombre',
      email: json['email'] as String? ?? '',
    );
  }
}
```

### 1.5 Objetos Anidados (Modelos Complejos)
Cuando una Entidad contiene sub-objetos (composicion), la capa de Data debe replicar esta jerarquia siguiendo la regla de extension nativa. Cada sub-modelo debe extender a su correspondiente sub-entidad.

#### Reglas de Composicion:
1. **Jerarquia de Extension**: Si `UserEntity` tiene un `RoleEntity`, entonces `UserModel` debe tener un `RoleModel` que extienda de `RoleEntity`.
2. **Mapeo en Cascada**: El `fromJson` del modelo principal es responsable de invocar el `fromJson` de los modelos hijos, asegurando que toda la estructura se transforme correctamente desde el mapa crudo.

#### Ejemplo de Implementacion Nativa:

**En Domain:**
```dart
// role_entity.dart
class RoleEntity {
  final String name;
  final List<String> permissions;
  const RoleEntity({required this.name, required this.permissions});
}

// user_entity.dart
class UserEntity {
  final String id;
  final RoleEntity role; // Usa la Entidad
  const UserEntity({required this.id, required this.role});
}
```

**En Data:**
```dart
// role_model.dart
class RoleModel extends RoleEntity {
  const RoleModel({required super.name, required super.permissions});

  factory RoleModel.fromJson(Map<String, dynamic> json) {
    return RoleModel(
      name: json['name'] as String? ?? 'guest',
      permissions: List<String>.from(json['permissions'] ?? []),
    );
  }
}

// user_model.dart
class UserModel extends UserEntity {
  const UserModel({required super.id, required RoleModel super.role});

  factory UserModel.fromJson(Map<String, dynamic> json) {
    return UserModel(
      id: json['id'] as String? ?? '',
      role: RoleModel.fromJson(json['role'] as Map<String, dynamic>? ?? {}),
    );
  }
}
```

## 2. Reglas de Verificacion para el Auditor (Data Model Edition)

### 2.1 Integridad de Modelos
1. **Prohibicion de Generadores**: El Auditor debe marcar como ERROR el uso de `@freezed` o `@JsonSerializable` en los modelos. Se exige el uso de **Dart Nativo** con constructores `const` y factories `fromJson` manuales.
2. **Jerarquia de Herencia**: Verificar que cada `Model` extienda (`extends`) a su respectiva `Entity`. El Auditor debe confirmar que el constructor del modelo use `super` para inicializar las propiedades de la entidad.
3. **Anidacion Correcta**: Si una entidad tiene sub-entidades, el modelo correspondiente debe tener sub-modelos que tambien extiendan de dichas sub-entidades (ver Seccion 1.5).

### 2.2 Limpieza de Imports
1. **Cero Imports de UI**: El Auditor debe rechazar archivos en la capa de Data que importen `package:flutter/material.dart` o componentes de la capa de `presentation`.
