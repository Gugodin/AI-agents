---
name: domain-entity-specialist
description: Reglas para la creacion de Entidades inmutables en la capa de Domain.
---

# Skill: Domain Entity Specialist - El Corazon del Negocio

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

### 2. Reglas de Verificacion para el Auditor (Entity Edition)
El Auditor debe rechazar cualquier entidad que presente "fugas" de otras capas o que rompa la inmutabilidad del dominio.

## 2.1 Pureza de Tipos y Primitivos
Regla del Primitivo: El Auditor debe verificar que se prefieran tipos primitivos de Dart (double, int, String, bool) para representar datos de soporte (como coordenadas o fechas en int o String) en lugar de clases de librerías externas o de la capa de infraestructura.
Prohibición de Frameworks: Cualquier importación de package:flutter/... es motivo de RECHAZO inmediato. El dominio debe poder ejecutarse en un entorno puro de Dart.

## 2.2 Integridad Estructural
Inmutabilidad Estricta: Se debe comprobar que TODAS las propiedades sean final. Si existe una sola variable sin final, la entidad no es válida.
Constructor Constante: La ausencia de la palabra clave const en el constructor principal se considera un error de optimización y seguridad de datos.

## 2.3 Blindaje contra Serialización
Cero JSON: El Auditor debe buscar palabras clave como fromJson, toJson, Map<String, dynamic> o cualquier mención a serialización. Si se encuentran, deben ser movidas a un Model en la capa de Data.
Sin Generadores: No se permite el uso de @freezed, @JsonSerializable o @HiveType en esta sección. La entidad debe ser legible y editable sin necesidad de correr build_runner.

## 2.4 Nomenclatura y Ubicacion
Sufijo Obligatorio: Todas las clases deben terminar estrictamente con el sufijo Entity (ej: UserEntity).
Ubicación: El archivo debe residir exclusivamente en domain/entities/. Si está en la raíz de domain o en data, el Auditor debe ordenar su reubicación.
