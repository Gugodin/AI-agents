---
name: router-specialist
description: Reglas para la navegacion declarativa utilizando GoRouter con rutas tipadas, guards de autenticacion, deep linking e integracion con BLoC.
---

# Skill: Router Specialist - Navegacion Declarativa con GoRouter

Este manual define como implementar la navegacion en el proyecto Flutter utilizando `go_router` con un enfoque declarativo, rutas tipadas, guards de autenticacion y soporte para deep linking.

## 1. Configuracion de go_router

### 1.1 Dependencia en pubspec.yaml
```yaml
dependencies:
  go_router: ^14.0.0
```

### 1.2 Ubicacion del Router
La configuracion del router reside en `lib/config/router/`:

```
lib/config/router/
├── app_router.dart        # Configuracion principal del GoRouter
├── routes.dart            # Constantes de rutas (paths y nombres)
└── guards/
    └── auth_guard.dart    # Redirect logic para autenticacion
```

## 2. Definicion de Rutas

### 2.1 Constantes de Rutas (routes.dart)
Centralizar todos los paths en una clase de constantes para evitar strings magicos:

```dart
abstract class Routes {
  // Auth
  static const String login = '/login';
  static const String register = '/register';
  static const String forgotPassword = '/forgot-password';

  // Home
  static const String home = '/home';

  // Features
  static const String products = '/products';
  static const String productDetail = '/products/:id';

  // Profile
  static const String profile = '/profile';
  static const String settings = '/profile/settings';

  // Utility
  static const String splash = '/splash';
  static const String notFound = '/404';
}
```

### 2.2 Rutas con Parametros
Para rutas que reciben parametros, usar el patron `:paramName`:

```dart
// Definicion
static const String productDetail = '/products/:id';

// Uso en GoRoute
GoRoute(
  path: Routes.productDetail,
  builder: (context, state) {
    final productId = state.pathParameters['id']!;
    return ProductDetailPage(productId: productId);
  },
),
```

### 2.3 Rutas con Query Parameters
Para parametros opcionales, usar query parameters:

```dart
// Navegacion
context.go('${Routes.products}?category=electronics&sort=price');

// Lectura en el builder
GoRoute(
  path: Routes.products,
  builder: (context, state) {
    final category = state.uri.queryParameters['category'];
    final sort = state.uri.queryParameters['sort'];
    return ProductsPage(category: category, sort: sort);
  },
),
```

## 3. Configuracion del GoRouter (app_router.dart)

### 3.1 Estructura Principal
```dart
import 'package:go_router/go_router.dart';

final GlobalKey<NavigatorState> _rootNavigatorKey = GlobalKey<NavigatorState>();

GoRouter createRouter(AuthBloc authBloc) {
  return GoRouter(
    navigatorKey: _rootNavigatorKey,
    initialLocation: Routes.splash,
    debugLogDiagnostics: true,
    redirect: _globalRedirect,
    routes: [
      // Splash
      GoRoute(
        path: Routes.splash,
        builder: (context, state) => const SplashPage(),
      ),

      // Auth (sin shell)
      GoRoute(
        path: Routes.login,
        builder: (context, state) => const LoginPage(),
      ),
      GoRoute(
        path: Routes.register,
        builder: (context, state) => const RegisterPage(),
      ),

      // Shell principal (con BottomNavigationBar)
      ShellRoute(
        parentNavigatorKey: _rootNavigatorKey,
        builder: (context, state, child) => MainShell(child: child),
        routes: [
          GoRoute(
            path: Routes.home,
            builder: (context, state) => const HomePage(),
          ),
          GoRoute(
            path: Routes.products,
            builder: (context, state) => const ProductsPage(),
            routes: [
              GoRoute(
                path: ':id',
                builder: (context, state) {
                  final id = state.pathParameters['id']!;
                  return ProductDetailPage(productId: id);
                },
              ),
            ],
          ),
          GoRoute(
            path: Routes.profile,
            builder: (context, state) => const ProfilePage(),
            routes: [
              GoRoute(
                path: 'settings',
                builder: (context, state) => const SettingsPage(),
              ),
            ],
          ),
        ],
      ),

      // 404
      GoRoute(
        path: Routes.notFound,
        builder: (context, state) => const NotFoundPage(),
      ),
    ],
    errorBuilder: (context, state) => NotFoundPage(
      message: state.error?.message ?? 'Ruta no encontrada',
    ),
  );
}
```

### 3.2 ShellRoute para Navegacion Persistente
`ShellRoute` se usa para mantener un layout persistente (como un `BottomNavigationBar`) mientras las rutas hijas cambian:

```dart
class MainShell extends StatelessWidget {
  final Widget child;
  const MainShell({super.key, required this.child});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: child,
      bottomNavigationBar: BottomNavigationBar(
        currentIndex: _calculateSelectedIndex(context),
        onTap: (index) => _onItemTapped(index, context),
        items: const [
          BottomNavigationBarItem(icon: Icon(Icons.home), label: 'Inicio'),
          BottomNavigationBarItem(icon: Icon(Icons.inventory), label: 'Productos'),
          BottomNavigationBarItem(icon: Icon(Icons.person), label: 'Perfil'),
        ],
      ),
    );
  }

  int _calculateSelectedIndex(BuildContext context) {
    final location = GoRouterState.of(context).uri.path;
    if (location.startsWith(Routes.home)) return 0;
    if (location.startsWith(Routes.products)) return 1;
    if (location.startsWith(Routes.profile)) return 2;
    return 0;
  }

  void _onItemTapped(int index, BuildContext context) {
    switch (index) {
      case 0: context.go(Routes.home); break;
      case 1: context.go(Routes.products); break;
      case 2: context.go(Routes.profile); break;
    }
  }
}
```

## 4. Guards de Autenticacion (Redirect Global)

### 4.1 Logica de Redirect
El redirect global decide si el usuario puede acceder a una ruta basandose en su estado de autenticacion:

```dart
String? _globalRedirect(BuildContext context, GoRouterState state) {
  final authBloc = context.read<AuthBloc>();
  final isAuthenticated = authBloc.state.isAuthenticated;
  final currentPath = state.uri.path;

  // Rutas publicas (no requieren autenticacion)
  final publicRoutes = [Routes.login, Routes.register, Routes.forgotPassword, Routes.splash];
  final isPublicRoute = publicRoutes.any((route) => currentPath.startsWith(route));

  // Si no esta autenticado y la ruta NO es publica, redirigir a login
  if (!isAuthenticated && !isPublicRoute) {
    return Routes.login;
  }

  // Si esta autenticado y esta en login/register, redirigir a home
  if (isAuthenticated && isPublicRoute) {
    return Routes.home;
  }

  return null; // No redirect
}
```

### 4.2 RefreshListenable con BLoC
Para que el router reaccione a cambios de autenticacion en tiempo real:

```dart
GoRouter createRouter(AuthBloc authBloc) {
  return GoRouter(
    initialLocation: Routes.splash,
    redirect: _globalRedirect,
    refreshListenable: GoRouterRefreshStream(authBloc.stream),
    routes: [ /* ... */ ],
  );
}

class GoRouterRefreshStream extends ChangeNotifier {
  late final StreamSubscription _subscription;

  GoRouterRefreshStream(Stream stream) {
    _subscription = stream.listen((_) => notifyListeners());
  }

  @override
  void dispose() {
    _subscription.cancel();
    super.dispose();
  }
}
```

## 5. Navegacion Programatica

### 5.1 Metodos de GoRouter
| Metodo | Uso | Ejemplo |
|--------|-----|---------|
| `context.go(path)` | Reemplaza toda la pila de navegacion | `context.go(Routes.home)` |
| `context.push(path)` | Agrega a la pila (permite back) | `context.push(Routes.productDetail)` |
| `context.goNamed(name)` | Navega por nombre de ruta | `context.goNamed('home')` |
| `context.pop()` | Regresa a la pantalla anterior | `context.pop()` |
| `context.replace(path)` | Reemplaza la ruta actual | `context.replace(Routes.home)` |

### 5.2 Navegacion con Parametros
```dart
// Con path parameters
context.push('/products/${product.id}');

// Con query parameters
context.push('${Routes.products}?category=${category.id}');

// Con extra (objeto completo)
context.push(
  Routes.productDetail,
  extra: ProductDetailArgs(product: product, showReviews: true),
);
```

### 5.3 Lectura de Extra en el Builder
```dart
GoRoute(
  path: Routes.productDetail,
  builder: (context, state) {
    final args = state.extra as ProductDetailArgs?;
    return ProductDetailPage(
      productId: state.pathParameters['id']!,
      showReviews: args?.showReviews ?? false,
    );
  },
),
```

## 6. Deep Linking

### 6.1 Configuracion en Android (AndroidManifest.xml)
```xml
<intent-filter>
  <action android:name="android.intent.action.VIEW" />
  <category android:name="android.intent.category.DEFAULT" />
  <category android:name="android.intent.category.BROWSABLE" />
  <data android:scheme="https" android:host="miapp.com" />
</intent-filter>
```

### 6.2 Configuracion en iOS (Info.plist)
```xml
<key>FlutterDeepLinkingEnabled</key>
<true/>
```

### 6.3 Manejo de Deep Links
GoRouter maneja automaticamente los deep links si las rutas coinciden:

```
https://miapp.com/products/123  →  ProductDetailPage(productId: '123')
https://miapp.com/profile       →  ProfilePage()
```

## 7. Integracion con la Arquitectura

### 7.1 Router en el Inyector
El GoRouter se registra como Singleton en el inyector:

```dart
// injector/dependencies/router_dependencies.dart
void routerDependencies() {
  getIt.registerSingleton<GoRouter>(
    createRouter(getIt<AuthBloc>()),
  );
}
```

### 7.2 Uso en App Widget
```dart
class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    final router = getIt<GoRouter>();
    return MaterialApp.router(
      routerConfig: router,
      theme: appTheme,
    );
  }
}
```

### 7.3 Restricciones
* **Cero Navigator.push**: PROHIBIDO usar `Navigator.push` o `Navigator.of(context).push`. Toda navegacion debe pasar por GoRouter (`context.go`, `context.push`).
* **Rutas Centralizadas**: PROHIBIDO definir paths como strings literales en los widgets. Siempre usar las constantes de `Routes`.
* **Sin Lógica de Auth en Widgets**: La verificacion de autenticacion reside en el redirect global del router, NO en los widgets.

## 8. Reglas de Verificacion para el Auditor (Router Edition)

### 8.1 Cero Navigator.push
El Auditor debe buscar cualquier uso de `Navigator.push`, `Navigator.of(context).push` o `Navigator.pushNamed`. Es un ERROR CRITICO. Toda navegacion debe usar `context.go()` o `context.push()`.

### 8.2 Rutas Centralizadas
Verificar que NO existan strings literales de rutas en los widgets (ej: `context.go('/home')`). Siempre deben usarse las constantes de la clase `Routes`.

### 8.3 Guard de Autenticacion
Confirmar que el redirect global este implementado y que el `refreshListenable` este conectado al BLoC de autenticacion.

### 8.4 ShellRoute para Layout Persistente
Si la app tiene `BottomNavigationBar` o `Drawer`, verificar que se use `ShellRoute` para mantener el layout persistente.

### 8.5 Error Handling
Verificar que exista un `errorBuilder` en la configuracion del GoRouter para manejar rutas no encontradas.

### 8.6 Deep Linking
Si el proyecto requiere deep linking, verificar la configuracion nativa en AndroidManifest.xml e Info.plist.
