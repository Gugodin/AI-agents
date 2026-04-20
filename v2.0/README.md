# Segi AI Agents v2.0

Repositorio con el equipo de agentes de IA especializados en la Arquitectura Segi v2.0 para proyectos Flutter con Clean Architecture. Estos agentes están diseñados para trabajar dentro de **OpenCode** y colaborar entre sí para generar código de alta calidad de forma autónoma.

## El Equipo

| Agente | Rol | Modo |
|--------|-----|------|
| `technical-lead` | Orquestador principal. Analiza requerimientos, planifica y coordina al equipo completo. También gestiona la inyección de dependencias. | `all` (primary + subagente) |
| `feature-developer` | Genera el código de Dominio, Datos y BLoC siguiendo Clean Architecture. | `subagente` |
| `quality-auditor` | Audita el código generado y emite veredicto `APPROVED` o `REQUEST CHANGES`. | `subagente` |
| `ui-ux-craft` | Construye interfaces Flutter modulares consumiendo los estados del BLoC. | `subagente` |

## Estructura del Repositorio

```
v2.0/
├── agents/                        # Definiciones de agentes (formato OpenCode Markdown)
│   ├── technical-lead.md
│   ├── feature-developer.md
│   ├── quality-auditor.md
│   └── ui-ux-craft.md
└── modular-skills-v2/             # Manuales de conocimiento (Skills de OpenCode)
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
    │       └── ui-specialist/
    └── 03-quality-assurance/
        └── code-reviewer/
```

## Configuración en una Máquina Nueva

> Sigue estos pasos en orden para tener el equipo funcionando en cualquier máquina con OpenCode instalado.

### Requisitos Previos
- [OpenCode](https://opencode.ai) instalado y configurado con un proveedor de IA.
- Git instalado.

### Paso 1: Clonar el Repositorio

```bash
# Clona el repositorio en la ubicación que prefieras
git clone <URL_DEL_REPOSITORIO> ~/my-ai-agents
```

> Anota la ruta donde clonaste el repo. La necesitarás en el Paso 3.
> En este ejemplo usaremos `~/my-ai-agents/v2.0` como ruta base.

### Paso 2: Crear los Directorios de OpenCode

```bash
mkdir -p ~/.config/opencode/agents
mkdir -p ~/.config/opencode/skills
```

### Paso 3: Vincular Agentes con Symlinks

Los symlinks conectan los archivos del repositorio directamente con OpenCode. Cuando actualices el repo (git pull), los cambios se reflejan automáticamente sin reconfiguraciones adicionales.

```bash
# Reemplaza RUTA_BASE con la ruta absoluta donde clonaste el repo
# Ejemplo: /Users/tu_usuario/my-ai-agents/v2.0
RUTA_BASE="$HOME/my-ai-agents/v2.0"

# Vincular los 4 agentes
ln -sf "$RUTA_BASE/agents/technical-lead.md"    ~/.config/opencode/agents/technical-lead.md
ln -sf "$RUTA_BASE/agents/feature-developer.md" ~/.config/opencode/agents/feature-developer.md
ln -sf "$RUTA_BASE/agents/quality-auditor.md"   ~/.config/opencode/agents/quality-auditor.md
ln -sf "$RUTA_BASE/agents/ui-ux-craft.md"       ~/.config/opencode/agents/ui-ux-craft.md
```

### Paso 4: Vincular Skills con Symlinks

```bash
RUTA_BASE="$HOME/my-ai-agents/v2.0"
SKILLS_DIR="$RUTA_BASE/modular-skills-v2"

# Core Rules
ln -sf "$SKILLS_DIR/00-core-rules/blueprint-foundation"      ~/.config/opencode/skills/blueprint-foundation
ln -sf "$SKILLS_DIR/00-core-rules/data-state-standard"       ~/.config/opencode/skills/data-state-standard
ln -sf "$SKILLS_DIR/00-core-rules/network-standards"         ~/.config/opencode/skills/network-standards
ln -sf "$SKILLS_DIR/00-core-rules/use-case-standard"         ~/.config/opencode/skills/use-case-standard

# Project Setup
ln -sf "$SKILLS_DIR/01-project-setup/native-configuration"   ~/.config/opencode/skills/native-configuration
ln -sf "$SKILLS_DIR/01-project-setup/project-init"           ~/.config/opencode/skills/project-init
ln -sf "$SKILLS_DIR/01-project-setup/scaffolding-dart"       ~/.config/opencode/skills/scaffolding-dart

# Feature Factory — Data
ln -sf "$SKILLS_DIR/02-feature-factory/data/data-model-specialist"      ~/.config/opencode/skills/data-model-specialist
ln -sf "$SKILLS_DIR/02-feature-factory/data/data-repository-specialist" ~/.config/opencode/skills/data-repository-specialist
ln -sf "$SKILLS_DIR/02-feature-factory/data/data-source-addition"       ~/.config/opencode/skills/data-source-addition

# Feature Factory — Domain
ln -sf "$SKILLS_DIR/02-feature-factory/domain/domain-entity-specialist"     ~/.config/opencode/skills/domain-entity-specialist
ln -sf "$SKILLS_DIR/02-feature-factory/domain/domain-repository-specialist" ~/.config/opencode/skills/domain-repository-specialist
ln -sf "$SKILLS_DIR/02-feature-factory/domain/use-case-addition"            ~/.config/opencode/skills/use-case-addition

# Feature Factory — Presentation & Dependency
ln -sf "$SKILLS_DIR/02-feature-factory/presentation/bloc-specialist" ~/.config/opencode/skills/bloc-specialist
ln -sf "$SKILLS_DIR/02-feature-factory/presentation/ui-specialist"   ~/.config/opencode/skills/ui-specialist
ln -sf "$SKILLS_DIR/02-feature-factory/dependency-manager"           ~/.config/opencode/skills/dependency-manager

# Quality Assurance
ln -sf "$SKILLS_DIR/03-quality-assurance/code-reviewer" ~/.config/opencode/skills/code-reviewer
```

### Paso 5: Verificar la Configuración

```bash
# Verificar que los agentes están vinculados correctamente
ls -la ~/.config/opencode/agents/

# Verificar que los skills están vinculados correctamente
ls -la ~/.config/opencode/skills/
```

Deberías ver los 4 agentes y los 17 skills listados como symlinks (`->` apuntando al repositorio).

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
1. Crea la carpeta en la sección correcta de `modular-skills-v2/` con un `SKILL.md` que tenga frontmatter (`name` y `description`).
2. Crea el symlink correspondiente en `~/.config/opencode/skills/` en todas las máquinas.
3. Agrega el comando de symlink a la sección de "Paso 4" de este README.
4. Haz commit y push al repositorio.
