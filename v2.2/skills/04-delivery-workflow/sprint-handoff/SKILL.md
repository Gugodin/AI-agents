---
name: sprint-handoff
description: Plantillas y reglas para crear handoffs y planes de sprint en docs/sprints/. Use cuando el usuario pida "genera el handoff", "crea el handoff de sesión", "cierra la sesión", "handoff de consolidación", "handoff de cierre", o al abrir un sprint y crear su plan.md. Define los 4 tipos de handoff y el template de plan de sprint.
---

# Skill: Sprint Handoff — La Memoria del Proyecto

Este manual define la estructura obligatoria de los handoffs y planes de sprint. Un handoff es un documento que permite retomar el trabajo **sin contexto previo**: todo lo relevante de una sesión, condensado. Si un agente nuevo necesita preguntar algo que la conversación anterior sabía, el handoff falló.

## 0. Reglas de estructura

- **Ubicación:** `<proyecto>/docs/sprints/mvp-N/fase-NN-slug/handoffs/` (handoffs) y `<proyecto>/docs/sprints/mvp-N/fase-NN-slug/plan.md` (planes).
- **Naming handoff:** `handoff-YYYY-MM-DD-<nombre>.md` — nombre = feature o tema (ej. `handoff-2026-10-12-sistema-sprints.md`).
- **Escritor por defecto:** el agente `sprint-planner`. Cualquier otro agente puede escribir un handoff de sesión si está agotado o no tiene acceso a `Task` (regla de robustez de `context-guardian`) — SIEMPRE siguiendo estos templates.
- **Sprint activo:** se descubre leyendo `docs/sprints/mvp-N/*/plan.md` — el MVP activo es el primero (en orden numérico) con criterios de done sin cumplidos en su doc de iteración; dentro de ese MVP, el sprint activo es el primero con criterios de done del `plan.md` sin cumplir.

## 1. Invariantes de TODO handoff

1. **Autosuficiencia:** el handoff ES la fuente. Nunca referirse a "la conversación" o "lo anterior" como fuente de información.
2. **Precisión:** fechas, hashes, paths exactos, nombres de rama. Nada de "el archivo de ayer" o "ese bug".
3. **Accionabilidad:** los próximos pasos son secuencias ejecutables, no intenciones ("correr X y verificar Y", no "mejorar Z").
4. **Honestidad:** los intentos fallidos se documentan para no repetirlos. Un handoff que oculta fracasos genera trabajo repetido.
5. **Scope:** un handoff pertenece a UN sprint y a UN tema/feature. Si la sesión tocó varios temas, un handoff por tema.

## 2. Tipos de handoff

| Tipo | Cuándo | Quién lo dispara |
|---|---|---|
| 1. Sesión / parcial | Al pausar una feature sin cerrarla (fin de sesión o agotamiento de contexto) | Cualquier agente (regla context-guardian) |
| 2. Consolidación | Tras un feature-merge exitoso | `merge-manager` → `sprint-planner` |
| 3. Cierre de sprint | Cuando todas las features del plan.md están merged y done cumplido | `sprint-planner` |
| 4. Cierre de MVP | Tras un release-merge (development → main + tag) | `merge-manager` → `sprint-planner` |

## 3. Template — plan.md de sprint (apertura)

```markdown
# Sprint: fase-NN-slug — <Nombre>

> **<Sprint activo | Próximo sprint>.** Objetivo: <una frase>. Origen: <fase del doc de iteración + flujos>.

## Objetivo
<Qué logra este sprint, en lenguaje de negocio (2-4 líneas).>

## Features y ramas previstas
| Rama | Tarea | Referencia |
|---|---|---|
| `feature/<name>` | <tarea> | RF-NN; Flujo NN |

<Nota de paralelizabilidad: qué puede correr en sesiones paralelas, qué es secuencial y por qué.>

## Dependencias
- **Sprint anterior:** <cuál debe estar cerrado>.
- **Externas (usuario):** <config de consolas, credenciales, etc.>.

## Criterios de done
- [ ] <verificación concreta, medible>
- [ ] Auditoría `quality-auditor` → APPROVED
- [ ] Todas las ramas merged a `development` vía `merge-manager`

## Handoffs generados
- <link a cada handoff con una línea de resumen> (o "ninguno aún")
```

## 4. Template — Tipo 1: handoff de sesión/parcial

```markdown
# Handoff de Sesión — <feature> (YYYY-MM-DD)

> Estado parcial: la feature NO está cerrada. Rama activa: `feature/<name>`. Sprint: fase-NN-slug.

## 1. Objetivo de la sesión
<Qué se buscaba lograr (1-3 líneas).>

## 2. Estado actual
<Qué está hecho y VERIFICADO. Distinguir "escrito" de "verificado".>

## 3. Rama y commits
- Rama activa: `feature/<name>` (nace de `development`)
- Último commit: `<hash> <mensaje>`

## 4. Qué quedó a medias
<Archivo + qué falta exactamente. Ej: "`lib/features/auth/domain/repositories/auth_repository.dart` — contrato escrito, falta `loginWithPhone`.">

## 5. Intentos fallidos
<Camino que ya se probó y NO funcionó, con el porqué. Para no repetirlo.>

## 6. Próximos pasos exactos
1. <paso accionable>
2. <paso accionable>

## 7. Referencias
- Plan del sprint: [plan.md](../plan.md)
- <docs relevantes con paths exactos>
```

## 5. Template — Tipo 2: handoff de consolidación

```markdown
# Handoff de Consolidación — <feature> (YYYY-MM-DD)

> Merge de `feature/<name>` a `development`. Merge commit: `<hash>`. Sprint: fase-NN-slug.

## 1. Qué se integró
<Resumen de la feature + RF/Flujo que implementa (referencia a docs).>

## 2. Commits integrados
<Lista `hash mensaje`.>

## 3. Archivos modificados/creados
<Por capa (domain/data/presentation/infra).>

## 4. Gates corridos
<analyze/format/test/build: resultado de cada uno. Gaps registrados (ej: "sin tests aún").>

## 5. Conflictos resueltos
<Si no hubo: "ninguno". Si hubo: archivo, tipo (trivial/semántico), resolución.>

## 6. Estado del sprint
<Progreso contra plan.md: qué quedó done, qué falta para cerrar el sprint.>

## 7. Ramas activas restantes
<Otras feature/* en vuelo, con su estado de una línea (o "ninguna").>
```

## 6. Template — Tipo 3: handoff de cierre de sprint

```markdown
# Handoff de Cierre — Sprint fase-NN-slug (YYYY-MM-DD)

> Sprint cerrado: todas las features merged, criterios de done cumplidos.

## 1. Resumen del sprint
<Qué se construyó en lenguaje de negocio.>

## 2. Features integradas
<Rama → merge hash → handoff de consolidación (link).>

## 3. Criterios de done verificados
<El checklist de plan.md, marcado.>

## 4. Pendientes heredados
<Qué hereda el siguiente sprint (deuda, gaps, ramas sin cerrar — o "ninguno").>

## 5. Siguiente sprint
<Link al plan.md del siguiente sprint (creado JIT al cerrar este).>
```

## 7. Template — Tipo 4: handoff de cierre de MVP

```markdown
# Handoff de Cierre — MVP <N> (YYYY-MM-DD)

> Release: `development` → `main`. Tag: `vX.0.0`. Merge commit: `<hash>`.

## 1. Resumen del MVP
<Qué se lanzó y qué valida (lenguaje de negocio).>

## 2. Criterios de salida verificados
<Checklist de docs/iteraciones/MVP-NN con evidencia.>

## 3. Tags y hashes
- Merge a main: `<hash>`
- Tag: `vX.0.0`

## 4. Próximo MVP
<Estado, primer sprint sugerido, dependencias externas pendientes.>
```

## 8. Regla de descubrimiento para el retome

Toda sesión nueva que abre trabajo en un proyecto con sprints debe leer, en orden: `AGENTS.md` → `docs/sprints/README.md` → `plan.md` del sprint activo (en `docs/sprints/mvp-N/<sprint>/plan.md`) → handoff más reciente del sprint activo. El handoff más reciente manda sobre el plan.md si difieren (el handoff refleja realidad más fresca; actualizar plan.md es tarea de sprint-planner).
