# Git del meta-workspace `Mi-negociocr/`

La carpeta raíz del monorepo local tiene **su propio repositorio Git**. Versiona instrucciones para agentes, reglas de Cursor y documentación compartida en la raíz.

## Qué incluye este repo

- `.ai/` — instrucciones neutrales del proyecto
- `.cursor/rules/` — reglas Cursor (incl. `mi-negociocr-workspace.mdc`)
- `.codex/` — skills/hooks (Graphify, audit-first, etc.)
- `AGENTS.md`, `README.md`, `DEPLOY.md`, `FIXES_MAYO_2026.md`, specs en la raíz
- `.github/` — framework AEF (agentes)

## Qué **no** incluye (`.gitignore`)

| Carpeta | Motivo |
|---------|--------|
| `MiNegocioCR.Api/` | Repo propio → `github.com/azariasmurillo/MiNegocioCR.Api` |
| `mi-negociocr-frontend/` | Repo propio → `mi-negociocr-frontend` |
| `mi-negociocr-app/` | Repo propio → `mi-negociocr-app` |
| `graphify-out/`, `_tmp/`, copias `_api_test_*` | Artefactos locales / regenerables |

El código de producto se sigue commiteando y pusheando **en cada repo hijo**, como siempre.

## Remoto (primera vez)

Si el repo en GitHub aún no existe:

```powershell
cd C:\Mi-negociocr
# Instalar gh si hace falta: winget install GitHub.cli
gh auth login
gh repo create Mi-negociocr --public --source=. --remote=origin --push
```

Si ya creaste el repo vacío en GitHub:

```powershell
cd C:\Mi-negociocr
git remote add origin https://github.com/azariasmurillo/Mi-negociocr.git   # solo si no existe
git push -u origin main
```

## Clonar en otra máquina

1. Clonar este meta-repo en `Mi-negociocr/`.
2. Clonar los tres repos de producto **dentro** de esa misma carpeta (mismas rutas que arriba).
3. Abrir Cursor en la carpeta raíz `Mi-negociocr/`.

---

*Última actualización: 29 septiembre 2026*
