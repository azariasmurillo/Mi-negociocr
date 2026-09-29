# Project Profile — Marchamo Digital

Este Project Profile hace que Marchamo Digital sea un **consumidor** del AI Engineering Framework (AEF).

No forma parte del Core.

---

## Identidad

| Campo | Valor |
|---|---|
| Profile ID | `marchamo` |
| Proyecto | Marchamo Digital |
| Organización GitHub | `Grupo-INS` |
| Patrón de repos | `MarchamoDigital.*` |
| Repo de estándares | `MarchamoDigital.Base` |

---

## Docs Layout

| Clave | Valor |
|---|---|
| `docs.designs_root` | `docs/40_disenos_y_propuestas` |
| Plantilla borrador | `docs/00_plantillas/plantilla_borrador.md` |
| Plantilla vigente | `docs/00_plantillas/plantilla_documento_vigente.md` |
| Playbook documentación | `docs/00_plantillas/playbook_documentacion.md` |
| Playbook ADO | `Analisis Épicas/shared/ado_query_playbook.yaml` |
| Índice | `docs/INDEX.md` |

---

## Packs instalados

Este profile asume los siguientes Packs del AEF:

- `packs/dotnet`
- `packs/dotnet-react`
- `packs/azure`
- `packs/ado`
- `packs/docs`
- `packs/github-review`

---

## Stack Profile

Stack canónico del ecosistema Marchamo Digital:

- Clean Architecture
- SOLID
- CQRS
- MediatR
- FastEndpoints
- FluentValidation
- AutoMapper
- Autofac
- .NET / ASP.NET Core
- Cosmos DB (API MongoDB) / SQL según contexto
- xUnit

Estándares de referencia:

- `MarchamoDigital.Base/ESTANDAR_API_MARCHAMO.md`
- `MarchamoDigital.Base/ESTANDAR_SEGURIDAD_MARCHAMO.md`
- `MarchamoDigital.Base/.github/instructions/`

---

## Skill Map

Resolución de **intención → Skill** para este proyecto.

| Intención (Decision Tree) | Skill | Ubicación |
|---|---|---|
| `architecture` | `arquitecto-de-software` | Core `skills/` |
| `create-module` | `create-module` | Profile `skills/` |
| `implement` | `senior-dotnet` | Pack `dotnet` |
| `implement` (dotnet+react) | `senior-dev-dotnet-react` | Pack `dotnet-react` |
| `review-code` | `pr-review-marchamo` | Profile `skills/` |
| `review-code-hard` | `pr-hard-review` | Pack `github-review` |
| `validate-review-findings` | `code-review-validator` | Pack `github-review` |
| `publish-review` | `pr-review-publisher` | Profile `skills/` |
| `documentation` | `doc-tecnica` / `doc-funcional` / `doc-arquitectura` / `doc-procesos` | Packs / Profile |
| `generate-blueprint` | `generador-blueprint-hu` | Profile `skills/` |
| `data-modeling` | `modelado-datos-bd` | Profile `skills/` |
| `api-testing` | `qa-api-testing` / `localhost-api-testing` | Profile `skills/` |
| `platform` | `implementador-plataforma-azure` | Pack `azure` |
| `work-management` | `refinador-historias-ado` / `registrar-tareas-diarias` / `ado-tareas-report` / `analizador-de-requisitos` | Pack `ado` / Profile |
| `knowledge` | `faro-del-conocimiento` / `agent-customization` | Core `skills/` |
| `git-pr` | `github-pr-manager` / `pr-content-from-commits` | Core `skills/` |
| `architecture-quality` | `dotnet-architecture-quality` | Pack `dotnet` |
| `android-ui` | `android-ui-ux-expert` / `android-ui-ux-reviewer` / `android-device-deploy` | Profile `skills/` |
| `cosmos-explore` | `cosmos-marchamo-explorer` | Profile `skills/` |

Intenciones sin Skill mapeado (`add-operation`, `new-version`, `fix-build`, `fix-tests`, `environment`, `cross-repo-review`):

Usar el Workflow general del Core hasta que exista un Skill dedicado.

---

## Azure DevOps

| Campo | Valor |
|---|---|
| Organización | Según tenant del equipo |
| Proyecto | `Seguro Obligatorio Automotor` |
| Team | `Marchamo Digital Team` |
| Area Path | `Seguro Obligatorio Automotor\Marchamo Digital Team` |

Detalle operativo: skills `registrar-tareas-diarias`, `ado-tareas-report`, `refinador-historias-ado`.

---

## Infraestructura

Referencias operativas (detalle en skills del profile):

- Cosmos DB account / database del entorno Marchamo Digital
- AKS Dev / Local
- Pipelines: `Marchamo.Digital.Pipelines`

---

## Skills exclusivas del profile

Ubicación: `project-profiles/marchamo/skills/`

- `pr-review-marchamo`
- `pr-review-publisher`
- `create-module`
- `generador-blueprint-hu`
- `cosmos-marchamo-explorer`
- `localhost-api-testing`
- `qa-api-testing`
- `modelado-datos-bd`
- `doc-arquitectura`
- `doc-funcional`
- `registrar-tareas-diarias`
- `ado-tareas-report`
- `android-ui-ux-expert`
- `android-ui-ux-reviewer`
- `android-device-deploy`

---

## Uso

1. Cargar `AGENTS.md` y el Core (`framework/`).
2. Activar este Project Profile.
3. Cargar los Packs listados arriba.
4. Resolver intenciones con este skill-map.
