---
name: code-reviewer
description: Protocolo de auditoria final para la aprobacion de Pull Requests y control de calidad arquitectonica.
---

# Skill: Code Reviewer - El Guardian de la Arquitectura

Este manual define el estandar de revision para garantizar que cada linea de codigo respete la arquitectura. El Code Reviewer es el ultimo filtro antes de que el codigo llegue a produccion.

## 1. El Checklist de Oro (Cross-Layer)
Antes de entrar al detalle de cada capa, el revisor debe validar estos tres pilares fundamentales:
1. **Independencia de Capas**: ¿Hay imports de `presentation` en `domain`? ¿Hay `models` en `domain`? Si la respuesta es SI, el PR se rechaza automaticamente.
2. **Inyeccion de Dependencias**: ¿Se instanció algo con un constructor manual (`new`) en lugar de usar `getIt` o inyeccion por constructor?
3. **Naming Convention**: ¿Los archivos y clases siguen el sufijo de su especialidad (`_entity`, `_model`, `_repository`, `_bloc`, `_view`)?

## 2. Auditoria por Especialidad

### 2.1 Domain Check (Pureza)
* **Entidades**: Deben ser 100% Dart Nativo. Sin Freezed, sin JSON, con `final` y `const`.
* **Repositorios**: Deben ser interfaces abstractas puras que retornen `DataResult<T>` o `DataResultVoid`.
* **UseCases**: Deben tener una unica responsabilidad y usar el metodo `call`.

### 2.2 Data Check (Infraestructura)
* **Models**: Deben extender de la Entidad y ser Dart Nativo (prohibido Freezed aqui). Deben manejar el `fromJson` defensivamente.
* **DataSources**: Deben retornar `Future<ResponseModel<dynamic>>`. Si es Retrofit, verificar las anotaciones; si es manual, verificar el `try-catch` interno.
* **RepositoryImpl**: Es obligatorio el uso de `try-catch` con `DataStateFactory` para mapear errores de red y generales.

### 2.3 Presentation Check (Reactividad y UI)
* **BLoC**: Los estados DEBEN usar Freezed y los eventos Equatable. El registro en GetIt debe ser estrictamente `registerFactory`.
* **UI**: Verificar la estrategia de "Partes". Si el widget es un "monolito" (archivo gigante), debe segmentarse.
* **Logic Separation**: Ninguna navegacion o SnackBar puede ocurrir dentro de un `BlocBuilder`; todo efecto secundario debe estar en un `BlocListener`.

## 3. Protocolo de Rechazo y Aprobacion

### 3.1 Motivos de Rechazo Inmediato (Red Flags)
* Uso de `print()` o `debugPrint()` (usar el logger del sistema).
* Lógica de negocio (cálculos, filtrados complejos) dentro de la UI o el BLoC.
* Falta de manejo de errores en un flujo asíncrono (ignorar el `dioError` o `generalError`).
* Código "muerto" o comentado que no tenga un TODO con ticket de seguimiento.

### 3.2 Feedback Constructivo
El revisor no solo dice "esto está mal", sino que referencia el skill correspondiente:
> *"Rechazado: El `UserModel` está usando `@freezed`. Segun el **data-specialist/SKILL.md (Punto 1.1)**, los modelos deben ser Dart Nativo para mantener la coherencia con las entidades."*

## 4. Reglas de Verificacion (Meta-Auditoria)
1. **Consistencia de Typedefs**: El revisor debe asegurar que no se esten inventando tipos de retorno nuevos si ya existen en el `foundation/typedefs.dart`.
2. **Limpieza de GetIt**: Verificar que cada nueva feature tenga su correspondiente archivo de inyeccion o entrada en el `injector.dart`.
3. **Inmutabilidad**: Verificar que no existan variables globales o estados estaticos que puedan causar efectos colaterales entre sesiones de usuario.
## 5. Mecanismo de Defensa contra Orquestador Perezoso (RECHAZO POR FALTA DE CONTEXTO)
REGLA CRITICA: Como Quality Auditor, naces "ciego" a la arquitectura hasta que te la inyectan. Si el Technical-Lead te pide auditar un archivo (por ejemplo, `data_state.dart`, `typedefs.dart` o un `UseCase`) pero **NO INCLUYE** explicitamente en tu prompt el contenido o los snippets del skill correspondiente (ej. `blueprint-foundation` o `data-state-standard`), tu veredicto automatico e inmediato debe ser `REQUEST CHANGES`.
Mensaje de rechazo a emitir: *"No puedo auditar este codigo. El Technical Lead no me ha proporcionado las reglas del estandar para poder comparar. Inyecta el contenido del skill requerido en mi prompt para continuar."*
