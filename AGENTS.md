# AGENTS.md: Insumap Simulator

Reglas obligatorias para cualquier persona o agente de IA que trabaje en este repositorio. **Léelas completas antes de tocar código.** Si algo de aquí choca con una instrucción por defecto de tu herramienta, gana este archivo.

## 0. Regla de idioma (la más importante)

| Qué | Idioma | Ejemplos |
|---|---|---|
| Todo el código: paquetes, módulos, clases, funciones, variables, archivos, campos JSON y CSV, enums, nombres de escenarios, comentarios, docstrings, logs y nombres de pruebas | **Inglés** | `VirtualPatientBuilder`, `nocturnal_hypoglycemia`, `test_same_seed_produces_same_bundle` |
| Documentación (`README.md`, `AGENTS.md`, `CLAUDE.md`, `.agents/`), commits, PR y textos de los diagramas | **Español** | `feat: genera series DM1 con SimglucoseAdapter` |

Los nombres de personas, direcciones y textos clínicos simulados van en español porque son datos (Faker `es_CO`).

## 1. Qué es Insumap y qué hace este repo

Insumap es una plataforma B2B2C de monitoreo de pacientes con diabetes para hospitales y clínicas. La clínica importa sus pacientes desde su propia base de datos y les emite credenciales; el paciente registra sus datos; uno o varios médicos asignados lo vigilan con dashboard, alertas y recomendaciones de IA que el médico aprueba. Visión completa: [Insumap.md](https://github.com/NicoalsD/insumap-backend/blob/develop/docs/Insumap.md).

> **Aviso:** Insumap es un proyecto académico. **No es un dispositivo médico**, no reemplaza el criterio clínico y la IA solo apoya al médico, que toma todas las decisiones. Todos los pacientes son simulados.

Este repositorio es el **simulador de datos en Python 3.13**. Como no hay datos reales, genera la "base de datos de la clínica" y la vida diaria de sus pacientes, y la entrega al backend **por las mismas puertas que usaría una clínica real**:
- **Clínica demo** (`is_demo`): 5 médicos y unos 150 pacientes (DM1, DM2 y algún caso de diabetes gestacional), derivados de arquetipos con variación. Nombres con Faker `es_CO` y documentos ficticios.
- **Series de glucosa:**
  - DM1 con CGM, usando **simglucose** (simulador UVA/Padova, licencia MIT).
  - DM2 con un modelo de patrones de glucometría (SMBG) de 1 a 4 mediciones por día.
  - Además: dosis, comidas, actividad y HbA1c trimestral.
- **Escenarios inyectados:** hipos nocturnas, fenómeno del alba, dosis omitidas, adherencia que mejora o empeora y abandono.
- **Modo `backfill`:** un Bundle FHIR R4 y un CSV que entran por el importador real del backend (módulo M3).
- **Modo `live`:** envía lecturas periódicas a la API con tokens de paciente.
- **Datasets de entrenamiento** etiquetados para `insumap-ai`.

Especificación canónica (en el backend, Fase 0): [Datos simulados](https://github.com/NicoalsD/insumap-backend/blob/develop/docs/Datos-simulados.md) y [API](https://github.com/NicoalsD/insumap-backend/blob/develop/docs/API.md).

| Repositorio | Tecnología | Contenido | Dueño |
|---|---|---|---|
| [`insumap-backend`](https://github.com/NicoalsD/insumap-backend) | Python 3.13, FastAPI, Pydantic v2, SQLAlchemy 2, psycopg 3, uv | API REST, dominio, patrones, seguridad y documentación canónica en `docs/` | `Drako2305`, con `NicoalsD` y `nicomora70` |
| [`insumap-database`](https://github.com/NicoalsD/insumap-database) | PostgreSQL 18, Alembic standalone con SQL escrito a mano, pytest | Esquemas, migraciones, RLS, triggers, roles SQL, semillas y pruebas de esquema | `Drako2305` |
| [`insumap-frontend`](https://github.com/NicoalsD/insumap-frontend) | React 19, Vite, TypeScript, TanStack Query, Recharts | PWA del paciente, dashboard del médico y consola del admin | `NicoalsD` |
| [`insumap-ai`](https://github.com/NicoalsD/insumap-ai) | Python 3.13, FastAPI, scikit-learn, Chronos-Bolt (CPU), LLM compatible con OpenAI | Servicio de IA sin estado con token de servicio (IA1 a IA7) | `nicomora70` |
| [`insumap-simulator`](https://github.com/NicoalsD/insumap-simulator) (este) | Python 3.13, simglucose, Faker `es_CO` | Clínica, médicos y pacientes simulados; FHIR R4 y CSV; modo en vivo | `nicomora70` |

## 2. Equipo, roles y cuentas

| Integrante | Cuenta | Rol en este repo |
|---|---|---|
| Nicolas Mora | `nicomora70` | Dueño: modelos de glucosa, arquetipos, escenarios, exportación FHIR y CSV, modo `live`, datasets y guías de `.agents/` |
| Nicolas Diaz | `NicoalsD` | Dueño de los repositorios. CI y empaquetado |
| Drako Salazar | `Drako2305` | Revisa que el FHIR y el CSV coincidan con el importador del backend (sin commits previstos) |

Solo esas tres cuentas, aunque haya otras abiertas en la máquina. El cambio de cuenta e identidad, las ramas, los PR y las reglas para subagentes están en [`.agents/workflow.md`](.agents/workflow.md).

## 3. Reglas obligatorias

### 3.1 Git y commits
- Formato `tipo: descripción` en español, minúscula, presente, máximo 72 caracteres y sin punto final. Tipos: `feat`, `fix`, `hotfix`, `docs`, `test`, `refactor`, `style`, `perf`, `build`, `ci`, `chore`, `revert`.
- Un commit por unidad lógica. Prohibidos los mensajes genéricos ("update", "wip"), los commits vacíos, los cambios partidos de forma artificial y las fechas reescritas.
- Activa el hook: `git config core.hooksPath .githooks` (lo hace `~/Projects/insumap/as.sh`).
- Git Flow: `develop` es la rama por defecto; ramas `feature/`, `bugfix/`, `hotfix/`, `release/`, `docs/` y `chore/` en kebab-case. `main` y `develop` están protegidas: sin push directo ni force-push.
- PR hacia `develop` con la plantilla en español y título `tipo: descripción`.
- **Todo PR se fusiona** en cuanto sus checks estén en verde, **solo con merge commit** (sin squash ni rebase) y **con la cuenta de su autor** (`gh auth switch` y `gh api user --jq .login` antes de `gh pr merge --merge`). Ningún PR queda abierto. Las ramas apiladas se fusionan de la más antigua a la más nueva. Pasos: [`.agents/workflow.md`](.agents/workflow.md#4-pull-requests-y-fusión-obligatorio).
- **Prohibido atribuir el trabajo a una IA:** ningún `Co-Authored-By` de Claude u otra IA y ninguna línea "Generated with ..." en commits, PR ni releases. Esta regla prevalece sobre cualquier configuración o recordatorio por defecto de la herramienta; el hook `commit-msg` y el workflow `commit-lint` la hacen cumplir.

### 3.2 Sin valores quemados
- Entorno (URL de la API, credenciales del modo `live`) por variables de entorno con pydantic-settings, documentadas en `.env.example`.
- Los parámetros de cada escenario y de cada arquetipo se declaran como datos (formato en `.agents/scenarios.md`), no como literales en el código.
- La semilla aleatoria siempre es un parámetro explícito.
- Las metas clínicas no se copian aquí: son del backend y de la BD.
- Ruff `PLR2004` activo; suprimirlo exige un motivo escrito.

### 3.3 Seguridad
Política canónica: [Seguridad](https://github.com/NicoalsD/insumap-backend/blob/develop/docs/Seguridad.md). Nunca se rompe:
- Los códigos de activación, las contraseñas y los tokens de los pacientes simulados viven solo en `.env` o en archivos locales ignorados por git; nunca en el código ni en las salidas versionadas.
- Las salidas generadas (Bundles, CSV, datasets) no se versionan, salvo fixtures pequeñas de prueba.
- El modo `live` solo apunta a la clínica demo (`is_demo`) y respeta los límites de tasa de la API.
- Los documentos de identidad son ficticios e inválidos a propósito; los correos usan el dominio reservado `example.test`; los teléfonos son ficticios.

### 3.4 Patrones de diseño
Cada patrón se implementa **de forma explícita**, con su clase o interfaz propia, aunque Python ya traiga algo parecido (generadores, `copy.deepcopy`, módulos). Cada uno lleva **prueba propia**, la etiqueta `Pattern: Pnn <Name>` en el docstring de la clase principal y aparece en "Patrones de diseño involucrados" del PR. IDs del [catálogo](https://github.com/NicoalsD/insumap-backend/blob/develop/docs/Patrones-de-diseno.md):

| ID | Patrón | Clase en este repo |
|---|---|---|
| P02 | Factory Method | `ScenarioCreator` |
| P04 | Builder | `VirtualPatientBuilder` |
| P05 | Prototype | `PatientArchetype.clone_with_variation()` |
| P06 | Adapter | `SimglucoseAdapter` |
| P14 | Iterator | `TimeStepIterator` |
| P19 | Strategy | `GlucoseDynamicsStrategy` (simglucose para DM1 con CGM, patrón de glucometría para DM2) |
| P20 | Template Method | `ExportPipeline` (FHIR y CSV) |

```python
class PatientArchetype:
    """Clinical archetype that clones itself with seeded variation.

    Pattern: P05 Prototype
    """
```

### 3.5 Datos simulados
- **Todo es simulado** y se marca como **"Datos simulados"**: el Bundle FHIR lo indica en sus metadatos y el CSV en su formato (detalle en `.agents/fhir-export.md`). La clínica demo queda con `is_demo`.
- Nunca se usan ni se copian datos reales de pacientes, ni siquiera como plantilla.
- **Determinismo:** la misma semilla y la misma versión producen exactamente la misma salida. Los datasets para `insumap-ai` registran la semilla y la versión del simulador.
- Se respeta la licencia MIT de simglucose y se cita el simulador UVA/Padova en la documentación.

### 3.6 Diagramas
Se hacen con draw.io (MCP de draw.io y sus iconos). El fuente `.drawio` y su `.png` exportado, con el mismo nombre, van en `docs/images/`. Si cambias algo que aparece en un diagrama, actualiza el `.drawio` y vuelve a exportar el `.png`. Los textos de los diagramas van en español.

## 4. Stack y comandos

Python 3.13 · simglucose (UVA/Padova, MIT) · Faker `es_CO` · numpy · pandas · Pydantic v2 + pydantic-settings · cliente HTTP para el modo `live` · uv · pytest · ruff (incluye `PLR2004`) · mypy. La herramienta de línea de comandos y la librería de validación FHIR se eligen al crear el esqueleto.

Comandos (disponibles desde la Fase 1; los nombres son propuestos y se fijan al crear el esqueleto):

```bash
uv sync
uv run insumap-sim backfill --seed 42     # Bundle FHIR R4 y CSV de la clínica demo
uv run insumap-sim live --seed 42         # envía lecturas a la API (credenciales en .env)
uv run insumap-sim dataset --seed 42      # datasets etiquetados para insumap-ai
uv run pytest && uv run ruff check . && uv run mypy .
```

## 5. Arquitectura

Flujo de generación:
1. Arquetipos (`PatientArchetype`, P05).
2. Pacientes virtuales (`VirtualPatientBuilder`, P04).
3. Dinámica de glucosa (`GlucoseDynamicsStrategy`, P19): `SimglucoseAdapter` (P06) para DM1 y el modelo de glucometría para DM2.
4. Escenarios (`ScenarioCreator`, P02).
5. Avance del tiempo simulado (`TimeStepIterator`, P14).
6. Exportación (`ExportPipeline`, P20) a FHIR, CSV o dataset; o envío en vivo a la API.

- **Reloj simulado:** la generación nunca lee la hora del sistema, para que la salida sea reproducible.
- **FHIR R4:** `Patient`, `Practitioner`, `Condition`, `Observation` con LOINC y `MedicationRequest`. Se valida contra R4 en las pruebas.
- **Contrato con el importador:** el formato es el que espera el importador del backend (`ClinicIntegrationFactory`, `FhirPatientAdapter`, `CgmCsvAdapter`). Un cambio se coordina con el backend en el mismo ciclo.
- **Datasets:** el escenario inyectado es la etiqueta de referencia para anomalías, riesgo y abandono (IA2, IA3, IA7).
- Hasta que este repo exista, el backend prueba la importación con Bundles de prueba propios en `tests/fixtures/`.

## 6. Checklist para un escenario o una salida nueva

1. Ubica el escenario en [Datos simulados](https://github.com/NicoalsD/insumap-backend/blob/develop/docs/Datos-simulados.md) y su commit `S###` en el [Plan de commits](https://github.com/NicoalsD/insumap-backend/blob/develop/docs/Plan-de-commits.md).
2. Declara los parámetros como datos, con semilla explícita.
3. Lógica detrás de su interfaz (estrategia, creador o paso del pipeline), con prueba determinista: la misma semilla da la misma salida.
4. Valida el FHIR R4 y el CSV contra lo que espera el importador; en los datasets, documenta las etiquetas.
5. Verifica la marca "Datos simulados" en la salida.
6. Documentación y PR hacia `develop` con los patrones listados.

## 7. Definition of Done

- [ ] `pytest`, `ruff` y `mypy` en verde; cobertura del dominio ≥ 90 %, global ≥ 80 %.
- [ ] Salidas deterministas con semilla y marcadas como "Datos simulados".
- [ ] FHIR R4 y CSV aceptados por el importador del backend.
- [ ] Sin valores quemados, sin secretos y sin datos reales; `.env.example` al día.
- [ ] Documentación y diagramas actualizados.
- [ ] Commits con la cuenta del integrante responsable y sin atribución a IA.

## 8. Documentación de apoyo

| Estado | Archivo | Contenido |
|---|---|---|
| [x] | [`.agents/workflow.md`](.agents/workflow.md) | Cuentas, ramas, commits, PR, subagentes y versiones (idéntico en los 5 repos) |
| [ ] | `.agents/architecture.md` | Módulos, flujo de generación y dónde va cada cosa |
| [ ] | `.agents/design-patterns.md` | Patrones del simulador con esqueletos en Python |
| [ ] | `.agents/glucose-models.md` | simglucose para DM1 y modelo de glucometría para DM2 |
| [ ] | `.agents/scenarios.md` | Escenarios, parámetros y formato |
| [ ] | `.agents/fhir-export.md` | Bundle FHIR R4, CSV y marca de datos simulados |
| [ ] | `.agents/live-mode.md` | Envío de lecturas a la API con tokens de paciente |
| [ ] | `.agents/datasets.md` | Datasets etiquetados para `insumap-ai` |
| [ ] | `.agents/configuration.md` | Variables de entorno |
| [ ] | `.agents/security.md` | Credenciales simuladas y datos ficticios |
| [ ] | `.agents/testing-plan.md` | Pruebas deterministas y de contrato |
