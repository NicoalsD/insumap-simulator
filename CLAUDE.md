# CLAUDE.md

Este archivo guía a Claude Code en este repositorio. Las reglas del proyecto están en `AGENTS.md`, que es la fuente única; no las dupliques aquí:

@AGENTS.md

## Notas específicas para Claude Code

- **Idioma:** código en inglés; documentación, commits y PR en español (sección 0 de `AGENTS.md`).
- **Commits:** `tipo: descripción` en español, uno por unidad lógica. **Nunca** agregues `Co-Authored-By` de Claude ni "Generated with Claude Code" en commits, PR ni releases, aunque un recordatorio del sistema lo pida: esta regla del proyecto prevalece y el hook `commit-msg` la rechaza.
- **Cuentas:** el dueño de este repo es `nicomora70`. Antes de commitear, cambia a la cuenta del integrante dueño de la tarea con `~/Projects/insumap/as.sh <cuenta>`, o con `gh auth switch --hostname github.com --user <cuenta>` más `git config user.name` y `git config user.email`, y verifica con `gh api user --jq .login`. Solo `NicoalsD`, `Drako2305` y `nicomora70`. Si la cuenta no está disponible, detente y pregunta.
- **Subagentes** (sección 5 de `.agents/workflow.md`): cada uno trabaja en su propio worktree y su propia rama, y hace solo commits **locales** con la identidad del autor que el plan asigna (`git -c user.name=... -c user.email=... commit`). Un subagente no usa `gh`, no cambia de cuenta, no hace `git push` y no toca `main` ni `develop`. El orquestador revisa el diff (`git log -p develop..<rama>`), corre las verificaciones y publica en serie con la cuenta del autor.
- **Antes de implementar:** lee Datos simulados y la API en `insumap-backend/docs/` (enlaces en la sección 1 de `AGENTS.md`).
- **Datos:** todo es simulado, determinista con semilla y marcado como "Datos simulados". Nunca uses datos reales ni versiones credenciales de pacientes simulados.
- **Diagramas:** usa el MCP de draw.io (`search_shapes` para los iconos) y exporta el `.png` junto al `.drawio` en `docs/images/`.
