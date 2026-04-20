---
name: ui-specialist
description: Reglas para la construccion de interfaces modulares, manejo de layouts responsivos e integracion con BLoC.
---

# Skill: UI Specialist - El Rostro de la Feature

Este manual define como construir interfaces escalables y limpias. La premisa es: "Si el widget principal de una pantalla tiene mas de 100 lineas, necesita ser dividido".

## 1. Jerarquia de Construccion (Estrategia de "Partes")
Para mantener la legibilidad, dividiremos cada pantalla (`Page`) en componentes mas pequeños. No usaremos funciones que retornan widgets, sino clases de widgets independientes o archivos de "partes".

### 1.1 Estructura de Archivos
* **[feature]_page.dart**: Solo contiene el `BlocProvider` y el `Scaffold` basico. Es el punto de entrada.
* **[feature]_view.dart**: Contiene la estructura general (Header, Body, Footer) y el `BlocBuilder`.
* **widgets/**: Carpeta dentro de la feature para componentes reutilizables o partes complejas de la pantalla.

### 1.2 Layout Responsivo: Expanded vs Scroll
Para evitar errores de desbordamiento (Overflow), seguiremos estas reglas de oro:

* **Vistas "Snapshot" (Pantalla completa sin scroll)**: 
    * Se utiliza un `Column` como base.
    * Se dividen las secciones mediante widgets `Expanded` o `Flexible` para que la UI se adapte dinamicamente al alto de la pantalla.
* **Vistas con Scroll (Listados o Formularios)**:
    * Se utiliza un `SingleChildScrollView` o `ListView`.
    * Los widgets internos deben tener alturas claras. Si se requiere un widget que ocupe el resto del espacio dentro de un scroll, se deben usar `Sliver` widgets.

### 1.3 Ejemplo de Estructura de Pantalla
```dart
class LoginPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return BlocProvider(
      create: (context) => GetIt.I<LoginBloc>(),
      child: const Scaffold(
        body: LoginView(), // El "View" separa la inyeccion del layout
      ),
    );
  }
}

class LoginView extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        const _LoginHeader(), // Parte extraida
        Expanded(
          child: const _LoginForm(), // Parte que ocupa el espacio restante
        ),
        const _LoginFooter(), // Parte inferior fija
      ],
    );
  }
}
```

## 2. Integracion Reactiva con BLoC
La comunicacion entre el BLoC y la UI debe ser clara y unidireccional. Dado que el proyecto utiliza una estrategia de provision centralizada, la UI se enfoca principalmente en el consumo de estados.

### 2.1 Provision de Blocs (Global vs Local)
* **Provision Global (Estandar)**: Por regla general, los BLoCs se definen en el widget raiz (`SegiApp`) mediante un `MultiBlocProvider`. Esto garantiza que los estados esten disponibles en todo el arbol de navegacion sin necesidad de reinyeccion manual.
* **Provision Local (Excepcion)**: Solo se creara un `BlocProvider` a nivel de widget/pantalla en casos especiales donde el BLoC requiera parametros unicos del constructor de la vista o tenga un ciclo de vida estrictamente efimero.

### 2.2 BlocBuilder: El "Pintor" de la UI
Se utiliza para **dibujar** widgets basados en el estado actual.
* **Consumo de Contexto**: Al estar inyectados globalmente, se accede a ellos via `context.read<MyBloc>()` o simplemente dejando que el `BlocBuilder` encuentre el tipo en el arbol.
* **Uso de Extensions**: Es OBLIGATORIO usar los getters de la extension (ej: `state.isLoading`) para mantener la legibilidad.

### 2.3 BlocListener: Gestion de Efectos Secundarios
Se utiliza para acciones que no reconstruyen la UI (Navegacion, Snackbars, Dialogs). Es vital para manejar el flujo despues de una operacion exitosa o un error critico.

### 2.4 Ejemplo de Integracion (Consumo Global)
```dart
class CatalogsView extends StatelessWidget {
  const CatalogsView({super.key});

  @override
  Widget build(BuildContext context) {
    return BlocConsumer<CatalogsBloc, CatalogsState>(
      listener: (context, state) {
        if (state.isError) {
          // Toastification o SnackBar segun estandar del proyecto
        }
      },
      builder: (context, state) {
        // Uso de extensiones para un build limpio
        if (state.isLoading) return const Center(child: CircularProgressIndicator());
        
        return ListView.builder(
          itemCount: state.reasons.length,
          itemBuilder: (context, index) => ListTile(
            title: Text(state.reasons[index].name),
          ),
        );
      },
    );
  }
}
```

## 3. Reglas de Verificacion para el Auditor (UI Edition)
1. Verificacion de Inyeccion: El Auditor debe verificar si el BLoC ya existe en el MultiBlocProvider global antes de permitir un BlocProvider local. Se debe evitar la duplicidad de instancias.
2. Modularizacion (Regla 80/20): Si un widget de vista supera las 80 lineas, se debe exigir la extraccion de sub-widgets.
3. Uso de Const: Obligatorio en widgets estaticos para optimizar el rendimiento del framework.
4. Respeto al Listener: Queda prohibido realizar navegaciones (Navigator.push) dentro de un builder. El Auditor marcara esto como ERROR; debe moverse al listener.
5. Layout Safety: Verificar el uso de SafeArea y el manejo de desbordamientos mediante Expanded o SingleChildScrollView segun el tipo de vista (Punto 1.2).