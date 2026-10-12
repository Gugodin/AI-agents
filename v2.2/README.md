# AI Agents v2.2

Repositorio con el **equipo completo** de agentes de IA especializados en la Arquitectura v2.2 para proyectos Flutter con Clean Architecture. Estos agentes están diseñados para trabajar dentro de **OpenCode** y colaborar entre sí para generar código de alta calidad de forma autónoma.

## Novedades v2.2

Respecto a `v2.1`:

- **1 agente nuevo** que cierra el ciclo de proyecto nuevo:
  - `interviewer` — primer agente del ciclo. Recaba información del dominio con el usuario usando entrevista estructurada por árbol de decisiones y genera la documentación completa del proyecto (`docs/` + `AGENTS.md`).
- **2 skills nuevas**:
  - `interviewer` (en `01-project-setup/`) — protocolo completo de descubrimiento: detección de armazón de portafolio, rondas con frontera y respuestas recomendadas, hechos-vs-decisiones, cierre con entendimiento compartido, generación de docs. Mecánica adaptada de grilling.
  - `grilling` (en `04-delivery-workflow/`) — skill externa de Matt Pocock (MIT), instalada tal cual. Refinamiento implacable de planes, decisiones o ideas. Útil para stress-test de ADRs, features grandes y planes de arquitectura.
- **Bump de versión**: todos los agentes dicen `v2.2` en su descripción y cuerpo.

## El Equipo

| Agente | Rol | Modo |
|--------|-----|------|
| `interviewer` | Entrevistador de producto. Primer agente del ciclo de proyecto nuevo. | `all` (primary + subagente) |
| `technical-lead` | Orquestador principal. Analiza requerimientos, planifica y coordina al equipo completo. También gestiona la inyección de dependencias. | `all` (primary + subagente) |
| `feature-developer` | Genera el código de Dominio, Datos y BLoC siguiendo Clean Architecture. | `subagente` |
| `quality-auditor` | Audita el código generado y emite veredicto `APPROVED` o `REQUEST CHANGES`. | `subagente` |
| `ui-ux-craft` | Construye interfaces Flutter modulares consumiendo los estados del BLoC. | `subagente` |
| `merge-manager` | Integra feature branches a `development` (feature-merge) y `development` a `main` con tag `vX.0.0` (release-merge). | `subagente` |
| `sprint-planner` | Crea `plan.md` al abrir sprints y materializa los 4 tipos de handoff (sesión, consolidación, cierre de sprint, cierre de MVP). | `subagente` |

## Estructura del Repositorio

```
v2.2/
├── agents/                        # Definiciones de agentes (formato OpenCode Markdown)
│   ├── interviewer.md
│   ├── technical-lead.md
│   ├── feature-developer.md
│   ├── quality-auditor.md
│   ├── ui-ux-craft.md
│   ├── merge-manager.md
│   └── sprint-planner.md
└── skills/                        # Manuales de conocimiento (Skills de OpenCode)
    ├── 00-core-rules/             # Reglas base innegociables
    │   ├── blueprint-foundation/
    │   ├── data-state-standard/
    │   ├── network-standards/
    │   └── use-case-standard/
    ├── 01-project-setup/          # Configuración de proyectos nuevos
    │   ├── interviewer/           # ← NUEVO v2.2
    │   ├── native-configuration/
    │   ├── project-init/
    │   └── scaffolding-dart/
    ├── 02-feature-factory/        # Manuales por capa de Clean Architecture
    │   ├── data/
    │   │   ├── data-model-specialist/
    │   │   ├── data-repository-specialist/
    │   │   └── data-source-addition/
    │   ├── dependency-manager/
    │   ├── domain/
    │   │   ├── domain-entity-specialist/
    │   │   ├── domain-repository-specialist/
    │   │   └── use-case-addition/
    │   └── presentation/
    │       ├── bloc-specialist/
    │       ├── flutter-gen-specialist/
    │       ├── i18n-specialist/
    │       ├── router-specialist/
    │       └── ui-specialist/
    ├── 03-quality-assurance/
    │   └── code-reviewer/
    └── 04-delivery-workflow/      # Flujo de entrega: git, sprints, handoffs, contexto, refinamiento
        ├── context-guardian/
        ├── grilling/              # ← NUEVO v2.2 (externa, MIT)
        ├── merge-manager/
        └── sprint-handoff/
```

## Configuración en una Máquina Nueva

> Sigue estos pasos en orden para tener el equipo funcionando en cualquier máquina con OpenCode instalado.

### Requisitos Previos
- [OpenCode](https://opencode.ai) instalado y configurado con un proveedor de IA.
- Git instalado.

### Paso 1: Clonar el Repositorio

```bash
# Clona el repositorio en la ubicación que prefieras
git clone https://github.com/Gugodin/AI-agents.git ~/my-ai-agents
```

> Anota la ruta donde clonaste el repo. La necesitarás en el Paso 3.
> En este ejemplo usaremos `~/my-ai-agents/v2.2` como ruta base.

### Paso 2: Crear los Directorios de OpenCode

```bash
mkdir -p ~/.config/opencode/agents
mkdir -p ~/.config/opencode/skills
```

### Paso 3: Vincular Agentes con Symlinks

Los symlinks conectan los archivos del repositorio directamente con OpenCode. Cuando actualices el repo (git pull), los cambios se reflejan automáticamente sin reconfiguraciones adicionales.

```bash
# Reemplaza RUTA_BASE con la ruta absoluta donde clonaste el repo
# Ejemplo: /Users/tu_usuario/my-ai-agents/v2.2
RUTA_BASE="$HOME/my-ai-agents/v2.2"

# Vincular los 7 agentes
ln -sf "$RUTA_BASE/agents/interviewer.md"        ~/.config/opencode/agents/interviewer.md
ln -sf "$RUTA_BASE/agents/technical-lead.md"     ~/.config/opencode/agents/technical-lead.md
ln -sf "$RUTA_BASE/agents/feature-developer.md"  ~/.config/opencode/agents/feature-developer.md
ln -sf "$RUTA_BASE/agents/quality-auditor.md"    ~/.config/opencode/agents/quality-auditor.md
ln -sf "$RUTA_BASE/agents/ui-ux-craft.md"        ~/.config/opencode/agents/ui-ux-craft.md
ln -sf "$RUTA_BASE/agents/merge-manager.md"      ~/.config/opencode/agents/merge-manager.md
ln -sf "$RUTA_BASE/agents/sprint-planner.md"     ~/.config/opencode/agents/sprint-planner.md
```

### Paso 4: Vincular Skills con Symlinks

```bash
RUTA_BASE="$HOME/my-ai-agents/v2.2"
SKILLS_DIR="$RUTA_BASE/skills"

# Core Rules (00)
ln -sf "$SKILLS_DIR/00-core-rules/blueprint-foundation"      ~/.config/opencode/skills/blueprint-foundation
ln -sf "$SKILLS_DIR/00-core-rules/data-state-standard"       ~/.config/opencode/skills/data-state-standard
ln -sf "$SKILLS_DIR/00-core-rules/network-standards"         ~/.config/opencode/skills/network-standards
ln -sf "$SKILLS_DIR/00-core-rules/use-case-standard"         ~/.config/opencode/skills/use-case-standard

# Project Setup (01)
ln -sf "$SKILLS_DIR/01-project-setup/interviewer"            ~/.config/opencode/skills/interviewer
ln -sf "$SKILLS_DIR/01-project-setup/native-configuration"   ~/.config/opencode/skills/native-configuration
ln -sf "$SKILLS_DIR/01-project-setup/project-init"           ~/.config/opencode/skills/project-init
ln -sf "$SKILLS_DIR/01-project-setup/scaffolding-dart"       ~/.config/opencode/skills/scaffolding-dart

# Feature Factory — Data (02)
ln -sf "$SKILLS_DIR/02-feature-factory/data/data-model-specialist"      ~/.config/opencode/skills/data-model-specialist
ln -sf "$SKILLS_DIR/02-feature-factory/data/data-repository-specialist" ~/.config/opencode/skills/data-repository-specialist
ln -sf "$SKILLS_DIR/02-feature-factory/data/data-source-addition"       ~/.config/opencode/skills/data-source-addition

# Feature Factory — Domain (02)
ln -sf "$SKILLS_DIR/02-feature-factory/domain/domain-entity-specialist"     ~/.config/opencode/skills/domain-entity-specialist
ln -sf "$SKILLS_DIR/02-feature-factory/domain/domain-repository-specialist" ~/.config/opencode/skills/domain-repository-specialist
ln -sf "$SKILLS_DIR/02-feature-factory/domain/use-case-addition"            ~/.config/opencode/skills/use-case-addition

# Feature Factory — Presentation (02)
ln -sf "$SKILLS_DIR/02-feature-factory/presentation/bloc-specialist"    ~/.config/opencode/skills/bloc-specialist
ln -sf "$SKILLS_DIR/02-feature-factory/presentation/flutter-gen-specialist" ~/.config/opencode/skills/flutter-gen-specialist
ln -sf "$SKILLS_DIR/02-feature-factory/presentation/i18n-specialist"    ~/.config/opencode/skills/i18n-specialist
ln -sf "$SKILLS_DIR/02-feature-factory/presentation/router-specialist"  ~/.config/opencode/skills/router-specialist
ln -sf "$SKILLS_DIR/02-feature-factory/presentation/ui-specialist"      ~/.config/opencode/skills/ui-specialist
ln -sf "$SKILLS_DIR/02-feature-factory/dependency-manager"              ~/.config/opencode/skills/dependency-manager

# Quality Assurance (03)
ln -sf "$SKILLS_DIR/03-quality-assurance/code-reviewer" ~/.config/opencode/skills/code-reviewer

# Delivery Workflow (04)
ln -sf "$SKILLS_DIR/04-delivery-workflow/context-guardian" ~/.config/opencode/skills/context-guardian
ln -sf "$SKILLS_DIR/04-delivery-workflow/grilling"         ~/.config/opencode/skills/grilling
ln -sf "$SKILLS_DIR/04-delivery-workflow/merge-manager"    ~/.config/opencode/skills/merge-manager
ln -sf "$SKILLS_DIR/04-delivery-workflow/sprint-handoff"   ~/.config/opencode/skills/sprint-handoff
```

### Paso 5: Verificar la Configuración

```bash
# Verificar que los agentes están vinculados correctamente
ls -la ~/.config/opencode/agents/

# Verificar que los skills están vinculados correctamente
ls -la ~/.config/opencode/skills/
```

Deberías ver los **7 agentes** y los **25 skills** listados como symlinks (`->` apuntando al repositorio).

### Paso 6: Reiniciar OpenCode

Cierra y vuelve a abrir OpenCode. Los nuevos agentes estarán disponibles:
- Con la tecla `Tab` para ciclar entre agentes primarios (verás `interviewer` y `technical-lead`).
- Con `@` para mencionar cualquier subagente directamente en el chat.

## Uso del Equipo

### Ciclo Completo de un Proyecto Nuevo (Recomendado)

El ciclo va desde la idea hasta la feature integrada:

```
1. interviewer        → documenta el proyecto (docs/ + AGENTS.md)
2. project-init       → setup técnico del repo Flutter (skills 01-project-setup)
3. technical-lead     → orquesta features
4. feature-developer  → código de dominio/datos/BLoC
5. ui-ux-craft        → páginas, vistas, widgets
6. quality-auditor    → auditoría contra estándares
7. merge-manager      → integra a development
8. sprint-planner     → handoffs de consolidación
```

#### Paso 1: Arrancar un Proyecto Nuevo

Activa el agente `interviewer` con `Tab` (es primary — dialoga contigo directamente):

```
Quiero documentar una app nueva para el rubro de lavanderías de barrio.
```

El interviewer te entrevista en rondas, mapea tu árbol de decisiones, te da respuestas recomendadas, y al cerrar genera los `docs/` + `AGENTS.md` del proyecto.

#### Paso 2: Flujo de Feature

Una vez el proyecto está documentado, activa `technical-lead` con `Tab` y pide la feature completa:

```
Necesito crear la feature de inventario. El dominio son prendas por cliente con estado (recibida, en lavado, lista, entregada). El proyecto está en /ruta/al/proyecto.
```

El Technical Lead coordinará automáticamente a `feature-developer`, `quality-auditor` y `ui-ux-craft`.

#### Paso 3: Integración y Release

Cuando una feature esté lista:
```
@merge-manager Integra la rama feature/inventory a development con los gate checks.
```

Para cerrar un MVP y liberar a main:
```
@merge-manager Ejecuta el release-merge de development a main con tag v1.0.0.
```

#### Paso 4: Planificación de Sprints

Al iniciar un sprint o al cerrar una sesión:
```
@sprint-planner Genera el plan del sprint fase-01-auth con las features phone-auth y onboarding.
@sprint-planner Genera el handoff de consolidación tras el merge de feature/auth.
```

#### Paso 5: Stress-test de Planes/ADRs (skill `grilling`)

Para refinar cualquier plan o decisión antes de construir:
```
Estresa este ADR con la skill grilling — quiero asegurarme de que no hay suposiciones ocultas.
```

#### Invocación Directa de Subagentes

También puedes invocar subagentes directamente mencionándolos con `@`:
```
@quality-auditor Revisa los archivos de la feature inventory en lib/features/inventory/
@feature-developer Solo necesito el BLoC para la feature de perfil de usuario
```

## Instalación de Skills Externas (Terceros)

El equipo es extensible con skills de terceros (repos públicos como [`mattpocock/skills`](https://github.com/mattpocock/skills), licencia MIT/Apache, etc.). El proceso estándar en 5 pasos:

### Paso 1 — Evaluar la skill antes de instalar

- Lee su `SKILL.md` completo: ¿qué hace? ¿su formato cumple el estándar de OpenCode? (carpeta con el mismo nombre que el `name` del frontmatter, y `description` en tercera persona con las frases-disparador).
- **Licencia**: MIT/Apache = reutilizable con atribución. Sin archivo de licencia = "todos los derechos reservados" (no copiar).
- **Campos de otros runtimes**: los repos populares soportan varios agentes (Claude Code, OpenCode...). Campos como `disable-model-invocation: true` (de Claude Code) no son estándar de OpenCode — no los necesitamos. Instala solo lo que tenga frontmatter compatible.

### Paso 2 — Elegir el método de instalación

| Método | Cómo | Ventaja / Costo |
|---|---|---|
| **A. Al repo del equipo** (recomendado) | Copias la skill al repo `AI-agents` y luego a tu máquina | Control de versiones + equipo autocontenido |
| **B. Copia directa** | Copias la carpeta a `~/.config/opencode/skills/` | Simple / No se actualiza sola |
| **C. Symlink del repo externo clonado** | Clonas el repo externo en un lugar fijo → symlink de la skill | `git pull` externo actualiza / Depende del upstream |
| **D. `skills.paths` en opencode.json** | `"skills": { "paths": ["/ruta/al/repo/skills"] }` | Sin copiar nada / Carga TODO el directorio (incluye lo que no quieres) |

Recomendamos el **método A**: tus skills externas quedan congeladas en la versión que probaste y forman parte del equipo versionado. El upstream puede cambiar y romper tu setup.

### Paso 3 — Traer la skill con atribución

Copia el `SKILL.md` a la carpeta correspondiente del repo (ej. `v2.2/skills/04-delivery-workflow/<nueva-skill>/`) y añade **atribución** en el frontmatter:

```yaml
---
name: <nombre>
description: ...
license: MIT
metadata:
  source: https://github.com/<usuario>/<repo>/tree/main/...
  author: <Autor Original>
  frozen_date: "YYYY-MM-DD"
---
```

Documenta en el README del equipo (sección de changelog o de skills externas) la procedencia.

### Paso 4 — Reiniciar OpenCode (SIEMPRE)

Las skills se cargan al iniciar OpenCode — no hay hot-reload. La sesión actual no ve las skills nuevas; cierra y abre.

### Paso 5 — Verificar

Que la skill aparezca en las skills disponibles (el modelo las ve por sus `description`). Prueba real: invócala o usa una frase-disparador.

### Ejemplo: la skill `grilling` instalada en v2.2

`grilling` se instaló en v2.2 siguiendo exactamente este proceso: skill externa del repo de Matt Pocock, licencia MIT, atribuida en el frontmatter con `source`, `author` y `frozen_date`. Copiada a `v2.2/skills/04-delivery-workflow/grilling/` y vinculada con symlink.

## Mantenimiento y Actualizaciones

Para actualizar los agentes y skills en todas las máquinas:
```bash
# En la máquina donde tienes el repo
cd ~/my-ai-agents
git pull

# Los symlinks apuntan directamente al repo,
# por lo que los cambios son inmediatos en OpenCode.
# No es necesario repetir los pasos de configuración.
```

Para agregar un nuevo skill al repositorio:
1. Crea la carpeta en la sección correcta de `skills/` con un `SKILL.md` que tenga frontmatter (`name` y `description`).
2. Crea el symlink correspondiente en `~/.config/opencode/skills/` en todas las máquinas.
3. Agrega el comando de symlink a la sección de "Paso 4" de este README.
4. Haz commit y push al repositorio.
