---
name: refinador-historias-ado
description: "Refina historias de usuario del backlog de Azure DevOps aplicando el estandar Scrum (DoR/DoD), mejora la redaccion, define el entregable esperado con alcance explícito (en/fuera de alcance), genera casos de prueba y produce un archivo .md listo para actualizar manualmente el Work Item en ADO."
argument-hint: "Indica el ID de la historia a refinar, por ejemplo: 'refina la historia 261522'"
user-invocable: true
---

# Refinador de Historias para Azure DevOps

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
Aplicar el estándar DoR/DoD a historias de usuario del backlog, mejorar redacción, definir entregables y generar un archivo `.md` listo para que el desarrollador actualice manualmente el Work Item en ADO.

**Problema que resuelve:**
Las historias en el backlog suelen llegar al sprint con ambigüedad, criterios incompletos y sin alcance explícito — este Skill las prepara para cumplir el Definition of Ready.

**Responsabilidades que NO tiene este Skill:**
- NO publica automáticamente en Azure DevOps (el output es para actualización manual).
- NO inventa funcionalidades ni completa vacíos con suposiciones.
- No realiza operaciones Git.

---

## Propósito

Dado el ID de una historia de usuario del proyecto (Project Profile / ADO), este skill:

1. Extrae el detalle completo de la historia desde los archivos locales del repositorio.
2. Aplica el estándar de refinamiento ágil (DoR/DoD) para mejorar claridad y estructura.
3. Define explícitamente qué tipo de entregable produce la historia y sus pautas de calidad.
4. Genera un archivo `.md` listo para que el desarrollador actualice manualmente el Work Item en ADO.

El output **no se publica automáticamente** en Azure DevOps. Es un insumo de refinamiento para revisión antes del sprint.

---

## Cuándo usar este skill

- Cuando el desarrollador dice: _"refina la historia XXXXX"_
- Cuando se va a planificar un sprint y las historias necesitan estar en DoR.
- Cuando el equipo detecta ambigüedad en descripción o criterios de aceptación de una historia.
- Cuando se necesita alinear a todos los interesados sobre qué entrega exactamente una historia.

---

## Fuentes de información a consultar

1. `Analisis Épicas/stories/<sprint>-md/story_<id>.yaml` — detalle completo del work item local.
2. `Analisis Épicas/stories/pbi_userstory_detailed_index.yaml` — índice para localizar el sprint correcto.
3. `Analisis Épicas/stories/ai_pbi_userstory_conceptual_matrix.yaml` — dependencias conceptuales, feature y épica padre.
4. `Analisis Épicas/epics/epic_<id>_<codigo>.yaml` — contexto de la épica padre.
5. `Analisis Épicas/shared/architecture_baseline.yaml` — restricciones técnicas y guardrails de diseño.
6. `docs/40_disenos_y_propuestas/` — verificar si ya existe espacio de trabajo previo para la historia.

---

## Procedimiento paso a paso

### PASO 1 — Localizar la historia

1. Buscar el ID en `pbi_userstory_detailed_index.yaml` para identificar el sprint al que pertenece.
2. Leer `story_<id>.yaml` del sprint correspondiente para extraer:
   - Título
   - Descripción (`description_text`)
   - Criterios de aceptación (`acceptance_criteria_text`)
   - Feature padre (`parent_feature_id`)
   - Tags, estado, sprint
3. Leer la entrada de la historia en `ai_pbi_userstory_conceptual_matrix.yaml` para obtener:
   - Épica padre con código y título
   - Historias peer de la misma feature
   - Sistemas y equipos dependientes

### PASO 2 — Clasificar el tipo de historia

Antes de refinar, determinar la naturaleza de la historia:

| Tipo | Indicadores | Entregable esperado |
|---|---|---|
| **Arquitectónica / de diseño** | No genera código de negocio directo; establece contratos, reglas o restricciones que otros componentes deben cumplir | Documento técnico de diseño (.md) + diagramas |
| **Funcional / de implementación** | Genera código, endpoints, lógica de negocio, UI | Código + pruebas + documentación técnica |
| **De datos / modelo** | Define o modifica entidades, tablas, relaciones de BD | DDL + diccionario de datos + documento de diseño |
| **De integración** | Conecta componentes o sistemas externos | Especificación de contrato + código de integración + pruebas |
| **No funcional** | Performance, seguridad, observabilidad, escalabilidad | Documento de lineamientos + evidencia de validación |

### PASO 3 — Aplicar refinamiento (estándar DoR)

Aplicar las siguientes reglas:

**Reglas de mejora de redacción:**
- Mantener estrictamente el contexto original; NO agregar ni inventar funcionalidades.
- Una sola intención por historia.
- Un solo tipo de usuario/actor.
- Lenguaje claro, simple y no técnico en la descripción.
- Si hay ambigüedad, indicarla explícitamente y mejorar solo con la información existente.

**Formato obligatorio de la historia refinada:**

```
# <Número> <Título>

## Descripción
Como [tipo de usuario/sistema],
quiero [acción concreta],
para [beneficio claro y observable].

## Criterios de aceptación
**CA-N — <Título del criterio>**
Dado que ...
Cuando ...
Entonces ...

## Casos de prueba
### Casos positivos (máx. 3)
- ...

### Casos negativos (máx. 3)
- ...

## Dependencias identificadas
| Dependencia | Historia / Componente | Tipo |
|---|---|---|
| ... | ... | Bloqueante / Fuerte / Relacionada |
```

### PASO 4 — Definir el entregable explícito

Esta es la sección crítica que alinea expectativas. Para cada historia generar:

**4.1 Tipo de entregable**
Declarar en una línea qué tipo de entregable produce la historia (ver tabla del PASO 2).

**4.2 Qué DEBE contener el entregable (en alcance)**
Tabla con los elementos obligatorios numerados. Cada elemento debe ser concreto y verificable.

**4.3 Qué NO está en el alcance**
Tabla con exclusiones explícitas + la historia o componente donde sí se atiende.

**4.4 Pautas de calidad del entregable**
Lista de condiciones que debe cumplir el entregable para ser considerado aprobable:
- Lenguaje no ambiguo
- Verificable contra implementación
- Sin suposiciones implícitas
- Aprobado por los roles correctos
- Versionado en el repositorio

**4.5 Criterio de Done (DoD aplicado a esta historia)**
Checklist específico para esta historia. Formato: `- [ ] Condición verificable`.

### PASO 5 — Generar el archivo de salida

Crear el archivo en:
```
docs/40_disenos_y_propuestas/PBI-<id>-<short-name>/historia_ado_refinada.md
```

Si el directorio del PBI no existe, crearlo antes siguiendo el protocolo del skill `agent-customization`.

El archivo debe tener esta estructura:

> **Regla de encabezado:** el título del archivo SIEMPRE debe incluir el nombre exacto de la historia tal como aparece en ADO (campo `title` del YAML), NO la frase genérica "Texto definitivo para ADO". Formato obligatorio: `# Historia Refinada — US#<ID> — <Título exacto de la historia en ADO>`

```markdown
# Historia Refinada — US#<ID> — <Título exacto de la historia en ADO>

**Fecha de refinamiento:** YYYY-MM-DD
**Work Item ADO:** https://dev.azure.com/<ado.org>/_workitems/edit/<ID>
**Feature padre:** <id> — <nombre>
**Sprint:** <sprint>

---

## FASE 1 — Historia mejorada

[historia refinada completa]

---

## Entregable esperado — Definición explícita

[secciones 4.1 a 4.5]

---

## Notas de refinamiento

[observaciones: si la historia es demasiado grande, si hay particionamiento recomendado, si tiene dependencias bloqueantes, etc.]
```

---

## Señales de historia demasiado grande

Si se detectan **al menos 2** de las siguientes señales, proponer particionamiento vertical antes de refinar:

- Más de un actor u objetivo
- Múltiples integraciones en paralelo
- Cambios simultáneos en UI + Backend + Base de datos
- Flujos alternos complejos
- Dependencias críticas no resueltas
- Migraciones de datos incluidas

**Orden de particionamiento (si aplica):**
1. Flujo principal (caso feliz)
2. Validaciones básicas
3. Casos de error simples
4. Integraciones
5. Requerimientos no funcionales

Máximo 5–7 historias resultantes. Cada una debe ser independiente, entregar valor observable y ser estimable (<21 SP).

---

## Restricciones y límites del skill

- **NO publica automáticamente** en Azure DevOps. El output es para actualización manual.
- **NO inventa** funcionalidades ni completa vacíos con suposiciones.
- **NO evalúa** capacidad del equipo ni hace compromisos de sprint.
- **NO repite** definiciones que ya existen en otro documento; las referencia con link.
- Si hay información insuficiente en los archivos locales, indica explícitamente qué falta.

---

## Ejemplo de invocación

> "refina la historia 261522"
> "haz el refinamiento de la historia 261517"
> "quiero el texto definitivo para ADO de la historia 261523"

El agente localiza automáticamente el archivo de la historia, aplica el procedimiento completo y genera `historia_ado_refinada.md` en el espacio de trabajo del PBI.

---

## Skill Collaboration

Cuando la solicitud requiera capacidades fuera del dominio de este Skill, coordinar con el Skill correspondiente siguiendo `framework/DECISION-TREE.md`.

| Necesidad | Skill |
|---|---|
| Analizar requerimientos de la historia | `analizador-de-requisitos` |
| Generar el blueprint técnico tras el refinamiento | `generador-blueprint-hu` |

---

## Completion Checklist

Antes de considerar terminada cualquier tarea:

- □ Se respetó el Framework (Principles, Workflow, GIT-Policy, Conventions).
- □ Se siguió el Workflow.
- □ Se utilizó el Skill correcto.
- □ Se mantuvieron cambios mínimos.
- □ Se mantuvo consistencia con el proyecto.
- □ Se espera aprobación del desarrollador antes de cualquier operación Git.
