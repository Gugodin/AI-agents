---
name: flutter-gen-specialist
description: Reglas para la gestion de assets (imagenes, fuentes, iconos, SVGs) utilizando flutter_gen con tipos seguros y generacion automatica de codigo.
---

# Skill: Flutter Gen Specialist - Gestion de Assets con Tipos Seguros

Este manual define como gestionar los recursos estaticos del proyecto (imagenes, fuentes, iconos, archivos SVG) utilizando `flutter_gen` para generar referencias tipadas que eliminen los strings magicos y prevengan errores en tiempo de compilacion.

## 1. Configuracion de flutter_gen

### 1.1 Dependencias en pubspec.yaml
Agregar las siguientes dependencias de desarrollo:

```yaml
dev_dependencies:
  flutter_gen: ^5.4.0
  flutter_gen_runner: ^5.4.0
```

### 1.2 Configuracion en pubspec.yaml
La configuracion de `flutter_gen` reside en el `pubspec.yaml` bajo la clave `flutter_gen`:

```yaml
flutter_gen:
  output: lib/gen/
  line_length: 80
  null_safety: true

  integrations:
    flutter_svg: true

  assets:
    enabled: true
    outputs:
      class_name: Assets
      package_parameter_enabled: false
      style: camel-case

  fonts:
    enabled: true
    outputs:
      class_name: FontFamily

  colors:
    enabled: true
    inputs:
      - assets/colors/colors.xml
    outputs:
      class_name: AppColors
```

### 1.3 Declaracion de Assets en pubspec.yaml
Todos los assets deben declararse en la seccion `flutter.assets`:

```yaml
flutter:
  assets:
    - assets/images/
    - assets/images/icons/
    - assets/images/logos/
    - assets/svgs/
    - assets/animations/

  fonts:
    - family: Poppins
      fonts:
        - asset: assets/fonts/Poppins-Regular.ttf
        - asset: assets/fonts/Poppins-Bold.ttf
          weight: 700
        - asset: assets/fonts/Poppins-Medium.ttf
          weight: 500
```

## 2. Estructura de Carpetas de Assets

### 2.1 Organizacion Estandar
```
assets/
├── images/
│   ├── icons/           # Iconos PNG/JPG
│   ├── logos/           # Logos de la empresa/app
│   ├── backgrounds/     # Fondos de pantalla
│   └── illustrations/   # Ilustraciones y graficos
├── svgs/                # Archivos SVG
├── animations/          # Archivos Lottie/Rive
├── fonts/               # Tipografias personalizadas
└── colors/              # Archivo colors.xml (opcional)
```

### 2.2 Nomenclatura de Archivos
* **Imagenes**: `snake_case` descriptivo (ej: `ic_home.png`, `logo_company.png`, `bg_login.jpg`)
* **SVGs**: `snake_case` descriptivo (ej: `ic_arrow_right.svg`, `illustration_empty.svg`)
* **Fuentes**: Nombre oficial de la fuente (ej: `Poppins-Regular.ttf`, `Poppins-Bold.ttf`)
* **Prefijos recomendados**:
  - `ic_` para iconos
  - `bg_` para fondos
  - `img_` para imagenes generales
  - `logo_` para logos

## 3. Generacion de Codigo

### 3.1 Comando de Generacion
Despues de agregar nuevos assets o modificar la configuracion, ejecutar:

```bash
dart run build_runner build --delete-conflicting-outputs
```

O para generacion continua durante desarrollo:

```bash
dart run build_runner watch --delete-conflicting-outputs
```

### 3.2 Archivos Generados
`flutter_gen` genera los siguientes archivos en `lib/gen/`:

* **assets.gen.dart**: Contiene la clase `Assets` con referencias tipadas a todas las imagenes
* **fonts.gen.dart**: Contiene la clase `FontFamily` con las fuentes disponibles
* **colors.gen.dart**: Contiene la clase `AppColors` con los colores definidos (si se configura)

## 4. Uso en Widgets

### 4.1 Imagenes con Assets
```dart
import 'package:mi_proyecto/gen/assets.gen.dart';

class LogoWidget extends StatelessWidget {
  const LogoWidget({super.key});

  @override
  Widget build(BuildContext context) {
    return Assets.images.logos.companyLogo.image(
      width: 120,
      fit: BoxFit.contain,
    );
  }
}
```

### 4.2 SVGs con flutter_svg
```dart
import 'package:mi_proyecto/gen/assets.gen.dart';

class IconArrow extends StatelessWidget {
  const IconArrow({super.key});

  @override
  Widget build(BuildContext context) {
    return Assets.svgs.icArrowRight.svg(
      width: 24,
      height: 24,
      colorFilter: ColorFilter.mode(
        Theme.of(context).colorScheme.primary,
        BlendMode.srcIn,
      ),
    );
  }
}
```

### 4.3 Fuentes con FontFamily
```dart
import 'package:mi_proyecto/gen/fonts.gen.dart';

TextStyle headlineStyle = TextStyle(
  fontFamily: FontFamily.poppins,
  fontWeight: FontWeight.w700,
  fontSize: 24,
);
```

### 4.4 Colores con AppColors
```dart
import 'package:mi_proyecto/gen/colors.gen.dart';

Container(
  color: AppColors.primaryBlue,
  child: Text(
    'Hola',
    style: TextStyle(color: AppColors.textWhite),
  ),
);
```

## 5. Reglas de Uso Estrictas

### 5.1 Cero Strings Magicos
PROHIBIDO usar strings directos para referenciar assets:

```dart
// INCORRECTO - String magico
Image.asset('assets/images/logos/company_logo.png');

// CORRECTO - Referencia tipada
Assets.images.logos.companyLogo.image();
```

### 5.2 Centralizacion en gen/
Todos los archivos generados residen en `lib/gen/`. Esta carpeta:
* NO debe editarse manualmente
* Debe estar en `.gitignore` (ya que se regenera con `build_runner`)
* Es la unica fuente de verdad para referencias de assets

### 5.3 Importacion Correcta
Siempre importar desde el package del proyecto:
```dart
import 'package:mi_proyecto/gen/assets.gen.dart';
import 'package:mi_proyecto/gen/fonts.gen.dart';
import 'package:mi_proyecto/gen/colors.gen.dart';
```

## 6. Reglas de Verificacion para el Auditor (Flutter Gen Edition)

### 6.1 Cero Strings Magicos
El Auditor debe buscar en todo el proyecto referencias directas a `assets/` usando grep o glob. Cualquier `Image.asset('assets/...')` o `SvgPicture.asset('assets/...')` es un ERROR CRITICO. Debe reemplazarse por la referencia tipada de `Assets`.

### 6.2 Declaracion Completa
Verificar que todo asset nuevo este declarado en el `pubspec.yaml` bajo `flutter.assets`. Un asset sin declarar no sera incluido en el build de produccion.

### 6.3 Nomenclatura de Archivos
Confirmar que los nombres de archivos de assets sigan `snake_case` con prefijos descriptivos (`ic_`, `bg_`, `img_`, `logo_`).

### 6.4 Generacion Actualizada
Si se agregaron assets nuevos, verificar que `dart run build_runner build` se ejecuto correctamente y que los archivos en `lib/gen/` estan actualizados.

### 6.5 Ubicacion de gen/
Confirmar que `lib/gen/` este en `.gitignore` y que ningun archivo generado este versionado en el repositorio.
