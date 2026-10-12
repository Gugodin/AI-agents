---
name: native-configuration
description: Estandares para la configuracion de plataformas nativas (Android/iOS), entornos y gestion modular de permisos.
---

# Skill: Native Configuration - Estandares de Plataforma

Este documento define las reglas para la configuracion de los archivos nativos. La premisa principal es el **Principio de Necesidad**: No se debe incluir ninguna configuracion, permiso o recurso que no haya sido solicitado explicitamente en la fase de inicializacion del proyecto.

## 1. Gestion de Entornos (Flavors / Schemes)
Es obligatorio que la aplicacion soporte multiples ambientes de ejecucion para separar los datos de desarrollo de los de produccion.

### 1.1 Definicion de Ambientes
A menos que se especifique lo contrario, el estandar base incluye:
* **Development (dev)**: Entorno de construccion diaria.
* **Quality Assurance (qa)**: Entorno de pruebas y certificacion.
* **Production (prod)**: Entorno final para el usuario final.

### 1.2 Identificadores de Paquete
Se debe utilizar una nomenclatura jerarquica para permitir la coexistencia de diferentes versiones del mismo proyecto en un dispositivo:
* **Prod**: `[org].[nombre_proyecto]`
* **QA**: `[org].[nombre_proyecto].qa`
* **Dev**: `[org].[nombre_proyecto].dev`

## 2. Configuracion Base de Android
El archivo `build.gradle` y el `AndroidManifest.xml` deben configurarse dinamicamente segun los requerimientos del proyecto.

* **minSdkVersion**: Se debe ajustar segun los Modulos Core seleccionados (ej: Biometria requiere min 23).
* **Namespace**: Debe coincidir con la organizacion definida en la entrevista inicial.

## 3. Configuracion Base de iOS (Runner)
La configuracion de iOS debe ser limpia y evitar la manipulacion manual de esquemas dentro de Xcode siempre que sea posible, priorizando archivos de configuracion.

* **Configuracion de Schemes**: Si se activaron Flavors, se deben crear esquemas correspondientes (dev, qa, prod).
* **Bundle Identifier**: Debe seguir la nomenclatura dinamica `[org].[proyecto].[sufijo]` definida en el protocolo de inicio.
* **Deployment Target**: Se debe establecer una version minima compatible con los Modulos Core seleccionados (ej: min 12.0 para soporte moderno).

## 4. Catalogo de Permisos Condicionales
El Agente de Setup solo aplicara estas configuraciones si el modulo correspondiente fue marcado como **True** en la entrevista inicial.

### 4.1 Modulo: Localizacion
* **Android**: Inyectar `<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />` en el `AndroidManifest.xml`.
* **iOS**: Inyectar las llaves `NSLocationWhenInUseUsageDescription` y `NSLocationAlwaysUsageDescription` en `Info.plist`.

### 4.2 Modulo: Biometria
* **Android**: Inyectar `<uses-permission android:name="android.permission.USE_BIOMETRIC" />`.
* **iOS**: Inyectar `NSFaceIDUsageDescription` en `Info.plist`.

### 4.3 Modulo: Almacenamiento / Camara
* Aplicar solo si el proyecto requiere gestion de archivos o captura de imagenes, siguiendo el mismo patron de inyeccion en los manifiestos de ambas plataformas.

## 5. Reglas de Verificacion para el Auditor (Native Edition)
Criterios obligatorios para validar la consistencia de las plataformas nativas:

1. **Principio de Minimos Permisos**: Es un ERROR CRITICO incluir permisos de GPS, Biometria o Camara si el usuario no los solicito expresamente en la entrevista.
2. **Coherencia de Identificadores**: El Auditor debe verificar que el Package Name en Android y el Bundle ID en iOS sean identicos y respeten los sufijos de entorno (.dev, .qa).
3. **Descripciones de Privacidad**: En iOS, los mensajes de "Usage Description" no pueden estar vacios ni ser genericos; deben describir brevemente para que se usa el sensor en el contexto del proyecto.