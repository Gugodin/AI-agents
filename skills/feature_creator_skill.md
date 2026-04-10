# Skill: Generador de Features Universal (Clean Architecture)

## 🎯 Objetivo
Generar un feature completo (Domain, Data, Presentation) siguiendo un estándar de código estricto, pero permitiendo flexibilidad total en la lógica de cada caso de uso.

## 🛠️ Reglas Arquitectónicas Absolutas
Para garantizar la consistencia, el agente DEBE seguir estas reglas sin excepción:

1. **Gestión de Errores (Obligatorio)**: En la capa de `Data` (`RepositoryImpl`), es obligatorio el uso de `DataState` junto con sus factorías (`DataStateFactory.fromDioException` y `DataStateFactory.fromException`) para el manejo de excepciones.
2. **Hard-Coding**: Si el usuario indica que un parámetro es "SIEMPRE" un valor, este debe quedar quemado en la implementación de la capa de Data, sin exponerlo al UseCase.
3. **Reutilización de Parámetros**: Si múltiples campos de la API requieren el mismo dato, el UseCase solo debe recibir un parámetro y el Repository se encarga de distribuirlo a los campos correspondientes del DataSource.
4. **Estructura de Feature**: Se deben crear las carpetas `data`, `domain` y `presentation`, cada una con sus respectivos archivos y Barrel Exports correspondientes.
5. **Inyección de Dependencias**: Crear el archivo `{feature}_dependencies.dart` registrando DataSources/Repos/UseCases como `factory` y BLoCs como `lazySingleton`.

## 🧠 Lógica de Negocio Flexible
A diferencia de las reglas arquitectónicas, la lógica de transformación de datos (como conversiones de montos, formateo de fechas o split de strings) se definirá **caso por caso** en el apartado de **Observaciones** del prompt. El agente debe ser capaz de traducir instrucciones lógicas (incluso fragmentos de código de otros lenguajes) a código Dart funcional dentro del `Model` o `RepositoryImpl`.

## ⚡ Disparador Interactivo
Si el usuario escribe **"Generaré un feature"** o **"Generaremos un nuevo feature"**, el agente debe responder EXACTAMENTE:

> "Claro, para generar un feature óptimo necesito los siguientes datos:
> 
> - **Feature**: [Nombre]
> - **Caso de Uso**: [Nombre] | Path: [Ruta] | Método: [GET/POST]
> - **Body**: [JSON de parámetros]
> - **Observaciones**: [Ej: Monto * 100, Split de string, etc.]
> - **Response**: [Estructura del JSON esperado]"

## ✅ Checklist de Salida
Cada feature generado debe cumplir con los puntos de la Sección 10 del Blueprint (Entity con mock, Repository interface e impl, UseCase con Params, BLoC con Freezed y registro en GetIt).