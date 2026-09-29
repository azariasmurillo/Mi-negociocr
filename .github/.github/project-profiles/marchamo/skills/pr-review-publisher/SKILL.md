---
name: pr-review-publisher
description: "Publica en GitHub la review de un PR del ecosistema MarchamoDigital a partir de un reporte local generado por la skill `pr-review-marchamo`. Crea (1) un comentario general resumen en el PR y (2) un comentario inline por cada hallazgo, incluyendo una propuesta concreta de solución (suggestion block o bloque de código). USE WHEN: publicar review en GitHub, comentar PR, dejar feedback en PR, subir el reporte de review al PR, comentar inline cambios."
argument-hint: "Indica el reporte local a publicar y el PR destino, por ejemplo: 'publica el review pr-review-63-HU261529-2026-05-12.md en el PR #63 de MarchamoDigital.Auditoria.Correlacion'"
user-invocable: true
---

# PR Review Publisher — Ecosistema MarchamoDigital

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
Publicar en GitHub un reporte de revisión local previamente generado por `pr-review-marchamo` y aprobado por el desarrollador, como comentario general e inline comments con propuestas de solución.

**Problema que resuelve:**
El desarrollador necesita publicar los hallazgos del code review en el PR de GitHub con formato estructurado, incluyendo suggestion blocks, sin hacerlo manualmente.

**Responsabilidades que NO tiene este Skill:**
- No hace code review desde cero (eso es `pr-review-marchamo`).
- Nunca aprueba ni rechaza el PR automáticamente — solo publica comentarios (event: COMMENT).
- Requiere confirmación explícita del desarrollador antes de cualquier escritura en GitHub (ver `framework/GIT-POLICY.md`).

---

## Propósito

Tomar un reporte de revisión local — generado previamente por la skill **`pr-review-marchamo`** y validado/negociado por el usuario — y publicarlo en el Pull Request correspondiente de GitHub (`Grupo-INS/MarchamoDigital.*`) como:

1. **Un comentario general** (issue comment a nivel de PR) con el resumen ejecutivo, veredicto y conteo de hallazgos.
2. **Un comentario inline por cada hallazgo**, anclado al archivo y línea afectados, incluyendo:
   - Descripción del hallazgo y severidad.
   - Regla violada (ID + nombre de la categoría: A, B, C, D, E, F, G, H, I/S).
   - **Propuesta de solución**: bloque `suggestion` de GitHub (cuando el cambio es de una sola línea o un rango contiguo) o un bloque ` ```csharp ` con el código propuesto cuando la solución requiere contexto adicional.

> **Principio rector:** esta skill NO inventa hallazgos. Solo publica los que ya están en el reporte local aprobado por el usuario. Si un hallazgo no tiene archivo+línea, se publica como parte del comentario general (no inline).

> **Operación visible a terceros:** publicar comentarios en un PR del compañero es una acción NO reversible silenciosamente (queda en historial). Por eso esta skill SIEMPRE pide confirmación explícita antes de publicar.

---

## Cuándo usar este skill

- Cuando el usuario dice: _"publica este review"_, _"comenta el PR"_, _"sube los hallazgos al PR"_, _"deja el feedback en GitHub"_.
- Cuando ya existe un archivo `docs/90_borradores/pr-review-{n}-{scope}-{fecha}.md` revisado.
- NO usar para revisar un PR desde cero — para eso es `pr-hard-review-marchamo`.

---

## Prerrequisitos

1. **Reporte local existente** en `docs/90_borradores/pr-review-*.md` con el formato emitido por `pr-review-marchamo` (resumen ejecutivo + hallazgos por severidad con `Archivo:` y `Acción:`).
2. **Acceso al PR via GitHub MCP** (`mcp_io_github_git_*`). No solicitar tokens al usuario; usar exclusivamente el MCP integrado (ver `AGENTS.md` → "Acceso a repositorios privados de GitHub").
3. **Permisos de review** en el repositorio destino (el usuario debe ser colaborador con permiso de review).

---

## Procedimiento paso a paso

### PASO 1 — Cargar y parsear el reporte local

1. Leer el archivo `.md` indicado por el usuario (o, si no especifica, el más reciente en `docs/90_borradores/pr-review-*.md`).
2. Extraer:
   - **Metadatos:** repo, número de PR, rama, fecha, veredicto, conteo de hallazgos.
   - **Resumen ejecutivo:** texto del bloque "Resumen ejecutivo" y "Puntos positivos".
   - **Hallazgos:** lista estructurada con `severidad`, `categoría` (A/B/C/.../S), `regla_id` (ej: `C4`, `S2`), `título`, `archivo`, `línea` (puede ser rango), `descripción`, `acción`, y `propuesta_de_solución` (código propuesto, si existe).
3. Si el reporte no incluye propuesta de solución para un hallazgo, **derivarla** a partir de la `acción` recomendada y de los patrones de código canónicos definidos en `pr-review-marchamo/SKILL.md` (sección "Patrones de código de referencia"). Marcar la propuesta como "propuesta automática" para que el usuario la valide.

### PASO 2 — Validar el PR destino

1. Confirmar repo + número de PR con `mcp_io_github_git_pull_request_read` (state, head SHA, base SHA, archivos).
2. Verificar que el PR esté **abierto** (no merged, no closed). Si está cerrado, abortar y avisar.
3. Obtener el **commit SHA del head actual** del PR — los comentarios inline se anclan a ese SHA. Si el head SHA del reporte (si lo trae) difiere del head actual, advertir al usuario: el código pudo haber cambiado y algunas líneas pueden ya no existir.
4. Obtener la lista de archivos del PR con `mcp_io_github_git_get_pull_request_changes` o equivalente, para validar que cada `archivo` referenciado en los hallazgos existe en el diff. Hallazgos sobre archivos NO presentes en el diff se publican como parte del comentario general (no inline).

### PASO 3 — Confirmación obligatoria con el usuario

Antes de publicar NADA en GitHub, mostrar al usuario:

```
Voy a publicar la review en el PR #{n} de {repo} (head: {sha_short}):
  • 1 comentario general (resumen ejecutivo + veredicto)
  • {X} comentarios inline (🔴 {a}  🟡 {b}  🟢 {c})

Hallazgos no inline (sin archivo o archivo fuera del diff): {Y} → irán dentro del comentario general.

¿Confirmas la publicación? (sí / no / preview)
```

- `sí` → continuar.
- `preview` → mostrar el cuerpo exacto del comentario general y el listado de inline comments (archivo + línea + suggestion) sin publicar; volver a pedir confirmación.
- `no` → abortar sin tocar el PR.

**No publicar nada si la respuesta no es un "sí" explícito.**

### PASO 4 — Publicar el comentario general

Usar `mcp_io_github_git_add_issue_comment` (un PR es un issue para efectos de comentarios a nivel del PR) con el siguiente cuerpo:

```markdown
## 🔎 Revisión automatizada — {repo} PR #{n}

**Veredicto:** {✅ APROBADO | ⚠️ APROBADO CON CONDICIONES | 🚫 REQUIERE CAMBIOS}
**Hallazgos:** 🔴 {bloqueantes}  🟡 {advertencias}  🟢 {sugerencias}
**Fecha de revisión local:** {YYYY-MM-DD}
**Reporte fuente:** `docs/90_borradores/pr-review-{n}-{scope}-{fecha}.md`

### Puntos positivos
{listado}

### Resumen
{resumen ejecutivo, máximo ~10 líneas}

### Hallazgos sin ancla en el diff
{si los hay; cada uno con regla_id, severidad, descripción breve y acción}

### Condición de aprobación
{texto del reporte}

---
> Generado automáticamente por la skill `pr-review-publisher` a partir de la skill `pr-review-marchamo`.
> Cada hallazgo con archivo/línea aparece como comentario inline en este mismo PR.
> Los bloques `suggestion` son **propuestas**, no decisiones del revisor humano — validar antes de aceptar.
```

Guardar el `id` del comentario general devuelto por la API.

### PASO 5 — Crear una pending review y agregar los comentarios inline

1. Crear una review pendiente con `mcp_io_github_git_pull_request_review_write` en modo `create` (sin body, sin event aún).
2. Para cada hallazgo con `archivo` + `línea`:
   - Usar `mcp_io_github_git_add_comment_to_pending_review` con el `pullNumber`, el `path`, la `line` (o rango `start_line`–`line`), el lado `RIGHT` (sobre el código nuevo del PR), y el cuerpo descrito abajo.
   - El `commitId` debe ser el **head SHA actual** obtenido en el PASO 2.
3. Si una línea ya no existe en el head actual (porque el autor modificó el archivo entre la generación del reporte y la publicación), degradar ese hallazgo a entrada del comentario general (no inline) y registrar el motivo.

**Plantilla del cuerpo de cada inline comment:**

```markdown
{🔴|🟡|🟢} **[{regla_id}] {título}**

{descripción del hallazgo, 1–4 líneas}

**Acción recomendada:** {acción}

**Propuesta de solución:**

```suggestion
{código exacto para reemplazar las líneas seleccionadas}
```

> Regla: `{categoría}.{regla_id}` (ver `MarchamoDigital.Base/ESTANDAR_API_MARCHAMO.md` o `.github/instructions/`).
> Esta propuesta fue {extraída del reporte | generada automáticamente a partir de patrones canónicos} — validar antes de aceptar.
```

**Cuándo usar `suggestion` vs. bloque de código:**

| Situación | Formato |
|-----------|---------|
| Reemplazo de 1 línea o rango contiguo dentro del archivo, sin imports nuevos | ` ```suggestion ` (GitHub permite "Apply suggestion" con un click) |
| Cambio que requiere agregar `using`, mover código a otro archivo, o crear archivos nuevos | ` ```csharp ` (bloque de código informativo, NO `suggestion`) |
| Refactor amplio (extraer método/clase) | ` ```csharp ` + nota explicando que es ilustrativo |

**Importante sobre `suggestion`:** el contenido del bloque reemplaza EXACTAMENTE las líneas comentadas. Si la propuesta abarca un rango, el inline comment debe usar `start_line` + `line` (multi-line comment) y el bloque `suggestion` debe contener TODAS las líneas reemplazadas, incluyendo indentación correcta.

### PASO 6 — Submit de la review como COMMENT

Submit la pending review con `mcp_io_github_git_pull_request_review_write` en modo `submit` y `event: "COMMENT"`:

- **NUNCA usar `event: "APPROVE"`** desde esta skill — la aprobación es decisión del humano revisor.
- **NUNCA usar `event: "REQUEST_CHANGES"`** desde esta skill, aunque haya bloqueantes — la skill solo informa; la decisión de bloquear el merge es del revisor humano.
- El `body` del submit puede ir vacío (el resumen ya está en el issue comment del PASO 4) o incluir una línea recordando: _"Comentarios inline publicados por `pr-review-publisher`. Revisar y conversar con el autor antes de aprobar/rechazar."_

### PASO 7 — Confirmar al usuario el resultado

Devolver al usuario:

- URL del comentario general (`html_url` que retorna la API).
- URL de la review (la URL del primer inline comment sirve para llegar a la conversación).
- Conteo final publicado: `{N_general}` comentarios generales + `{N_inline_ok}` inline OK + `{N_degraded}` degradados a general (con motivo).
- Si hubo errores parciales (algún inline rechazado por la API), listar archivo + línea + mensaje de error y proponer reintento.

---

## Reglas de seguridad y buenas prácticas

1. **Confirmación obligatoria** (PASO 3) antes de cualquier escritura en GitHub. Sin "sí" explícito → no publicar.
2. **Nunca aprobar ni rechazar el PR** automáticamente. Solo `event: "COMMENT"`.
3. **Nunca editar/eliminar comentarios existentes** del PR sin que el usuario lo pida explícitamente. Esta skill es append-only.
4. **No filtrar PII ni secretos** al cuerpo del comentario. Si el reporte fuente menciona valores como connection strings, tokens, IDs personales, redactarlos como `***` antes de publicar.
5. **No usar `--no-verify`** ni omitir validaciones del PR. Esta skill respeta los gates de CI.
6. **Idempotencia:** si el usuario corre la skill dos veces sobre el mismo PR con el mismo reporte, advertir que ya hay comentarios previos publicados por la skill (detectables por el sello `Generado automáticamente por la skill 'pr-review-publisher'`) y preguntar si desea **agregar** una nueva tanda o **abortar**.
7. **Rate limit:** publicar inline comments uno por uno, no en paralelo, para evitar abuso del API.

---

## Mapeo de herramientas MCP (obligatorio)

| Acción | Herramienta MCP |
|--------|-----------------|
| Leer metadata del PR | `mcp_io_github_git_pull_request_read` |
| Listar archivos / diff del PR | `mcp_io_github_git_get_file_contents` (por archivo) — usar el SHA del head |
| Buscar PRs (si el usuario no dio número) | `mcp_io_github_git_list_pull_requests` / `mcp_io_github_git_search_pull_requests` |
| Comentario general (a nivel PR) | `mcp_io_github_git_add_issue_comment` |
| Crear pending review | `mcp_io_github_git_pull_request_review_write` (acción `create`) |
| Agregar comentario inline a pending review | `mcp_io_github_git_add_comment_to_pending_review` |
| Submit pending review como COMMENT | `mcp_io_github_git_pull_request_review_write` (acción `submit`, `event: "COMMENT"`) |
| Responder a comentario existente (opcional) | `mcp_io_github_git_add_reply_to_pull_request_comment` |

> Si alguna herramienta no está cargada, usar `tool_search` con la descripción de la acción antes de invocarla.

---

## Formato de hallazgo en el reporte fuente (contrato con `pr-review-marchamo`)

Para que esta skill pueda parsear sin ambigüedad, cada hallazgo del reporte local debe seguir este formato:

```markdown
#### {🔴|🟡|🟢} {NIVEL}

1. **[{regla_id}] {título corto}**
   Archivo: `path/relativo/al/repo/destino/Archivo.cs` líneas {inicio}-{fin}   ← rango opcional, si es 1 línea usar "línea N"
   Descripción: {1–4 líneas}
   Acción: {qué debe cambiar el autor}

   Propuesta:
   ```csharp
   // código propuesto (multi-línea OK)
   ```
   {Opcional} Tipo de propuesta: suggestion | snippet   ← default: suggestion si es ≤ rango contiguo
```

Si el reporte NO incluye `Propuesta`, esta skill generará una a partir de la `Acción` y los patrones canónicos de `pr-review-marchamo`. Marcar siempre como _"propuesta automática"_ para que el revisor humano la valide.

---

## Ejemplo de inline comment final (lo que se publica en GitHub)

```markdown
🔴 **[C4] Doble serialización JSON**

`EvaluarMotorReglasUserCase.Handle` está serializando `datos` cuando ya es un `JsonObject`. El doble pase
por `JsonSerializer` genera string escapado y rompe el contrato del endpoint.

**Acción recomendada:** agregar pattern matching para short-circuit cuando el tipo ya es `JsonNode`.

**Propuesta de solución:**

```suggestion
        var payload = datos switch
        {
            JsonObject json => json,
            JsonElement element => JsonNode.Parse(element.GetRawText())!,
            _ => JsonSerializer.SerializeToNode(datos)!
        };
        return Result.Success(payload);
```

> Regla: `C.C4` (ver `MarchamoDigital.Base/ESTANDAR_API_MARCHAMO.md`).
> Propuesta extraída del reporte local — validar antes de aceptar.
```

---

## Errores comunes y manejo

| Error | Causa | Acción |
|-------|-------|--------|
| `422 Unprocessable Entity` al crear inline | La línea no existe en el head SHA actual | Degradar a comentario general con nota "archivo modificado tras la revisión local" |
| `404` al buscar PR | Número incorrecto o repo equivocado | Confirmar con el usuario; no asumir |
| Inline comment publicado pero sin "Apply suggestion" | El bloque `suggestion` no cubre exactamente las líneas comentadas | Reintentar con rango correcto, o convertir a bloque ` ```csharp ` informativo |
| Rate limit (`403 secondary rate limit`) | Demasiados inline en paralelo | Serializar las llamadas; espaciar si vuelve a fallar |
| Reporte sin formato esperado | El reporte fue editado a mano y rompe el contrato | Pedir al usuario que regenere con `pr-review-marchamo` o aceptar publicarlo solo como comentario general |

---

## Skill Collaboration

Cuando la solicitud requiera capacidades fuera del dominio de este Skill, coordinar con el Skill correspondiente siguiendo `framework/DECISION-TREE.md`.

| Skill | Rol |
|-------|-----|
| `pr-review-marchamo` | **Genera** el reporte local de revisión. Esta skill es su consumidor downstream. |
| `pr-review-publisher` (esta) | **Publica** el reporte aprobado en el PR de GitHub. |
| `agent-customization` | Reglas generales del repo sobre entregables y seguridad. |

> Flujo recomendado: `pr-review-marchamo` → usuario revisa/ajusta el reporte local → `pr-review-publisher` con confirmación explícita.

---
## Alcance del Pull Request

El Pull Request debe describir únicamente el cambio implementado.

No debe hacer referencia a:

- Blueprint
- Documento de tareas
- Roles utilizados
- Workflow interno
- Prompts
- Proceso de desarrollo asistido por IA

El PR debe ser entendible por cualquier revisor del repositorio sin necesidad de conocer el proceso interno utilizado para construir la solución.
---
# Criterio para emitir READY FOR PR

El estado **READY FOR PR** únicamente puede emitirse cuando se cumplan todas las siguientes condiciones:

- No existen hallazgos aceptados pendientes de implementación.
- No existen tareas pendientes dentro del alcance del PR.
- No existen observaciones marcadas como "antes del merge".
- El Code Review fue aprobado.
- El QA fue aprobado.

Si cualquiera de estas condiciones no se cumple, el Skill debe detener el proceso y explicar claramente qué actividad continúa pendiente.

Nunca asumas que un hallazgo aceptado ya fue implementado. Debe verificarse explícitamente antes de emitir **READY FOR PR**.

---

## Completion Checklist

Antes de considerar terminada cualquier tarea:

- □ Se respetó el Framework (Principles, Workflow, GIT-Policy, Conventions).
- □ Se siguió el Workflow.
- □ Se utilizó el Skill correcto.
- □ Se mantuvieron cambios mínimos.
- □ Se mantuvo consistencia con el proyecto.
- □ Se espera aprobación del desarrollador antes de cualquier operación Git.
