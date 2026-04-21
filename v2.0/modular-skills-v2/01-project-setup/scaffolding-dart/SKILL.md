---
name: scaffolding-dart
description: Reglas para la creacion del esqueleto del proyecto basado en la arquitectura, incluyendo manejo de ambientes.
---

# Skill: Scaffolding Dart - Estructura Architecture

Este documento define la organizacion de archivos y carpetas obligatoria. El Agente de Setup debe replicar esta jerarquia exacta para garantizar que los demas agentes (Data, Domain, Feature) encuentren sus dependencias.

## 1. Jerarquia de Carpetas en lib/
El Agente de Setup debe crear la siguiente estructura de directorios y archivos base:

```text
lib/
├── app/                 # Widget raiz de la aplicacion ({nombre_del_proyecto}_app.dart)
├── config/              # Configuracion global: routes, theme, utils, config.dart
├── core/                # Infraestructura tecnica y base de la arquitectura
│   ├── components/      # Widgets globales reutilizables
│   ├── constants/       # Constantes del sistema
│   ├── contracts/       # Interfaces y clases abstractas globales
│   ├── events/          # Definicion de eventos globales o EventBus
│   ├── foundation/      # ADN Arquitectonico (DataState, Typedefs)
│   ├── helpers/         # Funciones de ayuda general
│   ├── network/         # Clientes API, interceptores y fabricas de error
│   └── utils/           # Utilidades especificas de core
├── features/            # Modulos de negocio (Vertical Slices)
├── gen/                 # Codigo generado (assets, fuentes, etc.)
├── injector/            # Inyeccion de dependencias (dependencies/, injector.dart)
├── main_dev.dart        # Punto de entrada Ambiente Desarrollo
├── main_qa.dart         # Punto de entrada Ambiente QA
└── main.dart            # Punto de entrada Ambiente Produccion
```

## 2. Definicion de Responsabilidades Core

* app/: Contiene el widget principal que configura el MaterialApp/CupertinoApp y los wrappers globales (ScreenUtil, etc.).
* config/: Centraliza la navegacion (routes) y el estilo visual (theme) para evitar que esten regados por el proyecto.
* core/foundation/: Es el lugar OBLIGATORIO donde reside el DataState y los Typedefs globales definidos en las reglas de oro.
* injector/: Centraliza la configuracion de GetIt, separando las dependencias globales de las de cada feature en la carpeta dependencies/.

## 3. Puntos de Entrada y Configuracion de Ambientes
El Agente de Setup debe asegurar que los tres archivos main (`main.dart`, `main_dev.dart`, `main_qa.dart`) utilicen una funcion de inicializacion compartida para evitar duplicidad de codigo.

* **main_*.dart**: Cada archivo debe configurar las variables de entorno (URL base de API, nombre del flavor) antes de llamar a la funcion `runApp()`.
* **config.dart**: Es el archivo central que expone las variables de entorno al resto de la aplicacion de forma segura.

## 4. Logica de Inyeccion (Injector)
La carpeta `injector/` es el unico punto de registro de dependencias. Debe seguir un patron modular:

* **injector.dart**: Contiene la instancia global de GetIt y el metodo `init()` que dispara el registro de todos los modulos.
* **dependencies/**: Debe contener archivos separados por responsabilidad (ej: `core_dependencies.dart`, `network_dependencies.dart`, `feature_name_dependencies.dart`) para evitar que un solo archivo crezca indefinidamente.

## 5. Archivos Base de la Arquitectura (REGLA DE DELEGACION ESTRICTA)
Al realizar el scaffolding, el agente debe crear obligatoriamente estos archivos con sus esqueletos iniciales.
¡ADVERTENCIA CRITICA PARA EL ORQUESTADOR!: Si vas a delegar la creacion de estos archivos a otro agente (ej. `general`), TIENES ESTRICTAMENTE PROHIBIDO pedirle que los cree basandose en su "conocimiento de Flutter/Freezed". 
Debes copiar explicitamente el contenido del skill `data-state-standard` y pasarselo en el prompt al sub-agente. Si no lo haces, los archivos se generaran incorrectamente.

1. **lib/core/foundation/data_state.dart**: Contiene la definicion de DataState con Freezed.
2. **lib/core/foundation/typedefs.dart**: Contiene los alias de DataResult y DataResultVoid.
3. **lib/core/network/data_state_factory.dart**: Contiene la extension con el sistema de logging y mapeo de errores.
4. **lib/core/foundation/data_state_convenience.dart**: Contiene los getters de utilidad y propagacion.
5. **lib/app/{nombre_del_proyecto}_app.dart**: Widget base que inyecta los temas y rutas globales.

## 6. Reglas de Verificacion para el Auditor (Scaffolding Edition)
Criterios obligatorios para validar la integridad del esqueleto:

1. **Fidelidad Visual**: El Auditor debe comparar la estructura creada contra la imagen de referencia de la Arquitectura. Cualquier carpeta faltante (como `gen/` o `injector/dependencies/`) se marca como ERROR.
2. **Ubicacion del ADN**: El Auditor debe confirmar que `DataState` y `Typedefs` esten estrictamente dentro de `core/foundation/` y NO en la raiz de core.
3. **Mains de Flavor**: Es obligatorio que existan los tres archivos de entrada en la raiz de `lib/` si el proyecto fue configurado con Flavors en la entrevista inicial.
4. **Barriles (Export Files)**: Verificar que archivos como `core.dart` existan y exporten correctamente las utilidades de sus subcarpetas para facilitar los imports en el resto del proyecto.
## 7. Protocolo de Cierre y Compilacion (OBLIGATORIO)
Una vez que el Agente o sus sub-agentes hayan terminado de crear la estructura de carpetas, los archivos base (como DataState) y se hayan inyectado las dependencias en el pubspec.yaml, el Orquestador TIENE LA OBLIGACION de usar su herramienta `bash` para ejecutar secuencialmente los siguientes comandos en la raiz del proyecto:

1. Descargar dependencias: `flutter pub get`
2. Generar codigo (Freezed/JSON Serializable): `dart run build_runner build --delete-conflicting-outputs`

El Orquestador no puede dar por terminada la tarea de inicializacion sin confirmar que ambos comandos se ejecutaron exitosamente sin errores de compilacion.
