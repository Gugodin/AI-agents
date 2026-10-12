# AI Agents v2.1

Repositorio con el **equipo completo** de agentes de IA especializados en la Arquitectura v2.1 para proyectos Flutter con Clean Architecture. Estos agentes están diseñados para trabajar dentro de **OpenCode** y colaborar entre sí para generar código de alta calidad de forma autónoma.

## Novedades v2.1

Respecto a `v2.0`:

- **2 agentes nuevos** que completan el ciclo de entrega:
  - `merge-manager` — integrador de ramas bajo Git Flow (protocolos `feature-merge` y `release-merge` con gate checks).
  - `sprint-planner` — planificador de sprints y cronista de handoffs (escritor por defecto de la memoria del proyecto).
- **3 skills nuevas** en la categoría `04-delivery-workflow`:
  - `merge-manager` — protocolo detallado de integración.
  - `sprint-handoff` — templates de `plan.md` y los 4 tipos de handoff.
  - `context-guardian` — disciplina de contexto (handoff-first al arranque + corte con handoff ante agotamiento).
- **Carpeta interna normalizada**: `modular-skills-v2/` → `skills/` (la versión la da la carpeta padre, no la interna).
- **Bump de versión**: todos los agentes dicen `v2.1` en su descripción y cuerpo.
- **Todos los agentes cargan `context-guardian`** como skill obligatoria (disciplina transversal).

## El Equipo

| Agente | Rol | Modo |
|--------|-----|------|
| `technical-lead` | Orquestador principal. Analiza requerimientos, planifica y coordina al equipo completo. También gestiona la inyección de dependencias. | `all` (primary + subagente) |
| `feature-developer` | Genera el código de Dominio, Datos y BLoC siguiendo Clean Architecture. | `subagente` |
| `quality-auditor` | Audita el código generado y emite veredicto `APPROVED` o `REQUEST CHANGES`. | `subagente` |
| `ui-ux-craft` | Construye interfaces Flutter modulares consumiendo los estados del BLoC. | `subagente` |
| `merge-manager` | Integra feature branches a `development` (feature-merge) y `development` a `main` con tag `vX.0.0` (release-merge). | `subagente` |
| `sprint-planner` | Crea `plan.md` al abrir sprints y materializa los 4 tipos de handoff (sesión, consolidación, cierre de sprint, cierre de MVP). | `subagente` |

## Estructura del Repositorio

```
v2.1/
├── agents/                        # Definiciones de agentes (formato OpenCode Markdown)
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
    └── 04-delivery-workflow/      # Flujo de entrega: git, sprints, handoffs, contexto
        ├── merge-manager/
        ├── sprint-handoff/
        └── context-guardian/
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
> En este ejemplo usaremos `~/my-ai-agents/v2.1` como ruta base.

### Paso 2: Crear los Directorios de OpenCode

```bash
mkdir -p ~/.config/opencode/agents
mkdir -p ~/.config/opencode/skills
```

### Paso 3: Vincular Agentes con Symlinks

Los symlinks conectan los archivos del repositorio directamente con OpenCode. Cuando actualices el repo (git pull), los cambios se reflejan automáticamente sin reconfiguraciones adicionales.

```bash
# Reemplaza RUTA_BASE con la ruta absoluta donde clonaste el repo
# Ejemplo: /Users/tu_usuario/my-ai-agents/v2.1
RUTA_BASE="$HOME/my-ai-agents/v2.1"

# Vincular los 6 agentes
ln -sf "$RUTA_BASE/agents/technical-lead.md"    ~/.config/opencode/agents/technical-lead.md
ln -sf "$RUTA_BASE/agents/feature-developer.md" ~/.config/opencode/agents/feature-developer.md
ln -sf "$RUTA_BASE/agents/quality-auditor.md"   ~/.config/opencode/agents/quality-auditor.md
ln -sf "$RUTA_BASE/agents/ui-ux-craft.md"       ~/.config/opencode/agents/ui-ux-craft.md
ln -sf "$RUTA_BASE/agents/merge-manager.md"     ~/.config/opencode/agents/merge-manager.md
ln -sf "$RUTA_BASE/agents/sprint-planner.md"    ~/.config/opencode/agents/sprint-planner.md
```

### Paso 4: Vincular Skills con Symlinks

```bash
RUTA_BASE="$HOME/my-ai-agents/v2.1"
SKILLS_DIR="$RUTA_BASE/skills"

# Core Rules (00)
ln -sf "$SKILLS_DIR/00-core-rules/blueprint-foundation"      ~/.config/opencode/skills/blueprint-foundation
ln -sf "$SKILLS_DIR/00-core-rules/data-state-standard"       ~/.config/opencode/skills/data-state-standard
ln -sf "$SKILLS_DIR/00-core-rules/network-standards"         ~/.config/opencode/skills/network-standards
ln -sf "$SKILLS_DIR/00-core-rules/use-case-standard"         ~/.config/opencode/skills/use-case-standard

# Project Setup (01)
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

# Delivery Workflow (04) — NUEVO en v2.1
ln -sf "$SKILLS_DIR/04-delivery-workflow/merge-manager"    ~/.config/opencode/skills/merge-manager
ln -sf "$SKILLS_DIR/04-delivery-workflow/sprint-handoff"   ~/.config/opencode/skills/sprint-handoff
ln -sf "$SKILLS_DIR/04-delivery-workflow/context-guardian" ~/.config/opencode/skills/context-guardian
```

### Paso 5: Verificar la Configuración

```bash
# Verificar que los agentes están vinculados correctamente
ls -la ~/.config/opencode/agents/

# Verificar que los skills están vinculados correctamente
ls -la ~/.config/opencode/skills/
```

Deberías ver los **6 agentes** y los **23 skills** listados como symlinks (`->` apuntando al repositorio).

### Paso 6: Reiniciar OpenCode

Cierra y vuelve a abrir OpenCode. Los nuevos agentes estarán disponibles:
- Con la tecla `Tab` para ciclar entre agentes primarios (verás `technical-lead`).
- Con `@` para mencionar cualquier subagente directamente en el chat.

## Uso del Equipo

### Flujo Completo (Recomendado)
Activa el agente `technical-lead` con `Tab` y pide la feature completa:
```
Necesito crear la feature de autenticación. El endpoint es POST /auth/login
con campos username y password. El proyecto está en /ruta/a/mi/proyecto/flutter.
```
El Technical Lead coordinará automáticamente a `feature-developer`, `quality-auditor` y `ui-ux-craft`.

### Integración y Release
Cuando una feature esté lista:
```
@merge-manager Integra la rama feature/auth a development con los gate checks.
```
Para cerrar un MVP y liberar a main:
```
@merge-manager Ejecuta el release-merge de development a main con tag v1.0.0.
```

### Planificación de Sprints
Al iniciar un sprint o al cerrar una sesión:
```
@sprint-planner Genera el plan del sprint fase-01-auth con las features phone-auth y onboarding.
@sprint-planner Genera el handoff de consolidación tras el merge de feature/auth.
```

### Invocación Directa de Subagentes
También puedes invocar subagentes directamente mencionándolos con `@`:
```
@quality-auditor Revisa los archivos de la feature auth en lib/features/auth/
@feature-developer Solo necesito el BLoC para la feature de perfil de usuario
```

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
