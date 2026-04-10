# Skill: Creador de Casos de Uso Quirúrgico

## 🎯 Objetivo
Añadir un nuevo caso de uso a un feature existente siguiendo estrictamente el flujo de Clean Architecture. Esta skill se encarga de actualizar todas las capas (Domain, Data y DI) para integrar la nueva funcionalidad de forma limpia.

## 🔗 Estándares de Implementación
Al agregar un caso de uso, el agente debe basarse en el `BLUEPRINT.md` para:
- **Estructura de UseCase**: Implementar `UseCaseWithParams` o `UseCaseWithoutParams`.
- **Manejo de Errores**: Usar obligatoriamente `DataStateFactory` en la implementación del repositorio.
- **Retrofit**: Actualizar el `DataSource` con las anotaciones correctas.

## ⚡ Disparador Interactivo
Si el usuario escribe **"Generaré un caso de uso"** o **"Añadir nuevo caso de uso"**, el agente debe responder EXACTAMENTE:

> "Claro, para generar un feature óptimo necesito los siguientes datos:
> 
> - **Feature**: [Nombre]
> - **Caso de Uso**: [Nombre] | Path: [Ruta] | Método: [GET/POST]
> - **Body**: [JSON de parámetros]
> - **Observaciones**: [Ej: Monto * 100, Split de string, etc.]
> - **Response**: [Estructura del JSON esperado]"

## 🛠️ Pasos de Ejecución (Flujo de Trabajo)
Una vez recibidos los datos, el agente debe realizar lo siguiente:

1. **Capa Domain**:
    - Agregar el método abstracto a la interfaz del `Repository` del feature.
    - Crear el archivo del nuevo `UseCase` y su clase de `Params` correspondiente.
2. **Capa Data**:
    - Actualizar el `DataSource` (Retrofit) con el nuevo endpoint, método HTTP y parámetros.
    - Crear o actualizar el `Model` si la respuesta de la API requiere mapeos especiales (ej. splits de strings).
    - Implementar el método en el `RepositoryImpl` usando bloques `try-catch` con `DataStateFactory`.
3. **Inyección de Dependencias**:
    - Registrar el nuevo `UseCase` en el archivo `{feature}_dependencies.dart` del injector.
4. **Mantenimiento**:
    - Actualizar los **Barrel Exports** (`.dart`) si se crearon archivos nuevos.

## 📝 Reglas de Transformación
- Aplicar cualquier lógica descrita en el campo **Observaciones** (conversión de tipos, cálculos, etc.).
- Mantener el principio de **Hard-Coding** para parámetros marcados como "SIEMPRE".
- Asegurar la **Reutilización de Parámetros** para evitar redundancia en los inputs del UseCase.