---
name: pr-content-from-commits
description: 'Genera contenido de Pull Request desde commits y diff, incluyendo comentarios por commit, riesgos y casos sugeridos de prueba manual. Use when: descripcion de PR, comentarios de commits, notas para reviewer, completar template de PR, redactar PR desde git log o git diff, checklist con evidencia, formato markdown para PR.'
argument-hint: 'Opcional: rama base (default origin/develop), ticket/HU y objetivo funcional para enriquecer la salida.'
---

# PR Content From Commits

---

## Framework Integration

Este Skill pertenece al AI Engineering Framework (Core).

Antes de ejecutar cualquier acción debe seguir obligatoriamente:

- `framework/PRINCIPLES.md` — Principios globales que gobiernan todo el ecosistema.
- `framework/WORKFLOW.md` — Flujo de trabajo oficial: comprender, analizar, planificar, implementar, validar, esperar aprobación.
- `framework/DECISION-TREE.md` — Árbol de decisión para selección de Skills.
- `framework/GIT-POLICY.md` — Política oficial para todas las operaciones Git.
- `framework/CONVENTIONS.md` — Convenciones de nombres, estructura y organización.

---

## Framework Role

**Responsabilidad de este Skill:**
Generar contenido listo para pegar en el template de Pull Request a partir de commits reales del repositorio.

**Problema que resuelve:**
Redactar descripciones de PR manualmente es tedioso y propenso a omisiones — este Skill extrae el contexto real de los commits y lo formatea según el template del proyecto.

**Responsabilidades que NO tiene este Skill:**
- No crea ni publica el PR (eso requiere autorización explícita, ver `framework/GIT-POLICY.md`).
- No revisa la calidad del código (eso es la intención `review-code` vía skill-map).

---

Genera contenido listo para pegar en el template de Pull Request del proyecto desde commits reales.

## Cuándo usar
- Redactar una descripción de PR con base en commits reales.
- Necesitas comentarios por commit con impacto y riesgo.
- Reducir texto manual y mantener formato consistente.
- Marcar checklist solo con evidencia real.

## Procedimiento

### 1. Recolectar contexto
```bash
# Merge-base entre HEAD y rama destino
git merge-base HEAD origin/develop

# Commits del rango
git log <merge-base>..HEAD --oneline

# Resumen de archivos impactados
git diff <merge-base>..HEAD --stat
git diff <merge-base>..HEAD --name-status
```

### 2. Clasificar tipo de cambio
Inferir `Feature`, `Bugfix`, `Hotfix`, `Refactor`, `Docs`, `Chore` desde commits y diff.

### 3. Redactar secciones del PR

**Descripción**: 1 párrafo + 3-6 bullets por componente impactado.

**Checklist**: no inventar validaciones; marcar pendiente cuando falte evidencia.

**Notas para reviewer**: comentarios por commit, riesgos y focos de revisión, casos sugeridos de prueba manual.

### 4. Validar consistencia
- No repetir texto entre secciones.
- No inventar endpoints, pruebas ni decisiones no evidenciadas.
- Mantener español técnico y concreto.

## Reglas de salida
- Solo las secciones del template — no agregar secciones extra.
- No incluir el output raw de git en la descripción del PR.
- Los riesgos deben ser concretos (qué puede romperse, por qué).
- Los casos de prueba deben ser ejecutables manualmente.
- Sin opiniones sobre decisiones de arquitectura a menos que sean un riesgo concreto de regresión.

## Template de salida

```markdown
## Descripción
[1 párrafo de contexto]
- [bullet por componente impactado]

## Tipo de cambio
- [ ] Feature
- [ ] Bugfix / Hotfix
- [ ] Refactor
- [ ] Docs / Chore

## Checklist
- [ ] Tests unitarios pasan (`dotnet test`)
- [ ] Sin findings Critical/High en security scan
- [ ] Health check responde 200
- [ ] CorrelationId propagado en logs

## Notas para el reviewer
[comentarios por commit + riesgos + casos de prueba manual]
```

---

## Skill Collaboration

Cuando la solicitud requiera capacidades fuera del dominio de este Skill, coordinar con el Skill correspondiente siguiendo `framework/DECISION-TREE.md`.

| Necesidad | Skill |
|---|---|
| Revisar el código del PR | intención `review-code` (skill-map) |
| Publicar el review en GitHub | intención `publish-review` (skill-map) |

---

## Completion Checklist

Antes de considerar terminada cualquier tarea:

- □ Se respetó el Framework (Principles, Workflow, GIT-Policy, Conventions).
- □ Se siguió el Workflow.
- □ Se utilizó el Skill correcto.
- □ Se mantuvieron cambios mínimos.
- □ Se mantuvo consistencia con el proyecto.
- □ Se espera aprobación del desarrollador antes de cualquier operación Git.
