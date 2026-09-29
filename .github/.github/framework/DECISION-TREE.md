# Framework Decision Tree

## Objetivo

Este documento define cómo el agente debe decidir qué Agent Skills utilizar para resolver una solicitud.

El Decision Tree trabaja por **intención**, no por nombres de Skills.

Los Agent Skills no deben contener árboles de decisión complejos.

Toda la lógica de selección por intención pertenece a este documento.

La resolución de intención → Skill concreto pertenece al **skill-map del Project Profile** y a los Packs instalados.

---

# Principio

Antes de generar código:

1. Comprender la solicitud.
2. Clasificar la **intención**.
3. Resolver la intención al Skill concreto (Project Profile + Packs).
4. Ejecutar únicamente los Skills necesarios.

Nunca ejecutar múltiples Skills si uno solo resuelve la solicitud.

Nunca hardcodear nombres de Skills de un proyecto en este documento.

---

# Resolución

```
Solicitud

↓

Clasificar intención (este documento)

↓

Consultar skill-map del Project Profile activo

↓

Consultar Packs instalados

↓

Cargar únicamente el Skill resuelto
```

Si la intención no tiene Skill mapeado:

1. Buscar el Skill más cercano en Core y Packs instalados.
2. Si no existe, responder con el Workflow general.
3. No inventar un Skill.

---

# Árbol de decisión por intención

## Arquitectura

Si la solicitud implica:

- Diseño
- Arquitectura
- Historia de Usuario
- Nueva funcionalidad
- Análisis técnico
- Diseño de solución

↓

**Intención:** `architecture`

---

## Crear un módulo

Si la solicitud implica:

- Nuevo módulo
- Nuevo CRUD
- Nueva entidad
- Nueva funcionalidad completa

↓

**Intención:** `create-module`

---

## Agregar funcionalidad

Si la solicitud implica:

- Nuevo endpoint
- Nueva operación
- Nueva consulta
- Nueva acción

↓

**Intención:** `add-operation`

---

## Nueva versión

Si la solicitud implica:

- Versionar API
- Crear V2
- Crear V3
- Mantener compatibilidad

↓

**Intención:** `new-version`

---

## Revisar código

Si la solicitud implica:

- Code Review
- Pull Request Review
- Revisar cambios
- Revisar calidad

↓

**Intención:** `review-code`

---

## Revisión profunda de PR

Si la solicitud implica:

- Análisis técnico profundo de un Pull Request
- Criterios exigentes de mantenibilidad y escalabilidad
- Hallazgos con impacto en producción o deuda técnica seria
- Review nivel Staff o Principal Engineer

↓

**Intención:** `review-code-hard`

---

## Validar findings de review

Si la solicitud implica:

- Validar observaciones de un code review
- Verificar si un finding es correcto o incorrecto
- Priorizar correcciones de review
- Plan de acción a partir de hallazgos existentes

↓

**Intención:** `validate-review-findings`

---

## Publicar review

Si la solicitud implica:

- Publicar un review en la plataforma
- Comentar un PR con hallazgos ya generados

↓

**Intención:** `publish-review`

---

## Revisión transversal

Si la solicitud afecta:

- Más de un repositorio
- Más de un microservicio
- Contratos compartidos
- Integraciones

↓

**Intención:** `cross-repo-review`

---

## Corregir Build

Si existen errores de compilación.

↓

**Intención:** `fix-build`

---

## Corregir Tests

Si existen errores de pruebas.

↓

**Intención:** `fix-tests`

---

## Documentación

Si la solicitud implica:

- README
- ADR
- Diagramas
- Manuales
- Documentación técnica, funcional, de procesos o de arquitectura

↓

**Intención:** `documentation`

---

## Blueprint de historia

Si la solicitud implica:

- Generar blueprint de una historia
- Preparar contexto de implementación desde una HU

↓

**Intención:** `generate-blueprint`

---

## Modelado de datos

Si la solicitud implica:

- Modelo ER
- DDL
- Diccionario de datos
- Diseño relacional

↓

**Intención:** `data-modeling`

---

## QA / pruebas de API

Si la solicitud implica:

- Plan de pruebas
- Colección Postman
- Pruebas de API
- Pruebas en localhost

↓

**Intención:** `api-testing`

---

## Implementación

Si la solicitud implica:

- Implementar código
- Refactorizar
- Desarrollar una TASK aprobada

↓

**Intención:** `implement`

---

## Plataforma / infraestructura

Si la solicitud implica:

- AKS, pipelines, ingress, cloud
- Configuración de plataforma

↓

**Intención:** `platform`

---

## Gestión de trabajo (ADO / backlog)

Si la solicitud implica:

- Refinar historias
- Registrar tareas
- Reportes de trabajo
- Análisis de requisitos en backlog

↓

**Intención:** `work-management`

---

## Conocimiento / curaduría

Si la solicitud implica:

- Organizar conocimiento
- Curar documentación del repo
- Estructura de entregables por PBI/HU

↓

**Intención:** `knowledge`

---

## Operaciones Git / PR

Si la solicitud implica:

- Crear PR
- Gestionar ramas
- Generar contenido de PR desde commits

↓

**Intención:** `git-pr`

---

## Entorno

Si la solicitud implica:

- Codespaces
- Dev Containers
- Configuración del entorno de desarrollo

↓

**Intención:** `environment`

---

# Solicitudes mixtas

Cuando una solicitud requiera varios pasos:

Seleccionar una intención principal.

Esa intención podrá utilizar otros Skills únicamente si es estrictamente necesario.

No ejecutar múltiples Skills por defecto.

---

# Prioridad

Cuando varias intenciones sean válidas:

1. Utilizar la intención más específica.
2. Evitar intenciones genéricas.
3. Evitar duplicación.
4. Reutilizar el trabajo existente.

---

# Resultado esperado

Todo Agent Skill debe comenzar porque fue seleccionado mediante:

1. Una intención de este árbol.
2. Una resolución en el skill-map del Project Profile / Packs.

Nunca por elección arbitraria.
