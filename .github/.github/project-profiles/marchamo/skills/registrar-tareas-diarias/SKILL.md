---
name: registrar-tareas-diarias
description: "Crea las Tasks diarias en ADO para una persona del equipo. Garantiza que el día cierre con exactamente 8.5h en CompletedWork, distribuyendo entre la Daily obligatoria (0.25h) y tarea(s) de trabajo sustantivo (complemento a 8.5h). Busca el sprint activo, el item 'Gestiones MD' y las US activas dinámicamente — nunca hardcodea IDs."
user-invocable: true
---

# Skill: Registrar Tareas Diarias en ADO

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
Automatizar la creación de Tasks diarias en Azure DevOps para los integrantes del equipo, garantizando que cada día laborable cierre con exactamente 8.5h en `CompletedWork`.

**Problema que resuelve:**
Registrar las tareas diarias en ADO correctamente requiere encontrar el sprint activo, el item 'Gestiones MD' y las US activas de forma dinámica, sin hardcodear IDs.

**Responsabilidades que NO tiene este Skill:**
- No genera reportes de horas (eso es `ado-tareas-report`).
- No realiza operaciones Git.

---

## Propósito
Automatizar la creación de Tasks diarias en Azure DevOps para una persona del equipo `Marchamo Digital Team`, asegurando que cada día laborable quede registrado con exactamente **8.5h** en `CompletedWork`.

---

## Parámetros de entrada

| Parámetro | Tipo | Valor por defecto | Descripción |
|---|---|---|---|
| `persona_email` | string | `ftorres@crnova.com` | Email ADO del integrante (ej. `ftorres@crnova.com`) |
| `fecha_trabajo` | string | Fecha de hoy | Fecha del día a registrar (`DD/MM/YYYY` o `YYYY-MM-DD`) |
| `contexto` | string | `""` | Descripción breve del trabajo realizado ese día. Se usa para generar el título y descripción de la tarea |

**Valores fijos del proyecto:**
- Proyecto ADO: `Seguro Obligatorio Automotor`
- Team: `Marchamo Digital Team`
- AreaPath: `Seguro Obligatorio Automotor\Marchamo Digital Team`
- `Custom.Serviciopor`: `Novacomp`
- `Custom.TipodeHora`: `Hora ordinaria`
- `Microsoft.VSTS.Common.Priority`: `2`

---

## Flujo de ejecución (paso a paso)

### PASO 1 — Resolver parámetros

1. Si no se provee `persona_email`, usar `ftorres@crnova.com`.
2. Si no se provee `fecha_trabajo`, usar la fecha de hoy.
3. Normalizar la fecha a dos formatos:
   - `fecha_display`: `DD/MM/YYYY` → ej. `04/05/2026` (para títulos y descripción)
   - `fecha_corta`: `DD/MM` → ej. `04/05` (para título de la Daily)
   - `fecha_ado`: `YYYY-MM-DDT06:00:00Z` → ej. `2026-05-04T06:00:00Z` (para el campo custom de fecha)

---

### PASO 2 — Encontrar el sprint activo

Usar `mcp_microsoft_azu_work_list_team_iterations` con:
- `project`: `Seguro Obligatorio Automotor`
- `team`: `Marchamo Digital Team`
- `timeframe`: `current`

Extraer el `id` y el `path` del sprint devuelto. Guardar como `sprint_id` y `sprint_path`.

Si no retorna sprint con `current`, listar todas las iteraciones (`mcp_microsoft_azu_work_list_iterations`) y seleccionar la que contenga la `fecha_trabajo` entre `startDate` y `finishDate`.

---

### PASO 3 — Encontrar el item "Gestiones MD" del sprint activo

Ejecutar WIQL via `mcp_microsoft_azu_wit_query_by_wiql`:

```wiql
SELECT [System.Id], [System.Title], [System.WorkItemType], [System.State]
FROM WorkItems
WHERE [System.TeamProject] = 'Seguro Obligatorio Automotor'
  AND [System.WorkItemType] = 'Administrativo'
  AND [System.Title] CONTAINS 'Gestiones MD'
  AND [System.IterationPath] = '<sprint_path>'
  AND [System.State] <> 'Closed'
ORDER BY [System.Id] DESC
```

- Tomar el primer resultado como `id_gestiones_md`.
- Si no existe ningún resultado, informar al usuario: _"No se encontró el item 'Gestiones MD' en el sprint actual. Verificar si fue creado."_ y detener.

---

### PASO 4 — Encontrar US activas del sprint asignadas a la persona

Ejecutar WIQL via `mcp_microsoft_azu_wit_query_by_wiql`:

```wiql
SELECT [System.Id], [System.Title], [System.WorkItemType], [System.State]
FROM WorkItems
WHERE [System.TeamProject] = 'Seguro Obligatorio Automotor'
  AND [System.WorkItemType] IN ('User Story', 'Product Backlog Item')
  AND [System.AssignedTo] = '<persona_email>'
  AND [System.IterationPath] = '<sprint_path>'
  AND [System.State] NOT IN ('Closed', 'Removed')
ORDER BY [System.ChangedDate] DESC
```

- Guardar lista como `us_activas`.
- Si hay 0 resultados: informar al usuario y detener.
- Si hay exactamente 1: usar esa US como `us_trabajo_id`.
- Si hay más de 1: mostrar la lista al usuario y preguntar a cuál vincular la tarea de trabajo. Si el usuario no responde o indica "la más reciente", usar la primera de la lista.

---

### PASO 5 — Verificar tareas ya existentes para ese día

Ejecutar WIQL via `mcp_microsoft_azu_wit_query_by_wiql`:

```wiql
SELECT [System.Id], [System.Title], [Microsoft.VSTS.Scheduling.CompletedWork]
FROM WorkItems
WHERE [System.TeamProject] = 'Seguro Obligatorio Automotor'
  AND [System.WorkItemType] = 'Task'
  AND [System.AssignedTo] = '<persona_email>'
  AND [Custom.ad762953-443e-4988-b93d-f892183f33c3] = '<fecha_ado>'
```

Para cada ID retornado, obtener sus campos `System.Title` y `Microsoft.VSTS.Scheduling.CompletedWork` via `mcp_microsoft_azu_wit_get_work_item`.

Calcular:
- `horas_ya_registradas` = suma de `CompletedWork` de todas las tareas encontradas
- `daily_existe` = `true` si algún título contiene `"Daily"` (case-insensitive)
- `horas_faltantes` = `8.5 - horas_ya_registradas`

Si `horas_faltantes <= 0`: informar _"El día <fecha_display> ya tiene <horas_ya_registradas>h registradas (≥ 8.5h). No se crean tareas adicionales."_ y terminar.

---

### PASO 6 — Determinar qué tareas crear

Con base en el análisis del PASO 5:

| Condición | Acción |
|---|---|
| `daily_existe = false` Y `horas_faltantes >= 8.5` | Crear Daily (0.25h) + Tarea de trabajo (8.25h) |
| `daily_existe = false` Y `horas_faltantes < 8.5` | Crear Daily (0.25h) + Tarea de trabajo (`horas_faltantes - 0.25`h) |
| `daily_existe = true` Y `horas_faltantes > 0` | Crear solo Tarea de trabajo con `horas_faltantes`h |
| `horas_faltantes <= 0` | No crear nada (ya cubierto en PASO 5) |

Si `horas_trabajo <= 0` después del cálculo, solo crear la Daily sin tarea de trabajo.

---

### PASO 7 — Generar contenido de las tareas

#### 7a — Tarea Daily
- **Título**: `Daily <fecha_corta>` → ej. `Daily 04/05`
- **Descripción**: `<div>Participación en la Daily del equipo Marchamo Digital.</div>`
- **CompletedWork**: `0.25`
- **Parent**: `id_gestiones_md` (el "Gestiones MD" del sprint activo)

#### 7b — Tarea de trabajo
- **Título**: Generado a partir del `contexto`. Reglas:
  - Si hay contexto: usar directamente (limpio, conciso, max 120 chars)
  - Si no hay contexto: inferir un título genérico con base en el título de la US activa. Ejemplo: si la US se llama "Implementación de módulo de consultas" → `"Desarrollo e implementación - <nombre corto de la US>"`
- **Descripción** (HTML): Construir con el contexto dado o con base en la US activa. Formato:
  ```html
  <div>
  <b>Actividades del <fecha_display>:</b><br>
  - <punto 1 derivado del contexto o US><br>
  - <punto 2><br>
  </div>
  ```
  Si el contexto es solo una frase, convertirlo en al menos 2-3 puntos de actividad.
- **CompletedWork**: `horas_trabajo` (calculado en PASO 6)
- **Parent**: `us_trabajo_id` (la US activa seleccionada)

---

### PASO 8 — Crear las tareas en ADO

Usar `mcp_microsoft_azu_wit_create_work_item` con los parámetros:
- `workItemType`: `"Task"` (parámetro de nivel superior, NO dentro de `fields`)
- `project`: `"Seguro Obligatorio Automotor"` (parámetro de nivel superior)
- `fields`: **array de objetos `{name, value}`** — ver schema exacto abajo

> ⚠️ **Schema OBLIGATORIO de `fields`** (confirmado en producción — cualquier otra forma falla):
> - `fields` es un **array**, no un objeto/dict
> - Cada elemento tiene las propiedades `name` (string) y `value` (string)
> - `value` SIEMPRE es string — números y booleanos también van como string: `"0.25"`, `"2"`
> - Para campos HTML/Markdown, agregar `"format": "Html"` o `"format": "Markdown"`

```json
{
  "workItemType": "Task",
  "project": "Seguro Obligatorio Automotor",
  "fields": [
    { "name": "System.Title",                                          "value": "<título>" },
    { "name": "System.AssignedTo",                                     "value": "<persona_email>" },
    { "name": "System.AreaPath",                                       "value": "Seguro Obligatorio Automotor\\Marchamo Digital Team" },
    { "name": "System.IterationPath",                                  "value": "<sprint_path>" },
    { "name": "Microsoft.VSTS.Scheduling.CompletedWork",               "value": "<horas>" },
    { "name": "Microsoft.VSTS.Common.Priority",                        "value": "2" },
    { "name": "Custom.ad762953-443e-4988-b93d-f892183f33c3",           "value": "<fecha_ado>" },
    { "name": "Custom.Serviciopor",                                    "value": "Novacomp" },
    { "name": "Custom.TipodeHora",                                     "value": "Hora ordinaria" },
    { "name": "System.Description", "format": "Html",                  "value": "<descripción HTML>" }
  ]
}
```

> ⚠️ **NO intentar crear directamente en estado `Closed`**. Crear en estado `New` (no pasar `System.State`). El cierre se hace en el PASO 8b.

Crear las dos tareas **en paralelo** (una sola llamada para cada una al mismo tiempo).

---

### PASO 8b — Cerrar las tareas (State → Closed)

Usar `mcp_microsoft_azu_wit_update_work_items_batch` con un **array plano** de operaciones. Cada elemento lleva: `id` (número), `path` (string con `/fields/...`), `value` (string), `op` (opcional, default `"Add"`).

> ⚠️ **Schema OBLIGATORIO** (confirmado en producción):
> - NO pasar `project` — no es un parámetro de esta herramienta
> - El array `updates` es **plano**: una entrada por campo por work item
> - NO usar subarrays ni objetos `{fields: [...]}`

```json
{
  "updates": [
    { "id": <id_daily>,   "path": "/fields/System.State",  "value": "Closed" },
    { "id": <id_daily>,   "path": "/fields/System.Reason", "value": "Completed" },
    { "id": <id_trabajo>, "path": "/fields/System.State",  "value": "Closed" },
    { "id": <id_trabajo>, "path": "/fields/System.Reason", "value": "Completed" }
  ]
}
```

Verificar que la respuesta tenga `"count": 2` y ambos items con `"System.State": "Closed"` y `"System.Reason": "Completed"`.

---

### PASO 9 — Vincular las tareas a sus padres

> **IMPORTANTE**: No usar `System.Parent` como campo en creación ni actualización — ADO no lo aplica como relación de jerarquía mediante esos métodos.

Usar **`mcp_microsoft_azu_wit_work_items_link`** con tipo `"parent"` en un solo llamado batch:

> ⚠️ **Schema OBLIGATORIO** (confirmado en producción):
> - `project` es parámetro de nivel superior (string, nombre del proyecto)
> - `updates` es array de objetos `{id, linkToId, type}`
> - `type` debe ser `"parent"` (string exacto, minúsculas)

```json
{
  "project": "Seguro Obligatorio Automotor",
  "updates": [
    { "id": <id_daily>,   "linkToId": <id_gestiones_md>, "type": "parent" },
    { "id": <id_trabajo>, "linkToId": <id_us_activa>,    "type": "parent" }
  ]
}
```

Verificar que en la respuesta el campo `System.Parent` aparezca en el body junto con la `relations` array con `"rel": "System.LinkTypes.Hierarchy-Reverse"`.

> **Paralelización**: El PASO 8b (cerrar) y el PASO 9 (vincular padres) son **independientes** y se pueden ejecutar en paralelo una vez creadas las tareas.

---

### PASO 10 — Reporte final

Al terminar, mostrar un resumen en tabla:

```
✅ Tareas registradas para <persona_email> — <fecha_display>

| ID     | Título                         | Horas | Parent ID | Parent Título       |
|--------|--------------------------------|-------|-----------|---------------------|
| 291XXX | Daily 04/05                    | 0.25  | XXXX      | Gestiones MD        |
| 291XXX | <título tarea trabajo>         | 8.25  | XXXX      | <título US activa>  |
|        | **TOTAL**                      | 8.50  |           |                     |
```

---

## Casos especiales

### Día con reunión larga u otras tareas preexistentes
Si el usuario ya creó manualmente una tarea (ej. reunión de 2h), el PASO 5 detectará `horas_ya_registradas = 2.0`. La skill creará:
- Daily: 0.25h (si no existe)
- Trabajo: 8.5 - 2.0 - 0.25 = 6.25h

### Múltiples US activas
Si la persona tiene varias US activas en el sprint, la skill pregunta a cuál vincular la tarea de trabajo. Si no se especifica, usa la US con `ChangedDate` más reciente.

### Fecha pasada
La skill permite crear tareas para días pasados. La fecha en `Custom.ad762953-443e-4988-b93d-f892183f33c3` se establece al día indicado (no al día de hoy). Esto es el comportamiento deseado para poner al día días faltantes.

### Fin de semana
Si `fecha_trabajo` cae en sábado o domingo, advertir: _"La fecha <fecha_display> es fin de semana. ¿Confirmas que deseas registrar tareas para ese día?"_. Proceder solo si el usuario confirma.

---

## Regla de Horas Extraordinarias

El día laboral estándar es **8.5 horas ordinarias**. Cuando el total de `CompletedWork` del día supera ese umbral, las horas excedentes deben registrarse con `Custom.TipodeHora = "Hora extraordinaria"`.

### Lógica de clasificación

| Condición | `TipodeHora` a usar |
|---|---|
| Tarea entra dentro del bloque de 8.5h del día | `Hora ordinaria` |
| Tarea supera las 8.5h acumuladas del día | `Hora extraordinaria` |

### Ejemplo (día de 12.5h)

```
Tarea A: 0.25h  → acumulado 0.25h  → Hora ordinaria
Tarea B: 8.25h  → acumulado 8.5h   → Hora ordinaria
Tarea C: 4.0h   → acumulado 12.5h  → Hora extraordinaria  (← supera 8.5h)
```

### En el fallback Python

```python
horas_acumuladas = 0.0
for tarea in lista_tareas_del_dia:
    tipo_hora = "Hora ordinaria" if horas_acumuladas < 8.5 else "Hora extraordinaria"
    wid = create_task(..., tipo_hora=tipo_hora)
    horas_acumuladas += tarea["hours"]
```

> El usuario también puede cambiar el `TipodeHora` manualmente desde ADO después de la creación. Al listar las tareas del día, siempre leer el valor actual del campo para no sobreescribir cambios manuales.

---

## Referencia de campos custom ADO

| Campo técnico | Nombre visible | Valores posibles |
|---|---|---|
| `Custom.ad762953-443e-4988-b93d-f892183f33c3` | Fecha de atención | `YYYY-MM-DDT06:00:00Z` |
| `Custom.Serviciopor` | Servicio por | `Novacomp` |
| `Custom.TipodeHora` | Tipo de hora | `Hora ordinaria` / `Hora extraordinaria` |

---

## Herramientas ADO a usar (en orden)

1. `mcp_microsoft_azu_work_list_team_iterations` → sprint activo
2. `mcp_microsoft_azu_wit_query_by_wiql` → Gestiones MD, US activas, tareas existentes del día
3. `mcp_microsoft_azu_wit_get_work_item` → leer CompletedWork de tareas encontradas
4. `mcp_microsoft_azu_wit_create_work_item` → crear Task (en estado `New`)
5. `mcp_microsoft_azu_wit_update_work_items_batch` → cerrar ambas tareas (State=Closed, Reason=Completed)
6. `mcp_microsoft_azu_wit_work_items_link` → vincular padre de cada tarea

> **Prefijo exclusivo**: Siempre usar `mcp_microsoft_azu_*`. El prefijo `mcp_azure_devops_*` produce errores de identidad AAD y no debe usarse.

---

## Esquemas confirmados en producción

### `mcp_microsoft_azu_wit_create_work_item`
- `workItemType`: string (parámetro raíz)
- `project`: string (parámetro raíz)
- `fields`: **array** de `{ name: string, value: string, format?: "Html"|"Markdown" }`
- ⛔ `value` es SIEMPRE string — los números también: `"0.25"`, `"2"`

---

## Skill Collaboration

Cuando la solicitud requiera capacidades fuera del dominio de este Skill, coordinar con el Skill correspondiente siguiendo `framework/DECISION-TREE.md`.

| Necesidad | Skill |
|---|---|
| Consultar o reportar las horas registradas | `ado-tareas-report` |

---

## Completion Checklist

Antes de considerar terminada cualquier tarea:

- □ Se respetó el Framework (Principles, Workflow, GIT-Policy, Conventions).
- □ Se siguió el Workflow.
- □ Se utilizó el Skill correcto.
- □ Se mantuvieron cambios mínimos.
- □ Se mantuvo consistencia con el proyecto.
- □ Se espera aprobación del desarrollador antes de cualquier operación Git.
- ⛔ NO pasar `System.State` — crear siempre en `New`

### `mcp_microsoft_azu_wit_update_work_items_batch`
- `updates`: **array plano** de `{ id: number, path: string, value: string, op?: "Add"|"Replace"|"Remove" }`
- Una entrada por campo por work item (ej. 4 entradas para 2 items × 2 campos)
- ⛔ NO pasar `project` — no existe como parámetro
- ⛔ NO usar subarrays ni objetos `{fields: [...]}`

### `mcp_microsoft_azu_wit_work_items_link`
- `project`: string (parámetro raíz, nombre del proyecto)
- `updates`: array de `{ id: number, linkToId: number, type: string }`
- Para jerarquía padre-hijo usar `type: "parent"` en el item hijo
- ⛔ `System.Parent` como campo en update/create NO crea relación jerárquica real

### `mcp_microsoft_azu_wit_query_by_wiql` — filtrar por fecha custom
Cuando se filtra por el campo de fecha `Custom.ad762953-443e-4988-b93d-f892183f33c3`, usar `timePrecision: true`:
```json
{ "query": "SELECT ... WHERE [Custom.ad762953-443e-4988-b93d-f892183f33c3] = '2026-05-05T06:00:00Z'", "timePrecision": true }
```

---

## Fallback: Script Python con REST API directa (cuando MCP falla)

### Cuándo usar este fallback
Usar este método cuando los tools MCP no estén disponibles o fallen con errores de identidad (`AadUserStateException`) o con el error `A potentially dangerous Request.Path value was detected from the client (:)`. El fallback usa Python con `urllib.request` y autenticación Basic (PAT) directamente contra la API REST de Azure DevOps. **Es la vía más ágil y transparente cuando MCP falla.**

### Prerrequisitos

**Linux:**
```bash
export ADO_PAT="<token>"
# Verificar:
echo $ADO_PAT
```

**Windows (PowerShell):**
```powershell
$env:ADO_PAT = "<token>"
```

- El PAT debe tener scopes: **Work Items (Read & Write)** y **Project and Team (Read)**
- Python: `python3` en Linux / `C:\Program Files\Python312\python.exe` en Windows
- No se requieren librerías externas — solo `urllib.request` (stdlib)

### Flujo de estados para Tasks (CRÍTICO)

> ⚠️ Las Tasks en este proyecto siguen el flujo: **`New → Closed`** (directo)
> - Crear siempre **sin** `System.State` (queda en `New` por defecto)
> - Luego actualizar a `"Closed"` con un segundo PATCH que incluya `Custom.ad762953-443e-4988-b93d-f892183f33c3` (Fecha de atención)
>
> Intentar crear directamente en `"Closed"` o `"Active"` produce HTTP 400.
> El campo `System.Description` es **obligatorio** en la creación — sin él también produce HTTP 400.

### Patrón de creación de Task (PATCH JSON-patch+json)

```python
import os, json, base64, urllib.request, urllib.parse

org = 'grupoins'
project = 'Seguro Obligatorio Automotor'
pat = os.environ['ADO_PAT']
pq = urllib.parse.quote(project, safe='')

def call(method, url, body=None, patch=False):
    headers = {"Authorization": "Basic " + base64.b64encode((':' + pat).encode()).decode()}
    data = None
    if body is not None:
        headers['Content-Type'] = 'application/json-patch+json' if patch else 'application/json'
        data = json.dumps(body).encode('utf-8')
    req = urllib.request.Request(url, data=data, method=method, headers=headers)
    with urllib.request.urlopen(req, timeout=90) as r:
        return json.loads(r.read().decode('utf-8'))

def create_task(title, desc, hours, email, sprint_path, fecha_custom, tipo_hora='Hora ordinaria'):
    ops = [
        {'op': 'add', 'path': '/fields/System.Title',                                        'value': title},
        {'op': 'add', 'path': '/fields/System.AssignedTo',                                   'value': email},
        {'op': 'add', 'path': '/fields/System.AreaPath',                                     'value': 'Seguro Obligatorio Automotor\\Marchamo Digital Team'},
        {'op': 'add', 'path': '/fields/System.IterationPath',                                'value': sprint_path},
        {'op': 'add', 'path': '/fields/System.State',                                        'value': 'New'},
        {'op': 'add', 'path': '/fields/System.Description',                                  'value': desc},
        {'op': 'add', 'path': '/fields/Microsoft.VSTS.Scheduling.CompletedWork',             'value': hours},
        {'op': 'add', 'path': '/fields/Microsoft.VSTS.Common.Priority',                      'value': 2},
        {'op': 'add', 'path': '/fields/Custom.ad762953-443e-4988-b93d-f892183f33c3',         'value': fecha_custom},
        {'op': 'add', 'path': '/fields/Custom.Serviciopor',                                  'value': 'Novacomp'},
        {'op': 'add', 'path': '/fields/Custom.TipodeHora',                                   'value': tipo_hora},
    ]
    item = call('PATCH', f'https://dev.azure.com/{org}/{pq}/_apis/wit/workitems/%24Task?api-version=7.1', ops, patch=True)
    wid = int(item['id'])
    # Flujo: New → Closed (directo). Incluir fecha_custom es obligatorio para que Closed sea válido.
    call('PATCH', f'https://dev.azure.com/{org}/{pq}/_apis/wit/workitems/{wid}?api-version=7.1',
         [{'op': 'add', 'path': '/fields/System.State', 'value': 'Closed'},
          {'op': 'add', 'path': '/fields/Custom.ad762953-443e-4988-b93d-f892183f33c3', 'value': fecha_custom}], patch=True)
    return wid

def link_parent(child_id, parent_id):
    rel = [{'op': 'add', 'path': '/relations/-', 'value': {
        'rel': 'System.LinkTypes.Hierarchy-Reverse',
        'url': f'https://dev.azure.com/{org}/_apis/wit/workItems/{parent_id}',
        'attributes': {'comment': 'Parent link'}
    }}]
    call('PATCH', f'https://dev.azure.com/{org}/{pq}/_apis/wit/workitems/{child_id}?api-version=7.1', rel, patch=True)
```

### Consultar el sprint activo (REST)

```python
team = urllib.parse.quote('Marchamo Digital Team', safe='')
it = call('GET', f'https://dev.azure.com/{org}/{pq}/{team}/_apis/work/teamsettings/iterations?$timeframe=current&api-version=7.1-preview.1')
sprint_path = it['value'][0]['path']
```

### Consultar con WIQL (REST)

```python
wiql_url = f'https://dev.azure.com/{org}/{pq}/_apis/wit/wiql?api-version=7.1'
result = call('POST', wiql_url, {'query': "SELECT [System.Id] FROM WorkItems WHERE ..."})
```

### Verificar estados válidos de un tipo de work item

```python
r = call('GET', f'https://dev.azure.com/{org}/{pq}/_apis/wit/workitemtypes/Task?api-version=7.1')
print(list(r.get('transitions', {}).keys()))
# Para este proyecto: ['Active', 'Closed', 'New', '']
```

### Guardar el script en un archivo temporal y ejecutarlo

```powershell
# Guardar script en archivo
$scriptPath = "$env:TEMP\registrar_horas_YYYY_MM_DD.py"

# Ejecutar
& 'C:\Program Files\Python312\python.exe' $scriptPath 2>&1
```

### Errores conocidos y soluciones

| Error | Causa | Solución |
|---|---|---|
| `HTTP 401 Unauthorized` | PAT inválido, expirado o sin permisos de escritura | Actualizar `$env:ADO_PAT` con un PAT que tenga scope `Work Items: Read & Write` |
| `HTTP 400: State 'Active' not in supported values` | Se intentó crear en `Active` directamente | Crear en `New` primero, luego transicionar a `Active`, luego a `Closed` |
| `AadUserStateException` en MCP tools | Identidad del agente eliminada del tenant ADO | Usar el fallback Python con PAT directo |
| `HTTP 400: AreaPath inválido` | El AreaPath no existe o tiene formato incorrecto | Usar `Seguro Obligatorio Automotor\Marchamo Digital Team` (sin comillas extra) |
| `HTTP 400: Rule Error for field Description` | `System.Description` no fue enviado en la creación | Incluir siempre `System.Description` en el PATCH de creación, aunque sea el mismo título |
| `HTTP 400: Rule Error for field Fecha de atención` | El campo `Custom.ad762953-443e-4988-b93d-f892183f33c3` no fue enviado al cerrar | Incluir la fecha custom en el PATCH de cierre (State=Closed) |
| `A potentially dangerous Request.Path value (:)` | Error del servidor MCP ADO — no es recuperable reintentando | Usar el fallback Python con PAT directo |
