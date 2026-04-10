# Index: AI Agents Standards & Skills (v2.0)

Este repositorio centraliza los estándares de arquitectura Flutter y las habilidades operativas de Javier. Este ecosistema está diseñado para que cualquier Agente de IA genere código de nivel senior, siguiendo un flujo de trabajo profesional, modular y libre de errores de inicialización.

---

## 📚 Documentación Base (El "Saber")

- **[Blueprint de Arquitectura](./BLUEPRINT.md)**: La fuente única de verdad. Define el stack tecnológico, el ciclo de vida crítico de la app, el manejo de estados granulares y los patrones de persistencia mediante Mixins.

---

## 🛠️ Skills Operativas (El "Hacer")

Utiliza estas habilidades según la tarea solicitada, respetando siempre las versiones v2.x:

### 1. [Setup de Proyecto (v2.3)](./setup_project_clean.md)
- **Comando**: "Inicializar Proyecto".
- **Función**: Configura la base del proyecto (Dart + Nativo). Asegura el orden correcto en `main.dart`, la jerarquía del `ToastificationWrapper` y los permisos en Android/iOS.

### 2. [Generador de Features (v2.2)](./feature_creator_skill.md)
- **Comando**: "Generaré un feature" o "Generaremos un nuevo feature".
- **Función**: Crea módulos completos (Domain, Data, Presentation). Implementa mapeo de modelos con `toEntity()`, estados de BLoC granulares y preservación de datos para Skeletons.

### 3. [Creador de Casos de Uso (v2.2)](./use_case_creator_skill.md)
- **Comando**: "Generaré un caso de uso" o "Añadir nuevo caso de uso".
- **Función**: Integración quirúrgica de funcionalidades en features existentes. Actualiza Repositorios y DataSources (Retrofit) y gestiona la persistencia local mediante Mixins.

---

## 🚦 Reglas de Oro para el Agente

1. **Prioridad de Inicialización**: Nunca alteres el orden del `main.dart`; el `SharedPreferenceHelper` debe inicializarse antes del `runApp`.
2. **Fidelidad y Transformación**: El mapeo estándar debe ser fiel al JSON de respuesta. Las transformaciones complejas solo se aplican si se detallan en "Observaciones".
3. **Calidad Visual**: Los estados de BLoC deben permitir la preservación de datos para evitar pantallas en blanco durante cargas o errores.
4. **Nomenclatura e Imports**: Usa siempre el **Nombre del Proyecto** proporcionado para generar los imports de paquete correctamente.