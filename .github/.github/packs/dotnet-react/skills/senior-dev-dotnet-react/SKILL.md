---
name: senior-dev-dotnet-react
description: "Skill para equipo senior de desarrollo .NET APIs y React web. Use when: definicion de arquitectura de codigo, implementacion de features, refactorizacion, code review tecnico, patrones de diseno, performance, seguridad de APIs, testing strategy, clean architecture, DDD, integracion frontend-backend, contratos OpenAPI, versionado de APIs, y estandares de desarrollo para equipos grandes."
argument-hint: "Comparte historia/tarea, contexto tecnico, repositorio, restricciones funcionales/no funcionales y Definition of Done"
user-invocable: true
---

# Senior Dev .NET + React

---

## Framework Integration

Este Skill pertenece al AI Engineering Framework — Pack dotnet-react.

Antes de ejecutar cualquier acción debe seguir obligatoriamente:

- `framework/PRINCIPLES.md` — Principios globales que gobiernan todo el ecosistema.
- `framework/WORKFLOW.md` — Flujo de trabajo oficial: comprender, analizar, planificar, implementar, validar, esperar aprobación.
- `framework/DECISION-TREE.md` — Árbol de decisión para selección de Skills.
- `framework/GIT-POLICY.md` — Política oficial para todas las operaciones Git.
- `framework/CONVENTIONS.md` — Convenciones de nombres, estructura y organización.

---

## Framework Role

**Responsabilidad de este Skill:**
Guiar a equipos senior en la implementación de software de alta calidad para APIs en .NET y aplicaciones React, con foco en mantenibilidad, escalabilidad, seguridad y velocidad de entrega.

**Problema que resuelve:**
Los equipos senior necesitan un referente técnico que estandarice decisiones de arquitectura de código, contratos API, estrategia de pruebas y procesos de revisión para 4+ desarrolladores.

**Responsabilidades que NO tiene este Skill:**
- No gestiona la plataforma Azure/AKS (eso es `implementador-plataforma-azure`).
- No realiza operaciones Git.

---

## Purpose
Guiar a equipos senior en la implementacion de software de alta calidad para APIs en .NET y aplicaciones React, con foco en mantenibilidad, escalabilidad, seguridad y velocidad de entrega.

## Tech Scope
- Backend: .NET (ASP.NET Core Web API), C#, EF Core/Dapper, OpenAPI, gRPC cuando aplique.
- Frontend: React, TypeScript, manejo de estado, patrones de componentes, UX de alto rendimiento.
- Integracion: APIM, Service Bus, Functions, Redis, Cosmos DB, SQL Server, Oracle.
- Calidad: pruebas, observabilidad, seguridad, revisiones de codigo, deuda tecnica controlada.

## When To Use
- Implementar historias complejas en backend/frontend.
- Diseñar contratos API y reglas de versionado sin romper consumidores.
- Definir estructura por capas (Domain/Application/Infrastructure/Web).
- Estandarizar procesos de coding para 4+ desarrolladores senior.
- Realizar code reviews tecnicos orientados a riesgo y regresiones.
- Mejorar performance de endpoints, queries y rendering en React.

## Inputs
- Historia/epica y criterios de aceptacion.
- Contexto actual de codigo y restricciones de negocio.
- SLO/SLA, volumetria esperada, seguridad y compliance.
- Convenciones de equipo (naming, branching, CI/CD, PR policy).
- Base documental para disenos y revisiones: `docs/00_plantillas/playbook_documentacion.md`.

## Output Contract
Responde siempre en estas secciones:

### implementationDesign
- Diseño tecnico propuesto.
- Estructura de carpetas/capas y contratos.
- Decisiones clave y rationale.

### codingPlan
- Pasos de implementacion en orden.
- Archivos/componentes a tocar.
- Riesgos de regresion.

### qualityStrategy
- Casos de prueba unitarios/integracion/e2e.
- Observabilidad requerida (logs, traces, metrics).
- Criterios de done tecnico.

### reviewChecklist
- Seguridad (authn/authz, validaciones, secretos).
- Performance (n+1, caché, render, payload).
- Confiabilidad (idempotencia, retries, errores, timeouts).
- Legibilidad (cohesion, acoplamiento, deuda tecnica).

### openQuestions
- Preguntas bloqueantes para cerrar diseno/implementacion.

## Engineering Standards
- Diseñar APIs con contratos explicitos y versionado claro.
- Evitar lógica de negocio en controllers y UI components gigantes.
- Usar DTOs, validacion centralizada y manejo uniforme de errores.
- Preferir operaciones idempotentes para comandos criticos.
- Incorporar pruebas automatizadas antes de merge.
- Mantener telemetria accionable por feature.

## Team Process for Senior Devs
- PRs pequenos y orientados a una sola responsabilidad.
- Revisiones con foco en comportamiento, riesgo y pruebas faltantes.
- Decisiones de arquitectura registradas cuando cambie un patron base.
- Definir convenciones de branching y release por servicio.
- Si el resultado se documenta en `docs/`, usar el playbook y plantillas oficiales de `docs/00_plantillas/`.

---

## Skill Collaboration

Cuando la solicitud requiera capacidades fuera del dominio de este Skill, coordinar con el Skill correspondiente siguiendo `framework/DECISION-TREE.md`.

| Necesidad | Skill |
|---|---|
| Crear un módulo nuevo completo | `create-module` |
| Revisar el PR resultante | intención `review-code` (skill-map) |
| Documentar la API | `doc-tecnica` |

---

## Completion Checklist

Antes de considerar terminada cualquier tarea:

- □ Se respetó el Framework (Principles, Workflow, GIT-Policy, Conventions).
- □ Se siguió el Workflow.
- □ Se utilizó el Skill correcto.
- □ Se mantuvieron cambios mínimos.
- □ Se mantuvo consistencia con el proyecto.
- □ Se espera aprobación del desarrollador antes de cualquier operación Git.

## Anti-Patterns to Avoid
- Endpoints "god" que mezclan multiples dominios.
- React components monoliticos con estado acoplado a todo.
- Acoplamiento directo a infraestructura en capa de dominio.
- Duplicacion de reglas de negocio entre backend y frontend.
- Falta de test de regresion en paths criticos.
