---
name: dotnet-architecture-quality
description: "Analiza calidad de código C# .NET como arquitecto senior de soluciones con AKS, Cosmos DB/MongoDB y Azure. Evalúa SOLID, Clean Architecture, patrones async/await, rendimiento en base de datos distribuida, resiliencia, observabilidad y seguridad. USA CUANDO: revisar calidad de código .NET, analizar use cases, revisar repositorios, evaluar patrones de diseño, identificar antipatrones, revisar performance en Cosmos DB o MongoDB, revisar observabilidad, analizar resiliencia, revisar seguridad OWASP en APIs, auditar arquitectura limpia, revisar handlers MediatR, analizar código C# para AKS."
argument-hint: "Pega el código a revisar o indica el(los) archivo(s) a analizar"
---

# .NET Architecture Quality Analyzer

---

## Framework Integration

Este Skill pertenece al AI Engineering Framework — Pack dotnet.

Antes de ejecutar cualquier acción debe seguir obligatoriamente:

- `framework/PRINCIPLES.md` — Principios globales que gobiernan todo el ecosistema.
- `framework/WORKFLOW.md` — Flujo de trabajo oficial: comprender, analizar, planificar, implementar, validar, esperar aprobación.
- `framework/DECISION-TREE.md` — Árbol de decisión para selección de Skills.
- `framework/GIT-POLICY.md` — Política oficial para todas las operaciones Git.
- `framework/CONVENTIONS.md` — Convenciones de nombres, estructura y organización.

---

## Framework Role

**Responsabilidad de este Skill:**
Analizar la calidad del código C# .NET desde la perspectiva de un arquitecto senior, evaluando SOLID, Clean Architecture, async/await, rendimiento en Cosmos DB, resiliencia, observabilidad y seguridad.

**Problema que resuelve:**
Los desarrolladores necesitan feedback objetivo sobre la calidad arquitectónica de su código antes de hacer merge, con hallazgos priorizados por severidad.

**Responsabilidades que NO tiene este Skill:**
- No hace code review de Pull Requests completos (eso es intención `review-code` (skill-map)).
- No publica comentarios en GitHub.
- No realiza operaciones Git.

---

Analiza calidad de código C# .NET desde la perspectiva de un **arquitecto senior** de soluciones cloud-native sobre **Azure Kubernetes Service (AKS)**, usando **Cosmos DB (API Mongo)** como base de datos, con estándares de observabilidad, resiliencia y seguridad enterprise.

## Pilares de evaluación

### P1 — SOLID & Clean Architecture

| Principio | Qué verificar |
|-----------|--------------|
| **SRP** | ¿Cada clase/método tiene una sola razón para cambiar? ¿`Handle` es orquestador delgado? |
| **OCP** | ¿El código es extensible sin modificar? |
| **DIP** | ¿Depende de abstracciones (interfaces) y no de implementaciones concretas? |
| **Capas** | ¿Application depende solo de Domain? ¿No filtra Infrastructure hacia arriba? |

### P2 — Patrones Async/Await (.NET)

| Antipatrón | Impacto |
|------------|---------|
| `.Result` / `.Wait()` | Deadlock en ASP.NET Core |
| `async void` | Excepciones no capturables |
| `CancellationToken` no propagado | Requests zombie en AKS |
| `async` sin `await` | Código síncrono disfrazado |

### P3 — Rendimiento en Cosmos DB / MongoDB (CRÍTICO)

| Antipatrón | Severidad |
|------------|-----------|
| **N+1 Queries** — loop con `GetAsync` individual | 🔴 CRÍTICO |
| **N+1 Writes** — loop con `UpdateAsync` individual | 🔴 CRÍTICO |
| **BulkRead no usado** — usar `GetManyAsync(IEnumerable<string>)` | 🔴 CRÍTICO |
| **BulkWrite no usado** — operaciones batch de MongoDB | 🔴 CRÍTICO |
| **Cross-partition queries** — sin partition key en filtro | 🔴 CRÍTICO |
| Sin paginación en consultas de rango | 🟡 MEDIO |
| `DateTime` vs `DateTimeOffset` inconsistente | 🟡 MEDIO |

#### Regla de oro para loops sobre base de datos
```
❌ MAL — N+1:
for (var i = start; i <= end; i++) {
    var doc = await repo.GetAsync(i.ToString(), ct);
    await repo.UpdateAsync(doc, ct);
}

✅ BIEN — Bulk:
var docs = await repo.GetManyAsync(ids, ct);
await repo.BulkUpdateAsync(docs.Where(d => d.IsEligible).ToList(), ct);
```

### P4 — Resiliencia en AKS

- Circuit breakers para dependencias externas (Polly).
- Retry con backoff exponencial para operaciones idempotentes.
- Health checks en `/health` respondiendo 200 en menos de 2s.
- Timeout explícito en llamadas HTTP salientes.
- Manejo de `OperationCanceledException` en todos los handlers.

### P5 — Observabilidad

- `correlationId` propagado en todos los logs y respuestas.
- Logs con nivel correcto: `Information` para flujo normal, `Warning` para degradación, `Error` solo para excepciones reales.
- No log de datos sensibles (PII, tokens, connection strings).
- `X-Correlation-ID` retornado en el header de respuesta.

### P6 — Seguridad (OWASP Top 10 relevante para APIs .NET)

- Validación de entrada en todos los endpoints (FluentValidation).
- Sin stack traces en respuestas al cliente.
- Sin secrets hardcodeados.
- Headers de seguridad configurados.
- Autorización explícita (no solo `AllowAnonymous` por defecto en producción).

## Formato del informe

```markdown
## Análisis de calidad — {Archivo/Módulo}

### Hallazgos

| # | Pilar | Descripción | Severidad | Recomendación |
|---|-------|-------------|-----------|---------------|
| 1 | P3    | N+1 en loop de etiquetas | 🔴 CRÍTICO | Usar BulkRead |

### Resumen
- 🔴 Críticos: X
- 🟠 Altos: X  
- 🟡 Medios: X
- 🟢 Menores: X

### Código propuesto (para hallazgos críticos)
```

---

## Skill Collaboration

Cuando la solicitud requiera capacidades fuera del dominio de este Skill, coordinar con el Skill correspondiente siguiendo `framework/DECISION-TREE.md`.

| Necesidad | Skill |
|---|---|
| Code review de PR completo | intención `review-code` (skill-map) |
| Implementación .NET | `senior-dotnet` |

---

## Completion Checklist

Antes de considerar terminada cualquier tarea:

- □ Se respetó el Framework (Principles, Workflow, GIT-Policy, Conventions).
- □ Se siguió el Workflow.
- □ Se utilizó el Skill correcto.
- □ Se mantuvieron cambios mínimos.
- □ Se mantuvo consistencia con el proyecto.
- □ Se espera aprobación del desarrollador antes de cualquier operación Git.
