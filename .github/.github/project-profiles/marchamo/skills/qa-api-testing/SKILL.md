---
name: qa-api-testing
description: 'Genera entregables de QA para APIs REST de la plataforma MarchamoDigital: colección Postman importable, environments (Local + AKS Dev) y plan de pruebas con escenarios happy path y error cases. USA CUANDO: generar plan de pruebas, generar colección postman, entregables QA, documentación pruebas API, test cases para QA, exportar endpoints para probar, casos de prueba, generar suite de pruebas, artefactos QA, documento de pruebas, guía de pruebas para QA externo, QA nube AKS, ambiente AKS dev.'
argument-hint: 'Especifica el ID de la HU (ej: US-261779), el microservicio (ej: estados-transaccionales) y los endpoints a incluir.'
---

# QA API Testing — Plataforma MarchamoDigital

---

## Framework Integration

Este Skill pertenece al Project Profile Marchamo del AI Engineering Framework.

Antes de ejecutar cualquier acción debe seguir obligatoriamente:

- `framework/PRINCIPLES.md` — Principios globales que gobiernan todo el ecosistema.
- `framework/WORKFLOW.md` — Flujo de trabajo oficial: comprender, analizar, planificar, implementar, validar, esperar aprobación.
- `framework/DECISION-TREE.md` — Árbol de decisión para selección de Skills.
- `framework/GIT-POLICY.md` — Política oficial para todas las operaciones Git.
- `framework/CONVENTIONS.md` — Convenciones de nombres, estructura y organización.

---

## Framework Role

**Responsabilidad de este Skill:**
Generar todos los artefactos necesarios para que el equipo de QA pruebe una API REST de MarchamoDigital: colección Postman, environments y plan de pruebas con escenarios happy path y error cases.

**Problema que resuelve:**
El equipo de QA necesita artefactos listos para importar y ejecutar en Postman, sin acceso al código fuente ni conocimiento de la infraestructura interna.

**Responsabilidades que NO tiene este Skill:**
- No ejecuta las pruebas (solo genera los artefactos).
- No levanta el ambiente de pruebas (eso es `localhost-api-testing`).
- No realiza operaciones Git.

---

## Descripción
Genera todos los artefactos necesarios para que el equipo de QA pruebe una API REST de MarchamoDigital sin acceso al código fuente.

## Convenciones de plataforma (aplican a TODOS los servicios)

1. **Sin prefijo de versión en URL.** Las rutas se publican como `/api/<servicio>/v{n}/{grupo}/{accion}`.
2. **JSON: camelCase** en request y response (incluyendo `correlationId`).
3. **Header `X-Correlation-ID`** es opcional; si no se envía, el servicio lo genera y retorna en header y body.
4. **Códigos HTTP**: `200 OK` happy path, `400 Bad Request` para validación de payload.
5. **Códigos de error funcionales**: formato `ERR-{PREFIJO}-{NNN}` (ej: `ERR-BJA-002`).
6. **`AllowAnonymous`** es deuda técnica aceptada — documentarlo en el entregable.
7. **Sin secretos en entregables**.

## Convenciones del repo actual

> **AL MIGRAR LA SKILL A OTRO SERVICIO, ACTUALIZAR SOLO ESTA SECCIÓN.**

Ajustar según el microservicio activo:
- **Dominio**: [describir el dominio del servicio]
- **Base URL AKS Dev**: `http://f2bag7c7d6g5bvbn.fz31.alb.azure.com/api/{nombre-servicio}`
- **Base URL Local**: `http://localhost:{puerto}`
- **Identificador principal**: [formato del ID del recurso principal, ej: `CodTagId`]
- **Datos sintéticos para QA**: [IDs válidos para pruebas]
- **Prefijos de error funcional**: [ej: `ERR-BJA-*`, `ERR-VIN-*`]
- **Carpeta de entregables**: `documentation/postman/US-{ID}/`

## Entregables que produce la skill

Para `US-{ID}` con feature corto `{feature}`, genera en `documentation/postman/US-{ID}/`:

| Archivo | Propósito |
|---|---|
| `US-{ID}-{feature}.postman_collection.json` | Colección Postman v2.1 con tests automáticos |
| `US-{ID}-{feature}.postman_environment.json` | Environment AKS Dev |
| `US-{ID}-{feature}.postman_environment.local.json` | Environment Local |
| `ArchivosDraft_HU-{ID}_{Autor}.md` | Draft consolidado QA-friendly (ver skill `qa-efull-draft`) |

## Procedimiento

### 1. Recolectar contexto
- Confirmar ID de la HU y feature corto.
- Identificar endpoints en alcance: método, ruta, request DTO, response DTO, validaciones, códigos de error.

### 2. Generar la colección Postman
Por cada endpoint:
- `name`: nombre humano del endpoint
- `request`: método, url con variables `{{baseUrl}}`, headers (`X-Correlation-ID`, `Content-Type`)
- `body`: JSON válido de ejemplo para happy path
- `tests`: scripts de validación de status code + estructura de response

### 3. Generar environments
```json
{
  "name": "AKS Dev — {Servicio}",
  "values": [
    { "key": "baseUrl", "value": "http://f2bag7c7d6g5bvbn.fz31.alb.azure.com/api/{servicio}" }
  ]
}
```

### 4. Generar el plan de pruebas
Dentro del `ArchivosDraft_HU-*.md`, incluir:
- Happy path por endpoint
- Casos de error funcional (uno por código `ERR-*`)
- Casos de validación de payload (`400`)
- Casos de estado inválido del recurso

### 5. Verificar entregables
- La colección importa sin errores en Postman.
- Todos los environments tienen `baseUrl` como única variable de entorno.
- El plan tiene al menos 1 happy path y 2 error cases por endpoint.

---

## Skill Collaboration

Cuando la solicitud requiera capacidades fuera del dominio de este Skill, coordinar con el Skill correspondiente siguiendo `framework/DECISION-TREE.md`.

| Necesidad | Skill |
|---|---|
| Levantar el servicio para ejecutar las pruebas | `localhost-api-testing` |
| Generar el consolidado QA-friendly de la HU | `qa-efull-draft` |

---

## Completion Checklist

Antes de considerar terminada cualquier tarea:

- □ Se respetó el Framework (Principles, Workflow, GIT-Policy, Conventions).
- □ Se siguió el Workflow.
- □ Se utilizó el Skill correcto.
- □ Se mantuvieron cambios mínimos.
- □ Se mantuvo consistencia con el proyecto.
- □ Se espera aprobación del desarrollador antes de cualquier operación Git.
