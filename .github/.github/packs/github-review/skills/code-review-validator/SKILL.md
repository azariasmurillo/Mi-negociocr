---
name: code-review-validator
description: "Valida findings de code review contra el código real y los estándares del equipo. Produce tabla de veredictos con plan de acción priorizado. Use when: validar code review, analizar findings, revisar hallazgos de PR, verificar observaciones de reviewer, priorizar correcciones de review, plan de acción de review, qué arreglar del code review, analizar comentarios de PR."
argument-hint: "Pega los findings del code review o indica el PR a analizar"
---

# Code Review Validator

---

## Framework Integration

Este Skill pertenece al AI Engineering Framework — Pack github-review.

Antes de ejecutar cualquier acción debe seguir obligatoriamente:

- `framework/PRINCIPLES.md` — Principios globales que gobiernan todo el ecosistema.
- `framework/WORKFLOW.md` — Flujo de trabajo oficial: comprender, analizar, planificar, implementar, validar, esperar aprobación.
- `framework/DECISION-TREE.md` — Árbol de decisión para selección de Skills.
- `framework/GIT-POLICY.md` — Política oficial para todas las operaciones Git.
- `framework/CONVENTIONS.md` — Convenciones de nombres, estructura y organización.

---

## Framework Role

**Responsabilidad de este Skill:**
Validar cada finding de un code review contra el código fuente real y los estándares del proyecto, produciendo un veredicto y plan de acción priorizados.

**Problema que resuelve:**
Los equipos reciben observaciones de code review que pueden ser válidas, inválidas o mal planteadas — este Skill distingue cuáles implementar, postergar o rechazar.

**Responsabilidades que NO tiene este Skill:**
- No hace code review desde cero (intención `review-code` / `review-code-hard` vía skill-map).
- No publica comentarios en GitHub (intención `publish-review` vía skill-map).
- No realiza operaciones Git.

---

Valida cada finding de un code review contra el código fuente real y los estándares del Project Profile, produciendo un veredicto fundamentado y un plan de acción priorizado.

## Procedimiento

### Fase 1 — Recopilar inputs
1. Obtener la lista de findings del code review.
2. Identificar archivos y líneas referenciadas por cada finding.
3. Clasificar por severidad declarada (🔴 bloqueante, 🟡 medio, 🟢 menor).

### Fase 2 — Validar contra código real
Para **cada finding**:
1. Leer el código actual en el archivo referenciado.
2. Verificar que el reviewer entendió correctamente el patrón.
3. Buscar si otros archivos del repo siguen el mismo patrón.
4. Clasificar:
   - ✅ **Válido** — El código tiene el problema descrito.
   - ⚠️ **Parcialmente válido** — El problema existe pero la solución propuesta es incorrecta.
   - ❌ **Inválido** — El código no tiene el problema o el reviewer malinterpretó.

### Fase 3 — Contrastar con estándares
Para cada finding válido:
1. Leer `.github/instructions/*.instructions.md` del repo.
2. Consultar los estándares de API definidos en el Project Profile (`standards.root`).
3. Determinar si el estándar lo exige (ALTA), sugiere (MEDIA), o no menciona (BAJA).

### Fase 4 — Tabla de veredictos
```markdown
| # | Sev | Finding | Veredicto | ¿Estándar lo exige? | Acción |
|---|-----|---------|-----------|---------------------|--------|
| 1 | 🔴  | [descripción] | ✅/⚠️/❌ | SÍ/NO/Parcial | Implementar / Postergar / Rechazar |
```

### Fase 5 — Plan de acción
1. **Implementar ahora** — Válidos + exigidos por estándar + bajo riesgo de regresión.
2. **Postergar** — Válidos pero no exigidos, o requieren refactoring mayor.
3. **Rechazar** — Inválidos, contradicen el estándar, o se basan en premisas falsas.

## Criterios de decisión

| Criterio | Implementar ahora | Postergar | Rechazar |
|----------|-------------------|-----------|----------|
| ¿El estándar lo exige? | SÍ | No | Contradice |
| ¿El código tiene el bug? | SÍ | SÍ | NO |
| ¿Bajo riesgo? | SÍ | NO | — |
| ¿Cabe en el scope del PR? | SÍ | NO | — |

---

## Skill Collaboration

Cuando la solicitud requiera capacidades fuera del dominio de este Skill, coordinar con el Skill correspondiente siguiendo `framework/DECISION-TREE.md`.

| Necesidad | Intención |
|---|---|
| Hacer el code review completo desde cero | `review-code` / `review-code-hard` |
| Publicar el resultado en GitHub | `publish-review` |

---

## Completion Checklist

Antes de considerar terminada cualquier tarea:

- □ Se respetó el Framework (Principles, Workflow, GIT-Policy, Conventions).
- □ Se siguió el Workflow.
- □ Se utilizó el Skill correcto.
- □ Se mantuvieron cambios mínimos.
- □ Se mantuvo consistencia con el proyecto.
- □ Se espera aprobación del desarrollador antes de cualquier operación Git.
