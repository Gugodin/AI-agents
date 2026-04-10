# Index: AI Agents Standards & Skills

Este repositorio centraliza los estándares de arquitectura y las habilidades operativas para el desarrollo de aplicaciones Flutter de Javier. El objetivo es proporcionar un contexto unificado para que cualquier Agente de IA pueda generar código coherente, profesional y escalable.

---

## 📚 Documentación Base (El "Saber")

- **[Blueprint de Arquitectura](docs/BLUEPRINT.md)**: Es el estándar de oro y la fuente única de verdad. Define la estructura de carpetas, el stack tecnológico, el manejo de estados con `DataState`, y los patrones de inyección de dependencias.

---

## 🛠️ Skills Disponibles (El "Hacer")

Utiliza las siguientes habilidades según la tarea solicitada por el usuario:

### 1. [Setup de Proyecto](skills/setup_proyect_clean.md)
- **Comando**: "Inicializar Proyecto".
- **Función**: Configura la base del proyecto (carpetas, dependencias y Core Foundation) vinculando cada componente a los ejemplos del Blueprint.

### 2. [Generador de Features](skills/feature_creator_skill.md)
- **Comando**: "Generaré un feature" o "Generaremos un nuevo feature".
- **Función**: Crea un módulo completo desde cero (Domain, Data, Presentation) siguiendo el checklist de calidad y las reglas de transformación de datos solicitadas.

### 3. [Creador de Casos de Uso](skills/use_case_creator_skill.md)
- **Comando**: "Generaré un caso de uso" o "Añadir nuevo caso de uso".
- **Función**: Integra quirúrgicamente nuevas funcionalidades a un feature existente, actualizando el Repository, DataSource, Model y la inyección de dependencias.

---

## 🚦 Instrucciones para el Agente de IA

1. **Lectura Obligatoria**: Antes de realizar cualquier acción, debes leer el archivo `BLUEPRINT.md` para asimilar el uso de `DataState`, `DataResult`, `UseCase` y `DataStateFactory`.
2. **Interactividad**: Si el usuario solicita un Feature o un Caso de Uso, DEBES responder con el diálogo de recolección de datos definido en la skill correspondiente para asegurar un resultado óptimo.
3. **Manejo de Errores**: Nunca omitas el uso de las factorías de error en la capa de datos.
4. **Clean Code**: Mantén siempre la flexibilidad en la lógica de negocio pero rigidez absoluta en la estructura arquitectónica.