---
name: i18n-specialist
description: Reglas para la internacionalizacion (i18n) del proyecto Flutter utilizando el paquete intl con archivos ARB y generacion automatica de localizations.
---

# Skill: I18n Specialist - Internacionalizacion con intl

Este manual define como implementar la internacionalizacion en el proyecto Flutter utilizando el paquete oficial `intl` con archivos ARB (Application Resource Bundle) y la generacion automatica de clases de localizacion.

## 1. Configuracion de intl

### 1.1 Dependencias en pubspec.yaml
```yaml
dependencies:
  flutter_localizations:
    sdk: flutter
  intl: any

flutter:
  generate: true
```

### 1.2 Archivo l10n.yaml
Crear el archivo `l10n.yaml` en la raiz del proyecto:

```yaml
arb-dir: lib/l10n
template-arb-file: app_es.arb
output-localization-file: app_localizations.dart
output-class: AppLocalizations
synthetic-package: false
output-dir: lib/l10n/generated
nullable-getter: false
```

### 1.3 Configuracion en MaterialApp
```dart
import 'package:flutter_localizations/flutter_localizations.dart';
import 'package:mi_proyecto/l10n/generated/app_localizations.dart';

MaterialApp(
  localizationsDelegates: const [
    AppLocalizations.delegate,
    GlobalMaterialLocalizations.delegate,
    GlobalWidgetsLocalizations.delegate,
    GlobalCupertinoLocalizations.delegate,
  ],
  supportedLocales: const [
    Locale('es'),
    Locale('en'),
    Locale('pt'),
  ],
  locale: const Locale('es'),
);
```

## 2. Estructura de Archivos ARB

### 2.1 Organizacion de Carpetas
```
lib/
└── l10n/
    ├── app_es.arb          # Archivo plantilla (espanol)
    ├── app_en.arb          # Ingles
    ├── app_pt.arb          # Portugues
    └── generated/          # Codigo generado (NO editar)
        ├── app_localizations.dart
        ├── app_localizations_es.dart
        ├── app_localizations_en.dart
        └── app_localizations_pt.dart
```

### 2.2 Archivo Plantilla (app_es.arb)
El archivo plantilla define todas las claves y sus metadatos:

```json
{
  "@@locale": "es",
  "appTitle": "Mi Aplicacion",
  "@appTitle": {
    "description": "Titulo de la aplicacion en la barra de estado"
  },
  "welcomeMessage": "Bienvenido, {userName}",
  "@welcomeMessage": {
    "description": "Mensaje de bienvenida en la pantalla de inicio",
    "placeholders": {
      "userName": {
        "type": "String",
        "example": "Juan"
      }
    }
  },
  "itemCount": "{count, plural, =0{No hay elementos} =1{1 elemento} other{{count} elementos}}",
  "@itemCount": {
    "description": "Contador de elementos con pluralizacion",
    "placeholders": {
      "count": {
        "type": "int"
      }
    }
  },
  "loginButton": "Iniciar Sesion",
  "@loginButton": {
    "description": "Texto del boton de login"
  },
  "errorGeneric": "Algo salio mal. Intenta de nuevo.",
  "@errorGeneric": {
    "description": "Mensaje de error generico reutilizable"
  }
}
```

### 2.3 Archivos de Traduccion (app_en.arb)
Los archivos de traduccion SOLO contienen las claves y los valores traducidos (sin metadatos):

```json
{
  "@@locale": "en",
  "appTitle": "My Application",
  "welcomeMessage": "Welcome, {userName}",
  "itemCount": "{count, plural, =0{No items} =1{1 item} other{{count} items}}",
  "loginButton": "Sign In",
  "errorGeneric": "Something went wrong. Try again."
}
```

## 3. Generacion de Localizations

### 3.1 Comando de Generacion
Despues de crear o modificar archivos ARB, ejecutar:

```bash
flutter gen-l10n
```

O para generacion automatica al compilar, agregar en `pubspec.yaml`:

```yaml
flutter:
  generate: true
```

### 3.2 Archivos Generados
La generacion produce:
* **app_localizations.dart**: Clase base `AppLocalizations` con el metodo `of(context)`
* **app_localizations_[locale].dart**: Implementaciones por idioma

## 4. Uso en Widgets

### 4.1 Acceso Basico
```dart
import 'package:mi_proyecto/l10n/generated/app_localizations.dart';

class WelcomeView extends StatelessWidget {
  const WelcomeView({super.key});

  @override
  Widget build(BuildContext context) {
    final l10n = AppLocalizations.of(context);
    
    return Text(l10n.welcomeMessage('Juan'));
  }
}
```

### 4.2 Extension de Contexto (Recomendado)
Crear una extension para acceso mas limpio:

```dart
// lib/core/extensions/context_extensions.dart
import 'package:mi_proyecto/l10n/generated/app_localizations.dart';

extension LocalizationExtension on BuildContext {
  AppLocalizations get l10n => AppLocalizations.of(this);
}
```

Uso:
```dart
import 'package:mi_proyecto/core/extensions/context_extensions.dart';

class LoginView extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return ElevatedButton(
      onPressed: () {},
      child: Text(context.l10n.loginButton),
    );
  }
}
```

### 4.3 Pluralizacion y Select
```dart
// Pluralizacion
Text(context.l10n.itemCount(5)); // "5 elementos"

// Select (genero, etc.)
// En ARB: "genderMessage": "{gender, select, male{El} female{La} other{El/La}} usuario"
Text(context.l10n.genderMessage('female')); // "La usuario"
```

## 5. Organizacion de Claves ARB

### 5.1 Nomenclatura de Claves
Usar `camelCase` con prefijos por feature o contexto:

| Prefijo | Uso | Ejemplo |
|---------|-----|---------|
| `common` | Textos reutilizables globales | `commonAccept`, `commonCancel` |
| `auth` | Feature de autenticacion | `authLoginTitle`, `authEmailLabel` |
| `home` | Feature de inicio | `homeWelcomeTitle`, `homeEmptyList` |
| `error` | Mensajes de error | `errorNetwork`, `errorGeneric` |
| `validation` | Validaciones de formularios | `validationRequired`, `validationEmail` |

### 5.2 Claves Comunes Obligatorias
Todo proyecto debe incluir estas claves base:

```json
{
  "commonAccept": "Aceptar",
  "commonCancel": "Cancelar",
  "commonSave": "Guardar",
  "commonDelete": "Eliminar",
  "commonEdit": "Editar",
  "commonSearch": "Buscar",
  "commonLoading": "Cargando...",
  "commonRetry": "Reintentar",
  "commonClose": "Cerrar",
  "commonYes": "Si",
  "commonNo": "No",
  "errorGeneric": "Algo salio mal. Intenta de nuevo.",
  "errorNetwork": "No hay conexion a internet.",
  "errorTimeout": "La conexion tardo demasiado."
}
```

## 6. Cambio de Idioma en Runtime

### 6.1 Estrategia con BLoC
Para cambio dinamico de idioma, usar un BLoC global que gestione el `Locale`:

```dart
class LocaleBloc extends Cubit<Locale> {
  LocaleBloc() : super(const Locale('es'));

  void changeLocale(String languageCode) {
    emit(Locale(languageCode));
  }
}
```

### 6.2 Integracion con MaterialApp
```dart
class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return BlocBuilder<LocaleBloc, Locale>(
      builder: (context, locale) {
        return MaterialApp(
          locale: locale,
          localizationsDelegates: const [
            AppLocalizations.delegate,
            GlobalMaterialLocalizations.delegate,
            GlobalWidgetsLocalizations.delegate,
            GlobalCupertinoLocalizations.delegate,
          ],
          supportedLocales: const [
            Locale('es'),
            Locale('en'),
          ],
        );
      },
    );
  }
}
```

## 7. Reglas de Verificacion para el Auditor (I18n Edition)

### 7.1 Cero Strings Hardcodeados
El Auditor debe buscar strings literales en espanol (o cualquier idioma) directamente en los widgets. Cualquier `Text('Hola mundo')` es un ERROR. Debe reemplazarse por `Text(context.l10n.someKey)`.

### 7.2 Archivo Plantilla Completo
Verificar que `app_es.arb` (o el idioma base) contenga TODAS las claves necesarias. Las traducciones faltantes en otros idiomas generaran warnings pero no errores de compilacion.

### 7.3 Metadatos en Plantilla
Confirmar que el archivo plantilla incluya metadatos (`@clave`) con `description` para cada entrada. Esto facilita el trabajo de los traductores.

### 7.4 Pluralizacion Correcta
Verificar que los contadores y cantidades usen `plural` en lugar de concatenacion manual de strings.

### 7.5 Generacion Actualizada
Si se modificaron archivos ARB, verificar que `flutter gen-l10n` se ejecuto correctamente.

### 7.6 Carpeta generated/ en .gitignore
Confirmar que `lib/l10n/generated/` este en `.gitignore` ya que se regenera automaticamente.

### 7.7 Nomenclatura de Claves
Verificar que las claves usen `camelCase` con prefijos por feature (`auth`, `home`, `common`, `error`).
