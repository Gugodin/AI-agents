---
name: merge-manager
description: Protocolo de integración de ramas bajo Git Flow simplificado. Use cuando el usuario pida "haz el merge", "integrar la feature", "mergear a development", "cerrar la feature", "release del MVP", "subir a main", o al cerrar una fase o un MVP. Cubre feature-merge (feature → development) y release-merge (development → main + tag vX.0.0).
---

# Skill: Merge Manager — El Integrador

Este manual define el protocolo obligatorio para integrar ramas en repos del portafolio SFT.Tec bajo Git Flow simplificado (`main` → `development` → `feature/*`). El merge NUNCA se ejecuta como secuencia de comandos suelta: cada paso es un gate, y cualquier gate rojo aborta la integración.

## 0. Reglas previas (leer antes de tocar git)

1. **Jamás force-push a `main` ni a `development`.** Force-push solo se permite en ramas `feature/*` propias durante rebase.
2. **Jamás borrar una rama sin verificación verde.**
3. **Nunca commitear secrets:** revisar `git diff` buscando API keys, tokens, credenciales antes de cualquier merge.
4. **Ante duda en un conflicto, clasificarlo como semántico** y consultar. Nunca adivinar semántica de negocio.
5. Al finalizar cualquier protocolo exitoso, delegar el handoff al agente `sprint-planner` vía `Task`. Si no hay acceso a `Task` o el propio contexto está agotado (regla de `context-guardian`), escribir el handoff directamente siguiendo el template del skill `sprint-handoff`.

## 1. Detección de entorno

- Si existe `.fvmrc` en la raíz del proyecto → prefijo `fvm` (`fvm flutter analyze`); si no → `flutter` directo.
- Si `build_runner` está en `dev_dependencies` → correr `dart run build_runner build --delete-conflicting-outputs` ANTES del gate check (regenerar `*.freezed.dart` / `*.g.dart` desde fuentes).
- Los comandos de Flutter se ejecutan desde la raíz del proyecto Flutter (p. ej. `code/tlaxi_app`, no la raíz del repo).
- Flavors: si el proyecto define flavors (`dev/stage/prod`), el build de verificación usa `--flavor dev`.

## 2. Protocolo feature-merge (`feature/<name>` → `development`)

### 2.1 Preparación

```
git fetch origin
git status                          ← working tree limpio (si no: abortar y reportar)
git checkout feature/<name>
git log --oneline -10               ← conocer los commits a integrar
git diff development...HEAD --stat  ← inventario de archivos tocados
```

- Revisar el diff: si la feature toca archivos fuera de su scope o contiene secrets → abortar y reportar.
- Verificar que la rama nació de `development` (no de `main`): `git merge-base --is-ancestor $(git merge-base development HEAD) HEAD` y revisar el base contra development.

### 2.2 Rebase sobre development

```
git rebase origin/development
```

- Sin conflictos → continuar en 2.3.
- Conflictos → clasificar cada uno (§4) antes de tocar nada.

### 2.3 Gate checks (en la feature, tras el rebase)

En orden; cualquier fallo aborta (reportar output exacto):

1. `build_runner` (si aplica, ver §1)
2. `fvm flutter analyze` → 0 issues
3. `fvm dart format --set-exit-if-changed .` → sin cambios
4. `fvm flutter test` → verde (si `test/` no tiene tests todavía → NO abortar, pero registrar el gap en el reporte)
5. `fvm flutter build apk --debug --flavor dev` → exitoso (o `flutter build apk --debug` sin flavors)

### 2.4 Clasificación de conflictos (solo si el rebase falló)

**Triviales — auto-resolver:**

| Caso | Resolución |
|---|---|
| Imports añadidos en ambos lados sin colisión de nombres | Unión de ambos sets, orden alfabético |
| Keys nuevas en archivos `.arb` (l10n) | Unión de ambos lados |
| `pubspec.yaml`: dependencias DISTINTAS añadidas en ambos lados | Mantener ambas |
| Archivos generados (`*.freezed.dart`, `*.g.dart`, `*.drift.dart`) | NO resolver el generado: resolver el archivo fuente y regenerar con `build_runner` |
| Entradas nuevas al final de listas/registros independientes | Unión de ambas |

**Semánticos — abortar y consultar al usuario:**

- Mismo bloque de código modificado en ambos lados.
- Eliminación vs modificación del mismo código.
- Contratos de dominio: entidades, interfaces de repositorio, `typedefs` de foundation.
- Rutas del router o guards modificados en ambos lados.
- `pubspec.yaml`: versión de la MISMA dependencia divergente.
- Cualquier cosa sobre la que se dude (regla §0.4).

Al abortar: dejar el rebase en estado limpio (`git rebase --abort`), y reportar al usuario el conflicto con contexto de ambos lados y propuesta de resolución. Esperar instrucciones; nunca relanzar solo.

### 2.5 Merge

```
git checkout development
git merge --no-ff feature/<name> -m "<type>(<scope>): <descripción>"
git push origin development
```

- Mensaje conventional commit del tipo de la feature (`feat:`, `fix:`, `test:`, `spike:`, `docs:`).
- `--no-ff` SIEMPRE: preserva la historia de la feature como unidad.

### 2.6 Post-merge

```
git branch -d feature/<name>
git push origin --delete feature/<name>   ← solo si existía en el remoto
```

Luego:

1. `Task(sprint-planner)` → handoff de consolidación (ver skill `sprint-handoff` §tipo 2), indicando: rama mergeada, commits integrados, hash del merge, conflictos resueltos, sprint al que pertenece.
2. Reporte final al usuario: archivos integrados, commits, gates corridos (verde), handoff generado, ramas activas restantes.

## 3. Protocolo release-merge (`development` → `main`)

Se ejecuta SOLO al cerrar un MVP completo. Precondición documental: los criterios de salida del MVP (`docs/iteraciones/MVP-NN-*.md`) deben estar cumplidos y verificables — si no, abortar con el gap.

### 3.1 Verificación

```
git fetch origin
git checkout development
git status                          ← limpio
```

- Gate checks completos (§2.3) sobre `development` — todos verdes, sin excepciones (aquí el gap de tests SÍ aborta).

### 3.2 Release

```
git checkout main
git merge --no-ff development -m "release: MVP <N> — <nombre>"
git tag -a v<N>.0.0 -m "MVP <N> — <nombre> (criterios de salida cumplidos)"
git push origin main
git push origin v<N>.0.0
git checkout development            ← volver al flujo de trabajo
```

- Tag `vX.0.0`: MVP 1 → `v1.0.0`, MVP 2 → `v2.0.0`, MVP 3 → `v3.0.0`.
- NUNCA force-push a `main` (§0.1). Si el push rechaza, investigar divergencia, jamás forzar.

### 3.3 Post-release

1. `Task(sprint-planner)` → handoff de cierre de MVP (tipo 4 del skill `sprint-handoff`).
2. Reporte final: hash del merge, tag creado, criterios de salida referenciados, próximo MVP/sprint habilitado.

## 4. Reporte de aborto

Todo aborto reporta: gate que falló, output exacto del comando/conflicto, estado del repo al abortar (rama, rebase abortado o no), y qué se necesita del usuario para desbloquear. Un merge abortado deja el repo SIEMPRE en el estado previo al protocolo.
