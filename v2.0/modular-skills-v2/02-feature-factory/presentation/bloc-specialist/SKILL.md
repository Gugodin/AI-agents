---
name: bloc-specialist
description: Gestion de estado reactiva utilizando BLoC, Equatable para eventos y Freezed para estados exhaustivos.
---

# Skill: BLoC Specialist - Reactividad Blindada

Este manual define como implementar la logica de presentacion. El BLoC actua como el mediador entre la UI y el Domain, asegurando que cada cambio de estado sea predecible y eficiente.



## 1. Definicion de Eventos y Estados (Equatable & Freezed)
Para optimizar el rendimiento y la legibilidad, utilizaremos **Equatable** en los eventos (evitando rebuilds redundantes) y **Freezed** en los estados (para aprovechar el pattern matching).

### 1.1 Eventos con Equatable
Los eventos representan acciones del usuario. Al extender de `Equatable`, nos aseguramos de que el BLoC solo procese eventos si sus datos realmente han cambiado.

### 1.2 Estados con Freezed (Union Classes)
Utilizaremos las "Union Classes" de Freezed para representar los estados. Esto nos obliga a manejar todos los casos posibles (Loading, Success, Error) en la UI mediante metodos como `.when()` o `.maybeWhen()`.

### 1.3 Ubicacion y Nomenclatura
* **Directorio**: `lib/features/[feature_name]/presentation/bloc/`.
* **Archivos**: `[nombre]_bloc.dart`, `[nombre]_event.dart`, `[nombre]_state.dart`.

### 1.4 Ejemplo de Implementacion
```dart
// product_event.dart
import 'package:equatable/equatable.dart';

abstract class ProductEvent extends Equatable {
  const ProductEvent();
  @override
  List<Object?> get props => [];
}

class ProductFetchStarted extends ProductEvent {
  final String category;
  const ProductFetchStarted(this.category);

  @override
  List<Object?> get props => [category];
}

// product_state.dart
import 'package:freezed_annotation/freezed_annotation.dart';
import '../../../domain/entities/product_entity.dart';

part 'product_state.freezed.dart';

@freezed
class ProductState with _$ProductState {
  const factory ProductState.initial() = _Initial;
  const factory ProductState.loading() = _Loading;
  const factory ProductState.success(List<ProductEntity> products) = _Success;
  const factory ProductState.error({
    required String message,
    String? idError,
  }) = _Error;
}
```

### 1.5 Extensiones de Estado (UI Helpers)
Para evitar la contaminacion de logica en los Widgets (casting manual o validaciones complejas), se DEBE crear una extension sobre el estado de Freezed. Estas extensiones actuan como getters seguros para acceder a la data desde la UI.

#### Reglas de la Extension:
1. **Getters de Estado**: Crear booleanos claros para estados comunes (`isLoading`, `isSuccess`, etc.).
2. **Acceso Seguro a Data**: Si un estado contiene datos (ej: una lista), el getter de la extension debe verificar el tipo y retornar el dato o un valor por defecto (ej: lista vacia) para evitar errores de nulidad o casting en la vista.

#### Ejemplo de Implementacion:
```dart
extension CatalogsStateX on CatalogsState {
  // Verificaciones rapidas de fase
  bool get isInitial => this is _Initial;
  bool get isLoading => this is _Loading;
  bool get isSuccess => this is _AllCatalogsLoaded;
  bool get isError => this is _Error;

  // Acceso seguro a las entidades
  List<StateEntity> get states => this is _AllCatalogsLoaded
      ? (this as _AllCatalogsLoaded).states
      : [];
  
  List<ReasonEntity> get reasons => this is _AllCatalogsLoaded
      ? (this as _AllCatalogsLoaded).reasons
      : [];

  // Error tracking simplificado
  String? get errorMessage => this is _Error 
      ? (this as _Error).message 
      : null;
}
```

## 2. Logica del BLoC y Mapeo de UseCases
El BLoC tiene prohibido instanciar dependencias internamente. Su funcion es coordinar los flujos de datos recibidos y emitir el estado visual correspondiente.

### 2.1 Inyeccion de Casos de Uso
Todos los Casos de Uso necesarios para la feature deben ser inyectados a traves del constructor. Esto facilita el testing y asegura que el BLoC sea una pieza intercambiable de la arquitectura.

### 2.2 Mapeo con Pattern Matching (DataResult.when)
Dado que el `UseCase` retorna un `DataResult` (basado en Freezed), el BLoC debe utilizar el metodo `.when()` para procesar la respuesta. Esto garantiza un manejo exhaustivo de:
1. **success**: Emision del estado con la data de la entidad.
2. **dioError**: Emision del estado de error con detalles especificos de red.
3. **generalError**: Emision del estado de error para fallos logicos o inesperados.

### 2.3 Ejemplo de Implementacion (Flujo Estandar)
```dart
class CatalogsBloc extends Bloc<CatalogsEvent, CatalogsState> {
  final GetAllCatalogsUseCase _getAllCatalogsUseCase;

  CatalogsBloc({
    required GetAllCatalogsUseCase getAllCatalogsUseCase,
  })  : _getAllCatalogsUseCase = getAllCatalogsUseCase,
        super(const CatalogsState.initial()) {
    on<_GetAllCatalogRequested>(_onGetAllCatalogRequested);
  }

  Future<void> _onGetAllCatalogRequested(
    _GetAllCatalogRequested event,
    Emitter<CatalogsState> emit,
  ) async {
    emit(const CatalogsState.loading());

    final result = await _getAllCatalogsUseCase();

    // Mapeo exhaustivo del resultado de dominio al estado de presentacion
    result.when(
      success: (allCatalogs) => emit(CatalogsState.allCatalogsLoaded(
        states: allCatalogs.states,
        catalogStatus: allCatalogs.catalogStatus,
        reasons: allCatalogs.reasons,
        results: allCatalogs.results,
        gestionTypes: allCatalogs.gestionTypes,
      )),
      dioError: (error, userMessage, technicalMessage, module, file, line, stackTrace) {
        emit(CatalogsState.error(
          message: userMessage ?? 'Error cargando catálogos',
          code: error.response?.statusCode.toString() ?? "unknown",
          technicalDetails: technicalMessage,
        ));
      },
      generalError: (userMessage, technicalMessage, module, file, line, stackTrace) {
        emit(CatalogsState.error(
          message: userMessage,
          code: "general_error",
          technicalDetails: technicalMessage,
        ));
      },
    );
  }
}
```

## 3. Reglas de Verificacion para el Auditor (BLoC Edition)

1. Exhaustividad de Errores: El Auditor debe rechazar cualquier BLoC que use .maybeWhen() para el DataResult si esto implica ignorar los errores de red (dioError). Se exige el manejo explícito de ambos tipos de error.
2. Inyeccion por Constructor: Verificar que no existan llamadas a GetIt.I<UseCase>() dentro de los metodos del BLoC. Todo debe entrar por constructor.
3. Uso de Extensiones: Si la UI realiza validaciones de tipo (ej: state is Success), el Auditor debe sugerir el uso de las extensiones definidas en el Punto 1.5 para limpiar la vista.
4. Cero Logica de Negocio: Si el BLoC contiene calculos matematicos, transformaciones complejas de strings o validaciones que no sean de UI, el Auditor debe ordenar mover esa logica al Entity o a un UseCase.