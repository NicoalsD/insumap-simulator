# Insumap - Simulador

Genera la clínica demo de Insumap: médicos y unos 150 pacientes simulados (DM1, DM2 y algún caso gestacional) con series de glucosa, dosis, comidas y actividad.

Exporta Bundles FHIR R4 y CSV para el importador del backend (modo backfill), envía lecturas en vivo a la API (modo live) y produce datasets de entrenamiento para `insumap-ai`.

## Estado

En construcción (Fase 0: documentación y planeación).

## Repositorios de Insumap

| Repositorio | Rol |
|---|---|
| [insumap-backend](https://github.com/NicoalsD/insumap-backend) | API, dominio, patrones, seguridad y documentación canónica |
| [insumap-database](https://github.com/NicoalsD/insumap-database) | Esquemas, migraciones, RLS, roles SQL y semillas (PostgreSQL 18) |
| [insumap-frontend](https://github.com/NicoalsD/insumap-frontend) | PWA del paciente, dashboard del médico y consola del admin |
| [insumap-ai](https://github.com/NicoalsD/insumap-ai) | Servicio de IA: pronóstico, triage, patrones y LLM con guardrails |
| [insumap-simulator](https://github.com/NicoalsD/insumap-simulator) | Clínica, médicos y pacientes simulados; exporta FHIR y CSV |

## Stack

Python 3.13, simglucose, Faker `es_CO`, uv, ruff, mypy `--strict` y pytest.

## Inicio rápido

> Disponible desde la Fase 1.

```bash
uv sync
uv run insumap-simulator backfill --out out/    # Bundle FHIR R4 y CSV de la clínica demo
uv run insumap-simulator live                   # lecturas periódicas a la API
uv run pytest
uv run ruff check .
uv run mypy .
```

## Documentación

- [`AGENTS.md`](AGENTS.md): reglas para agentes y convenciones del repositorio.
- [`CONTRIBUTING.md`](CONTRIBUTING.md): cómo contribuir (cuentas, ramas, commits y PR).
- [Índice de documentación canónica](https://github.com/NicoalsD/insumap-backend/tree/develop/docs).

## Equipo

- Nicolas Diaz ([`NicoalsD`](https://github.com/NicoalsD))
- Drako Salazar ([`Drako2305`](https://github.com/Drako2305))
- Nicolas Mora ([`nicomora70`](https://github.com/nicomora70))

> Proyecto académico. No es un dispositivo médico ni reemplaza el criterio del médico tratante. Los datos de pacientes son simulados.
