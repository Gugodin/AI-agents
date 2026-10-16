---
name: context-guardian
description: Disciplina de ciclo de vida del contexto de agentes. Use al ARRANCAR cualquier trabajo (verificar handoff previo antes de empezar — handoff-first) y durante tareas largas (detectar agotamiento de contexto y cortar con handoff). Cubre reglas de arranque handoff-first, heurísticas de agotamiento y protocolo de corte.
---

# Skill: Context Guardian — El Guardián del Contexto

Este manual define la disciplina de contexto para todos los agentes del equipo. Dos frentes: **arrancar bien** (leer el handoff antes de trabajar) y **cortar a tiempo** (parar, handoff y sesión nueva antes de degradarse). Un agente degradado produce bugs que cuestan más que el corte. Un agente sin contexto previo repite errores que ya se documentaron.

## 1. Regla de arranque (handoff-first)

Antes de aceptar CUALQUIER tarea:

1. **Sesión principal:** buscar el handoff más reciente del sprint activo (`docs/sprints/mvp-N/fase-NN-slug/handoffs/`, el más reciente del sprint). Si existe → leerlo y extraer: objetivo, estado, rama activa, qué quedó a medias, intentos fallidos, próximos pasos. Si no existe → arrancar de `AGENTS.md` + `plan.md` del sprint.
2. **Subagentes:** si el prompt menciona estado previo, un handoff o una continuación y NO incluye su contenido → PEDIRLO antes de trabajar. Nunca asumir el estado previo. (Extiende el patrón anti-alucinación del skill `project-init` §5.)
3. Tras leer el handoff: si algo del handoff contradice lo que ves en el repo, lo que ves en el repo manda (el handoff puede estar desactualizado) — anotar la divergencia y corregirla en el próximo handoff.
4. Si no hay handoff Y no hay plan de sprint → es una sesión fundacional: trabajar normal, pero el handoff final es OBLIGATORIO (§3).

## 2. Detección de agotamiento (heurísticas)

Un agente no ve su contador de tokens, pero estas señales son medibles:

| Señal | Umbral |
|---|---|
| Tool calls acumuladas en la tarea actual | > 40 |
| Archivos escritos en la tarea | > 10 |
| Ciclos fix→verificación fallidos sobre el mismo problema | ≥ 3 |
| Re-leer archivos que ya se leyeron (memoria de trabajo perdida) | cualquier aparición |
| No poder enumerar los requisitos del prompt original sin re-leerlo | señal crítica |
| Plan de `todowrite` con >60% pendiente + sensación de contexto pesado | combinar con otra señal |

Cualquier señal crítica, o dos señales normales simultáneas → agotado. El conteo de tool calls y archivos se trackea mentalmente desde el inicio de la tarea; dudar del conteo también es señal.

## 3. Protocolo de corte (al detectar agotamiento)

1. **No aceptar trabajo nuevo.** Anunciar el corte de inmediato.
2. **Completar solo el paso atómico en curso.** Nunca cortar un archivo a medias: terminar la unidad (archivo/edición) y detenerse.
3. **Generar el handoff de sesión** (tipo 1 del skill `sprint-handoff`):
   - Cargar el skill `sprint-handoff` ANTES de escribir (obligatorio — garantiza template y ubicación correctos).
    - Ubicación: `docs/sprints/mvp-N/<sprint-activo>/handoffs/handoff-YYYY-MM-DD-<feature>.md`.
   - Contenido mínimo: objetivo, estado (hecho vs verificado), rama activa y último commit, qué quedó a medias con archivo/línea, intentos fallidos, próximos pasos exactos.
4. **Commit del handoff** (si se trabaja en un repo): `docs(handoff): <feature> — corte de sesión`. En la rama de la feature (o `development` si es sesión de infraestructura).
5. **Recomendar al usuario:** "abre una sesión nueva — el handoff está listo con el contexto completo para retomar".
6. **Subagentes sin `Task`:** el handoff escrito en disco ES el reporte; el mensaje final de vuelta incluye su ruta y un resumen de 5 líneas.

## 4. Reglas de honestidad

- **Mejor un corte a tiempo que trabajo degradado.** El costo de reabrir una sesión es minutos; el costo de un bug producido por un agente agotado puede ser horas.
- **Nunca seguir "para terminar nomás"** si las señales están presentes. Los últimos pasos degradados son los más peligrosos: parecen progreso.
- **El handoff no es opcional.** Toda sesión que tocó algo debe dejar handoff — incluso las sesiones cortas y las fundacionales. La próxima sesión no debería tener que re-descubrir nada.
- **Contar las señales en voz alta:** al notar una señal, decirla ("ya llevo 40+ tool calls"). El conteo explícito evita el sesgo de "creo que todavía puedo".

## 5. Integración con el equipo

| Situación | Comportamiento |
|---|---|
| `technical-lead` orquestando y un subagent reporta agotamiento | No relanzar al subagent con la misma tarea: pedir handoff, integrar el contexto y decidir si la sesión continúa o corta |
| `merge-manager` agotado a mitad de protocolo | Abortar el protocolo (estado limpio, §4 del skill merge-manager) y handoff — jamás dejar un merge a medias |
| `quality-auditor` agotado a mitad de auditoría | Emitir `REQUEST CHANGES` con "auditoría incompleta: <archivos faltantes>" — jamás APPROVED sin evidencia completa |
| Cualquier agente | El handoff escrito directo sigue el template de `sprint-handoff`; `sprint-planner` sigue siendo el escritor por defecto cuando existe `Task` disponible |
