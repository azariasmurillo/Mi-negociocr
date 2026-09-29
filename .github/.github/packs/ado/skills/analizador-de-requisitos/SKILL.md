---
name: analizador-de-requisitos
description: "Asiste al programador a entender historias en curso analizando documentacion funcional/tecnica y backlog de Azure DevOps en modo solo lectura. Extrae requerimientos clave, dependencias y criterios de aceptacion sin proponer ni ejecutar cambios en DevOps."
argument-hint: "Proporciona projectDocs y devOpsItems; opcionalmente indica contexto, alcance y fecha de corte"
user-invocable: true
---

# Requirements Analyzer

---

## Framework Integration

Este Skill pertenece al AI Engineering Framework — Pack ado.

Antes de ejecutar cualquier acción debe seguir obligatoriamente:

- `framework/PRINCIPLES.md` — Principios globales que gobiernan todo el ecosistema.
- `framework/WORKFLOW.md` — Flujo de trabajo oficial: comprender, analizar, planificar, implementar, validar, esperar aprobación.
- `framework/DECISION-TREE.md` — Árbol de decisión para selección de Skills.
- `framework/GIT-POLICY.md` — Política oficial para todas las operaciones Git.
- `framework/CONVENTIONS.md` — Convenciones de nombres, estructura y organización.

---

## Framework Role

**Responsabilidad de este Skill:**
Analizar documentación funcional/técnica y backlog de ADO en modo solo lectura para extraer requerimientos, dependencias y criterios de aceptación.

**Problema que resuelve:**
Los desarrolladores necesitan entender historias complejas con fuentes dispersas antes de implementar, sin modificar el backlog.

**Responsabilidades que NO tiene este Skill:**
- No crea, actualiza ni elimina work items en Azure DevOps.
- No recomienda acciones de escritura automática sobre el backlog.
- No realiza operaciones Git.

---

## Purpose
Asistir al programador a comprender las historias que esta trabajando, con analisis de contexto y trazabilidad en modo solo lectura.

## Inputs
- `projectDocs`: Documentacion funcional, tecnica, arquitectura, glosario, decisiones y anexos.
- `devOpsItems`: Epicas, features, historias, tareas, bugs, criterios existentes y relaciones en Azure DevOps.
- Base documental para salida en repositorio: `docs/00_plantillas/playbook_documentacion.md`.
- Playbook operativo ADO: `Analisis Épicas/shared/ado_query_playbook.yaml`.

## Outputs
- `requirementsSummary`: Resumen estructurado de requerimientos funcionales y no funcionales.
- `dependencies`: Dependencias tecnicas, de negocio, de datos, equipos y secuencias de entrega.
- `acceptanceCriteria`: Criterios de aceptacion claros, verificables y trazables por requerimiento.

## When To Use
- Cuando hay multiples documentos y backlog disperso que deben reconciliarse.
- Cuando se requiere identificar vacios, ambiguedades o conflictos entre documentacion y DevOps.
- Cuando se necesita preparar insumos para refinamiento, planificacion o validacion con stakeholders.
- Cuando un desarrollador necesita entender rapidamente una historia antes de implementarla.
- Cuando se requiere aclarar alcance tecnico sin modificar elementos de Azure DevOps.

## Read-Only Boundaries
- Esta skill NO crea, actualiza ni elimina work items, comentarios, relaciones, estados ni campos en Azure DevOps.
- Esta skill NO recomienda acciones de escritura automatica sobre backlog o tableros.
- Todo analisis se limita a lectura, interpretacion y sintesis para apoyo al desarrollo.

## Procedure
1. Delimitar alcance temporal y funcional del analisis.
2. Inventariar fuentes en `projectDocs` y `devOpsItems`.
3. Si hay consulta a ADO, ejecutar primero la secuencia recomendada en `Analisis Épicas/shared/ado_query_playbook.yaml`.
4. Extraer requerimientos explicitos e implicitos.
5. Normalizar terminos y eliminar duplicados semanticos.
6. Mapear trazabilidad documento-item-requerimiento.
7. Identificar dependencias criticas y supuestos.
8. Redactar criterios de aceptacion medibles por cada requerimiento.
9. Reportar brechas, conflictos y preguntas abiertas.
10. Si una secuencia alternativa en ADO es mas efectiva, documentarla en `Analisis Épicas/shared/ado_query_playbook.yaml` en la misma sesion.

## ADO Call Optimization Loop
- Minimizar llamadas de descubrimiento repetitivas cuando ya existe un patron validado.
- Priorizar llamadas de mayor precision (iteracion por GUID + batch por IDs) sobre busquedas abiertas.
- Registrar patrones exitosos con evidencia minima (sprint o work items obtenidos).

## Output Contract
Devuelve la respuesta en tres secciones obligatorias:

### requirementsSummary
- Lista priorizada por impacto/urgencia.
- Clasificacion: funcional, no funcional, restriccion, regla de negocio.
- Fuente de origen y referencia cruzada.

### dependencies
- Dependencias internas y externas.
- Tipo: tecnica, datos, proceso, aprobacion, tercero.
- Impacto si no se cumple y mitigacion sugerida.

### acceptanceCriteria
- Formato sugerido: Given/When/Then o checklist verificable.
- Debe ser atomico, testeable y sin ambiguedad.
- Incluir referencia al requerimiento que valida.

## Developer Assistance Focus
- Traducir cada requerimiento a impacto en implementacion (servicios, datos, validaciones, integraciones).
- Explicar dependencias que puedan bloquear desarrollo o pruebas.
- Senalar dudas que el programador debe aclarar antes de codificar.
- Priorizar claridad practica para ejecucion tecnica.

## Quality Rules
- No inferir hechos sin marcar "supuesto".
- Distinguir claramente hechos, interpretacion y recomendacion.
- Senalar inconsistencias con evidencia de fuente.
- Mantener redaccion orientada a validacion y ejecucion.
- Mantener enfoque de apoyo al programador, no de administracion de DevOps.
- Si el analisis se documenta en `docs/`, usar plantilla oficial (`docs/00_plantillas/plantilla_documento_vigente.md` o `docs/00_plantillas/plantilla_borrador.md`).
- Declarar `Estado: vigente` o `Estado: borrador` al inicio del documento.
- Referenciar el nuevo artefacto en `docs/INDEX.md` en la misma sesion.

---

## Skill Collaboration

Cuando la solicitud requiera capacidades fuera del dominio de este Skill, coordinar con el Skill correspondiente siguiendo `framework/DECISION-TREE.md`.

| Necesidad | Skill |
|---|---|
| Generar blueprint de implementación | `generador-blueprint-hu` |
| Refinar la historia para el sprint | `refinador-historias-ado` |
| Evaluar impacto arquitectónico | `arquitecto-de-software` |

---

## Completion Checklist

Antes de considerar terminada cualquier tarea:

- □ Se respetó el Framework (Principles, Workflow, GIT-Policy, Conventions).
- □ Se siguió el Workflow.
- □ Se utilizó el Skill correcto.
- □ Se mantuvieron cambios mínimos.
- □ Se mantuvo consistencia con el proyecto.
- □ Se espera aprobación del desarrollador antes de cualquier operación Git.
