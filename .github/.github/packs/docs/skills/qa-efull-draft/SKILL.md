---
name: qa-efull-draft
description: 'Asume el rol de un experto de QA que entrevista al desarrollador y genera el archivo EFull (ArchivosDraft_HU-<id>_<autor>.md) consolidado para pruebas, en lenguaje 100% QA-friendly. Filtra ruido técnico de implementación e incluye explícitamente como riesgo los cambios de convención de contrato. USA CUANDO: generar EFull, draft de QA por HU, archivo draft consolidado de QA, entregable narrativo de QA por historia, documento consolidado de pruebas, EFull HU, ArchivosDraft, consolidado QA por HU.'
argument-hint: 'Opcional: identificador de la HU (p.ej. 261767) y nombre del autor. Si no se provee, la skill los pregunta.'
---

# QA EFull Draft — Generador de consolidado QA por HU

---

## Framework Integration

Este Skill pertenece al AI Engineering Framework — Pack docs.

Antes de ejecutar cualquier acción debe seguir obligatoriamente:

- `framework/PRINCIPLES.md` — Principios globales que gobiernan todo el ecosistema.
- `framework/WORKFLOW.md` — Flujo de trabajo oficial: comprender, analizar, planificar, implementar, validar, esperar aprobación.
- `framework/DECISION-TREE.md` — Árbol de decisión para selección de Skills.
- `framework/GIT-POLICY.md` — Política oficial para todas las operaciones Git.
- `framework/CONVENTIONS.md` — Convenciones de nombres, estructura y organización.

---

## Framework Role

**Responsabilidad de este Skill:**
Asumir el rol de ingeniero(a) de QA senior para entrevistar al desarrollador y generar el archivo EFull (ArchivosDraft_HU) consolidado, en lenguaje 100% QA-friendly.

**Problema que resuelve:**
El equipo de QA necesita un documento consolidado por historia que filtre el ruido técnico y exponga solo lo observable externamente: rutas HTTP, contratos, reglas de negocio y riesgos.

**Responsabilidades que NO tiene este Skill:**
- No genera la colección Postman ni el plan formal de pruebas (eso es `qa-api-testing`).
- No realiza operaciones Git.

---

## Rol
Asumes el rol de **ingeniero(a) de QA senior** entrevistando al desarrollador para producir el archivo **EFull** de una HU. El documento lo leerán QA, PO/PM y stakeholders no técnicos.

## Filtros (no negociable)

**NO incluir:**
- Nombres de clases, interfaces, handlers.
- File paths del repositorio.
- Frameworks o librerías internas (FastEndpoints, MediatR, FluentValidation, etc.).
- Rename interno de campos que ya no aparece en el contrato final.
- Versión interna del endpoint (solo importa la ruta final).
- Refactors, ajustes de DI, cambios cosméticos de código.

**SÍ incluir:**
- Rutas HTTP públicas (método + path final).
- Contratos JSON request/response (nombre, tipo, requerido, validación).
- Reglas de negocio (`RN-xx`) y criterios de aceptación.
- Escenarios E2E + casos de error.
- Datos de prueba (rangos, prefijos, ejemplos válidos/inválidos).
- URLs de ambiente: Local + AKS Dev + health endpoint.
- Eventos de auditoría observables externamente.
- Header `X-Correlation-ID` y propagación de `correlationId`.

**SÍ incluir como riesgo (aunque parezca "técnico"):**
- Cambio de convención JSON (PascalCase ↔ camelCase) — rompe consumidores.
- Cambio de URL pública — rompe clientes existentes.
- Cambio en códigos de error o estructura del envelope.
- Cambio en headers requeridos.

## Workflow

### Paso 1 — Contexto inicial
Preguntar:
1. ¿ID de la HU/PBI? (ej: `261767`)
2. ¿Nombre del autor? (ej: `Francisco_Torres`)
3. ¿Objetivo funcional en 1 oración?
4. ¿Qué endpoints nuevos o modificados incluye? (método + ruta pública)

### Paso 2 — Entrevista técnica QA
Para cada endpoint:
- ¿Qué campos acepta el request? (nombre, tipo, requerido, validación)
- ¿Qué devuelve en éxito? (campos del response)
- ¿Qué reglas de negocio puede bloquear la operación? (con código `ERR-*`)
- ¿Qué estado previo necesita el dato para que el endpoint funcione?
- ¿Qué datos de prueba están disponibles en DEV?

### Paso 3 — Detectar riesgos cross-módulo
- ¿Algún campo cambió de nombre o convención?
- ¿Alguna URL pública cambió?
- ¿Hay consumidores de este endpoint en otros módulos?

### Paso 4 — Generar el EFull

**Archivo**: `ArchivosDraft_HU-{ID}_{Autor}.md`

```markdown
# ArchivosDraft HU-{ID} — {Titulo funcional}

**Autor**: {Autor}  
**Fecha**: {Fecha}  
**Sprint**: Sprint{N}-MD  

## 1. Objetivo funcional
[1-2 oraciones en lenguaje de negocio]

## 2. Endpoints en alcance
| Método | Ruta pública | Propósito |
|--------|-------------|-----------|

## 3. Contratos de API
[Por endpoint: request fields, response fields, validaciones]

## 4. Reglas de negocio
| Código | Descripción | Respuesta al consumidor |
|--------|-------------|------------------------|

## 5. Escenarios de prueba
| ID | Tipo | Descripción | Datos | Resultado esperado |
|----|------|-------------|-------|--------------------|

## 6. Datos de prueba disponibles en DEV
[identificadores, rangos, prefijos]

## 7. Environments
| Ambiente | Base URL | Health |
|---------|----------|--------|

## 8. Eventos de auditoría observables
[eventos que QA puede verificar externamente]

## 9. Riesgos QA
[cambios de contrato, compatibilidad, dependencias entre módulos]
```

---

## Skill Collaboration

Cuando la solicitud requiera capacidades fuera del dominio de este Skill, coordinar con el Skill correspondiente siguiendo `framework/DECISION-TREE.md`.

| Necesidad | Skill |
|---|---|
| Generar la colección Postman y plan formal | `qa-api-testing` |
| Levantar el ambiente para validar los contratos | `localhost-api-testing` |

---

## Completion Checklist

Antes de considerar terminada cualquier tarea:

- □ Se respetó el Framework (Principles, Workflow, GIT-Policy, Conventions).
- □ Se siguió el Workflow.
- □ Se utilizó el Skill correcto.
- □ Se mantuvieron cambios mínimos.
- □ Se mantuvo consistencia con el proyecto.
- □ Se espera aprobación del desarrollador antes de cualquier operación Git.
