# AI Engineering Framework (AEF)

Framework de ingeniería con agentes de IA, **independiente de proyecto y de herramienta**.

## Arquitectura

| Capa | Qué es | Ubicación |
|---|---|---|
| **Core** | Reglas genéricas, Roles, Workflow, Principles, Decision Tree, Git Policy | `framework/`, `skills/`, `AGENTS.md` |
| **Packs** | Conocimiento de tecnologías e integraciones | `packs/` |
| **Project Profiles** | Configuración de cada proyecto consumidor | `project-profiles/` |

## Punto de entrada

`AGENTS.md`

## Cómo se usa

1. Cargar `AGENTS.md` y el Core.
2. Activar el Project Profile del repositorio consumidor.
3. Cargar los Packs declarados por ese profile.
4. Clasificar la solicitud por **intención** (`framework/DECISION-TREE.md`).
5. Resolver la intención con el **skill-map** del Project Profile.
6. Ejecutar el Skill.

## Consumidor de referencia

`project-profiles/<proyecto>/` — cada proyecto es un consumidor del AEF (ejemplo de referencia: `marchamo`).

## Principios de producto

- No runtime nuevo.
- No capas fuera de Core / Packs / Project Profiles.
- Decision Tree por intención, no por nombres de Skills.
- El stack tecnológico vive en el Project Profile, no en el Core.
