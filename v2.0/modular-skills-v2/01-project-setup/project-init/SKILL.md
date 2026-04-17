---
name: project-init
description: Protocolo de entrevista inicial para la configuracion personalizada de proyectos Flutter (Arquitectura Segi).
---

# Skill: Project Initialization - El Guion del Entrevistador

Este documento define el comportamiento obligatorio del Agente de Setup al recibir la orden de crear un nuevo proyecto. No se permite la creacion de ningun archivo sin haber completado y validado este cuestionario con el usuario.

## 1. El Cuestionario de Configuracion
El Agente debe presentar estas preguntas de forma clara y esperar la respuesta del usuario para cada una (o procesarlas en bloque si el usuario las provee de inicio):

1. **Nombre del Proyecto**: Debe estar en `snake_case` (ej: `stoki_pos`, `app_cobranza`). Se utilizara para los imports internos y nombres de archivos.
2. **Organizacion (Org)**: El prefijo del identificador (ej: `com.tuempresa`). Se utilizara para generar el Bundle ID en iOS y Package Name en Android.
3. **Entornos (Flavors)**: Preguntar si requiere la triada estandar (`dev`, `qa`, `prod`) o si solo se trabajara en un entorno unico de produccion.
4. **Modulos Core Opcionales**: Ofrecer una lista de funcionalidades pre-validadas que se pueden inyectar desde el inicio para acelerar el desarrollo:
    * **Toastification**: Sistema de notificaciones visuales (Toasts).
    * **SharedPreferences**: Persistencia de datos simple (Key-Value).
    * **Biometria**: Autenticacion segura por huella o rostro.
    * **Connectivity**: Monitoreo del estado de conexion a internet.
    * **Geolocalizacion**: Acceso a coordenadas GPS.

## 2. Validaciones Tecnicas Inmediatas
El Agente no debe aceptar respuestas que violen los estandares de Flutter o de la arquitectura:

* **Nombre**: Si el usuario introduce espacios o mayusculas (ej: "Mi Proyecto"), el Agente debe corregir o solicitar el cambio a `mi_proyecto`.
* **Org**: Debe seguir el formato de dominio inverso (Reverse DNS).

## 3. Logica de Valores por Defecto (Fallback)
Para agilizar la entrevista, el Agente debe permitir que el usuario omita ciertos campos, aplicando las siguientes reglas de negocio:

1. **Organizacion (Org)**: Si el usuario no proporciona un identificador de organizacion, el Agente debe generar uno automaticamente siguiendo el patron `com.[nombre_del_proyecto]`.
    * Ejemplo: Si el proyecto se llama `stoki_pos`, el Bundle ID sera `com.stoki_pos`.
2. **Entornos**: Si se omite, se asumira por defecto que el proyecto requiere la triada de Flavors (`dev`, `qa`, `prod`) para cumplir con el estandar de la arquitectura Segi.
3. **Modulos Core**: Si el usuario no selecciona ninguno, el proyecto se creara unicamente con la estructura de carpetas base (Clean Scaffolding), sin dependencias adicionales de hardware o persistencia.

## 4. Mapeo de Decisiones para Agentes Secundarios
Una vez finalizada la entrevista, el Agente de Setup debe emitir una "Ficha de Proyecto" que servira como guia para los siguientes Skills:

* **Hacia native-configuration**: Pasar el Bundle ID final (provisto o generado) y la lista de permisos de hardware a inyectar.
* **Hacia scaffolding-dart**: Pasar el nombre del proyecto para configurar los imports y determinar si se crean los archivos `main_*.dart` segun los entornos seleccionados.
* **Hacia dependencies-manager**: Lista de paquetes (pubspec.yaml) que deben instalarse segun los Modulos Core seleccionados (ej: flutter_bloc, get_it, dio, biometrics).

## 5. Reglas de Verificacion para el Auditor (Init Edition)
1. **Validacion de Nombre**: El Auditor debe confirmar que el nombre del proyecto no contenga palabras reservadas de Dart (ej: `void`, `class`, `test`).
2. **Confirmacion de Org**: Si se uso el valor por defecto `com.[nombre]`, el Auditor debe verificar que el Agente lo haya notificado al usuario antes de proceder.
3. **Bloqueo de Scaffolding**: Es un ERROR que el Agente comience a crear carpetas si el usuario aun no ha confirmado el resumen final de la entrevista.