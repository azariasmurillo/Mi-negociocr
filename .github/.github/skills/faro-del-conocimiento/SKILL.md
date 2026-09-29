---
name: faro-del-conocimiento
description: "Skill de curaduria para mantener orden, claridad y sentido en un repositorio tipo centro de conocimiento. Use when: organizar documentacion, mejorar legibilidad, definir estructura, estandarizar naming, reforzar trazabilidad, limpiar duplicados, aplicar buenas practicas de knowledge base, y mantener consistencia entre indice, reglas y contenido."
argument-hint: "Comparte objetivo (orden, limpieza, reestructura o auditoria), alcance (carpetas/archivos), restricciones y nivel de profundidad"
user-invocable: true
---

# Faro del Conocimiento

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
Actuar como curador del repositorio de conocimiento para mantener una estructura fácil de leer, fácil de mantener y alineada con buenas prácticas de documentación técnica.

**Problema que resuelve:**
A medida que el repositorio crece, pierde coherencia estructural — este Skill detecta duplicados, inconsistencias y propone una arquitectura de información sostenible.

**Responsabilidades que NO tiene este Skill:**
- No genera documentación de código fuente (eso es un Skill del Pack docs / Profile).
- No toma decisiones arquitectónicas de software.
- No realiza operaciones Git.
- No impone paths de un proyecto concreto; usa `docs.layout` del Project Profile.

---

## Purpose
Actuar como curador de un repositorio de conocimiento para mantener una estructura facil de leer, facil de mantener y alineada con buenas practicas de documentacion tecnica.

## When To Use
- Cuando el repositorio crece y empieza a perder coherencia de estructura.
- Cuando existen documentos duplicados, ambiguos o desactualizados.
- Cuando se requiere mejorar navegacion para humanos y agentes.
- Cuando hay que alinear reglas, indices, glosario y artefactos por epica/historia.
- Cuando se necesita proponer una taxonomia estable para largo plazo.

## Inputs
- Estructura actual de carpetas y archivos.
- Reglas vigentes del repositorio (gobernanza y precedencia).
- Objetivo de curaduria (orden, simplificacion, trazabilidad, estandares).
- Nivel de intervencion permitido (solo diagnostico o diagnostico + cambios).
- Project Profile activo (`docs.layout`, plantillas, playbooks).

## Output Contract
Responder siempre en estas secciones:

### repositoryReadingMap
- Ruta de lectura recomendada para onboarding rapido.
- Orden minimo obligatorio y orden profundo por tema.
- Puntos de entrada para humanos y para agentes.

### informationArchitecture
- Propuesta de estructura objetivo (carpetas, roles y fronteras).
- Reglas de ubicacion de documentos por tipo.
- Convenciones de nombres y versionado documental.

### contentQualityFindings
- Hallazgos priorizados: duplicidad, inconsistencia, vacios, ruido, obsolescencia.
- Evidencia por archivo y severidad.
- Riesgo de no corregir cada hallazgo.

### curationPlan
- Plan por fases: quick wins, estabilizacion y mejora continua.
- Acciones concretas por archivo o carpeta.
- Criterios de exito verificables.

### governanceChecks
- Verificacion de cumplimiento contra reglas, indices y convenciones del proyecto.
- Alertas de trazabilidad rota (indice no apunta, resumen ausente, fuente no reconciliada).
- Recomendaciones para prevenir regresion documental.
- Si el Project Profile define playbooks de consulta a backlog, validar que esten actualizados.

### openQuestions
- Preguntas bloqueantes para cerrar decisiones de estructura o contenido.

## Curation Heuristics
- Un documento debe tener una funcion clara y unica.
- Todo archivo importante debe ser descubrible desde un indice.
- Toda fuente extensa debe tener resumen estructurado y trazable.
- Evitar carpetas cajon de sastre sin criterio explicito.
- Priorizar consistencia semantica sobre perfeccion cosmetica.
- Registrar supuestos cuando no exista evidencia.

## Naming and Structure Rules
- Nombres descriptivos, cortos y estables.
- Fechas en formato YYYY-MM-DD cuando aplique.
- Separar claramente: contexto, arquitectura, backlog, historias, decisiones y anexos.
- Mantener plantillas para analisis repetibles.
- Minimizar variaciones de formato sin justificacion.
- Usar las plantillas de documento vigente/borrador definidas por el Project Profile.
- Declarar `Estado: vigente` o `Estado: borrador` al inicio de todo .md documental cuando el proyecto lo requiera.

## Anti-Patterns to Avoid
- Multiples fuentes canonicas para el mismo tema.
- Indices que no reflejan el estado real del repositorio.
- Analisis sin trazabilidad a documento fuente o item origen.
- Mezclar decisiones finales con borradores sin etiquetado.
- Crear nuevas carpetas sin criterio de largo plazo.

## Definition of Done for Curation
- Existe una ruta de lectura clara para onboarding.
- Los documentos clave tienen ubicacion canonica y enlaces coherentes.
- Se reducen duplicados y se explicita que version es vigente.
- El indice principal permite navegar por dominios sin ambiguedad.
- Quedan reglas simples para sostener el orden en futuras iteraciones.
- Playbooks del Project Profile (si existen) reflejan los patrones exitosos usados.

---

## Skill Collaboration

Cuando la solicitud requiera capacidades fuera del dominio de este Skill, coordinar con el Skill correspondiente siguiendo `framework/DECISION-TREE.md` y el skill-map del Project Profile.

| Necesidad | Intención |
|---|---|
| Generar documentación técnica nueva | `documentation` |
| Exportar documentos a Word | según skill-map |

---

## Completion Checklist

Antes de considerar terminada cualquier tarea:

- □ Se respetó el Framework (Principles, Workflow, GIT-POLICY, Conventions).
- □ Se siguió el Workflow.
- □ Se utilizó el Skill correcto.
- □ Se consultó el Project Profile para layout y plantillas.
- □ Se mantuvieron cambios mínimos.
- □ Se mantuvo consistencia con el proyecto.
- □ Se espera aprobación del desarrollador antes de cualquier operación Git.
