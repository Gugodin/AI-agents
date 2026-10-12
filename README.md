# AI-Agents — El Equipo de Desarrollo Flutter

> Estamos armando un **equipo de trabajo de agentes de IA** especializado en el desarrollo de aplicaciones móviles con Flutter. No es un set de scripts: es un equipo con roles, responsabilidades, disciplina y flujo de entrega — los mismos que tendría un equipo humano de desarrollo, ejecutados por agentes que colaboran entre sí.

## La visión

Construir apps móviles de producto (no demos) usando agentes como si fueran los miembros de un equipo: un líder técnico que orquesta, desarrolladores que escriben código, un diseñador UI/UX que construye interfaces, un QA que audita estándares, un integrador que maneja ramas y releases, y un planificador que preserva la memoria del proyecto entre sesiones.

Cada agente tiene un rol definido, un manual técnico (`SKILL.md`) que sigue sin desviarse, y restricciones innegociables. El trabajo fluye por **Git Flow simplificado** (`main` → `development` → `feature/*`) y por **sprints documentados con handoffs** — para que ninguna sesión pierda el hilo de lo que ya se construyó.

## El equipo

| Rol | Agente | Responsabilidad |
|-----|--------|-----------------|
| Líder Técnico / Orquestador | `technical-lead` | Analiza requerimientos, planifica y coordina al equipo. Dueño de la inyección de dependencias. |
| Desarrollador de Código | `feature-developer` | Genera código de Dominio, Datos y BLoC bajo Clean Architecture. |
| Diseñador UI/UX | `ui-ux-craft` | Construye interfaces modulares y reactivas consumiendo los estados del BLoC. |
| QA / Auditor | `quality-auditor` | Audita el código contra los estándares. Veredicto `APPROVED` / `REQUEST CHANGES`. |
| Integrador / Release Manager | `merge-manager` | Protocolos de integración (`feature-merge` y `release-merge`), gate checks, resolución de conflictos. |
| Planificador / Cronista | `sprint-planner` | Planes de sprint, los 4 tipos de handoff, memoria del proyecto entre sesiones. |

El equipo se completa con tres **disciplinas transversales** (skills): `merge-manager` (protocolo), `sprint-handoff` (templates) y `context-guardian` (disciplina de contexto: handoff-first al arranque y corte con handoff ante agotamiento).

## Objetivos del equipo

### Del equipo como equipo

- **Autonomía controlada.** Los agentes trabajan solos dentro de sus manuales; el usuario solo aprueba decisiones de producto y conflictos semánticos.
- **Calidad como gate innegociable.** Nada se integra sin auditoría aprobada.
- **Flujo de entrega ordenado.** Git Flow simplificado (`main` → `development` → `feature/*`), merges con protocolo, releases etiquetados `vX.0.0`.
- **Memoria del proyecto.** Todo lo relevante se documenta en `plan.md` y handoffs — ninguna sesión arranca sin contexto previo ni termina sin dejar handoff.
- **Disciplina de contexto.** Todo agente arranca leyendo el handoff previo y corta con handoff al detectar agotamiento. Mejor un corte a tiempo que trabajo degradado.
- **Portabilidad al portafolio.** El sistema se diseña para servir a múltiples apps del portafolio (no solo una), con skills reutilizables.

### Del producto (portafolio SFT.Tec)

El equipo desarrolla las apps del portafolio **SFT.Tec** — tres apps Flutter independientes que comparten infraestructura y filosofía:

| App | Rubro | Estado |
|-----|-------|--------|
| **Tlaxi** | Tortillerías | En desarrollo — MVP 1 |
| **Stoki** | Tiendas de abarrotes | Documentada, pre-desarrollo |
| **Fuudi** | Restaurantes pequeños | Pendiente de documentar |

El orden de desarrollo lo define **ADR-009**: Tlaxi primero, su repo es el template de facto del portafolio. Stoki y Fuudi arrancan código solo cuando Tlaxi esté estable y monetizando.

## Evolución del sistema

| Versión | Qué aporta | Estado |
|---------|-----------|--------|
| `v1.0` | Primera iteración de skills monolíticas y agentes base. | Legado — conservada como referencia. |
| `v2.0` | Equipo modular: 4 agentes + 20 skills categorizadas (`00-core-rules`, `01-project-setup`, `02-feature-factory`, `03-quality-assurance`). Setup con symlinks. | Superada. |
| **`v2.1`** | **Equipo completo**: +`merge-manager`, +`sprint-planner`, +disciplina `context-guardian`. Flujo Git Flow + sprints con handoffs. Carpeta `04-delivery-workflow`. Carpeta interna normalizada a `skills/`. | **Vigente** |

## Setup → [Ver documentación de v2.1](./v2.1/README.md)

La versión vigente es `v2.1/`. Ahí está el README con la estructura detallada, comandos de symlinks, uso del equipo y mantenimiento.
