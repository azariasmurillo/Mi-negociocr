---
name: ado-tareas-report
description: "Experto en ejecutar, interpretar y operar el script ado_tareas_report.py del proyecto Marchamo Digital. Permite consultar, analizar, corregir y exportar tareas diarias de ADO por persona y mes. Cubre todos los modos (detalle, resumen, plano), la exportación a Excel, corrección de horas en ADO y diagnóstico del PAT. USA CUANDO: ver horas del mes, generar bitácora, revisar horas extra, exportar Excel de tareas, corregir CompletedWork, reporte de tareas ADO, bitácora novacomp, ver tareas por persona, reporte mensual."
user-invocable: true
---

# Skill: ADO Tareas Report

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
Asistir al usuario en el uso completo del script `ado_tareas_report.py` para consultar, analizar, corregir y exportar tareas diarias de Azure DevOps.

**Problema que resuelve:**
El equipo necesita generar bitácoras, detectar horas extra y corregir `CompletedWork` en ADO sin memorizar todos los parámetros y procedimientos del script.

**Responsabilidades que NO tiene este Skill:**
- No crea work items en ADO (eso es `registrar-tareas-diarias`).
- No realiza operaciones Git.

---

## Propósito

Asistir al usuario en el uso completo del script `Marchamo_Digital/scripts/ado_tareas_report.py`, que consulta las Tasks de Azure DevOps del proyecto **Seguro Obligatorio Automotor** y las presenta en distintos formatos. Permite además detectar y corregir errores de horas, y exportar una bitácora Excel lista para entregar.

**Ruta del script:**
```
/home/ftorres/GitHub_INS/Marchamo_Digital/scripts/ado_tareas_report.py
```

**Prerrequisito:** Variable de entorno `ADO_PAT` con un Personal Access Token válido para `dev.azure.com/grupoins`.

---

## Referencia rápida de parámetros

| Parámetro | Alias | Valores | Default | Descripción |
|-----------|-------|---------|---------|-------------|
| `--email` | `-e` | email ADO o `-` | `ftorres@crnova.com` (o `$ADO_USER_EMAIL`) | Persona a consultar. `-` = todas las personas del mes |
| `--mes` | `-m` | 1–12 | mes actual | Mes a consultar |
| `--anio` | `-a` | YYYY | año actual | Año a consultar |
| `--modo` | `-d` | `detalle` / `resumen` | `detalle` | Nivel de detalle de la salida |
| `--submodo` | `-s` | `agrupado` / `plano` | `agrupado` | Submodo del modo detalle |
| `--campo-fecha` | `-f` | `atencion` / `creacion` | `atencion` | Campo de fecha a usar para filtrar y agrupar |
| `--excel` | — | flag | no | Exportar bitácora Excel además de la salida en consola |
| `--salida` | — | ruta .xlsx | `~/bitacora_USUARIO_YYYY_MM.xlsx` | Ruta personalizada para el Excel |

---

## Campos de fecha

| Clave | Campo ADO | Cuándo usar |
|-------|-----------|-------------|
| `atencion` *(default)* | `Custom.ad762953-443e-4988-b93d-f892183f33c3` | Fecha de atención (campo personalizado). Representa el **día real de trabajo**. Lo setea el skill `registrar-tareas-diarias`. Usar siempre para reportes de horas. |
| `creacion` | `System.CreatedDate` | Fecha de creación del work item en ADO. Útil si la tarea fue creada manualmente sin usar el skill. |

---

## Modos de visualización

### `--modo detalle --submodo agrupado` *(default)*

Tabla completa por día, con cada tarea, horas, estado, padre y título. Ideal para auditar el detalle de lo registrado.

```
📅  2026-06-01 (Lunes)  [8.50h]
      ID       H  Estado        Padre                     Título
  301678   0.25h  Closed        Gestiones MD              Daily Scrum 01/06/2026
  301679   2.00h  Closed        Gestiones MD              Revisión y aprobación de Pull Requests
  301680   6.25h  Closed        4.8 Baja Administrativa…  HU-261779: Implementación...
```

### `--modo detalle --submodo plano`

Una tarjeta por día con los campos institucionales: Fecha, Tareas, Sistema, No Requerimiento, Consumidas, Fase, Recurso, Horas Extras.

```
  2026-06-01 (Lunes)
  ──────────────────────────────────────────
  Fecha:              2026-06-01
  Tareas:             Daily Scrum 01/06/2026
                      Revisión y aprobación de Pull Requests
  Sistema:            Seguro Obligatorio Automotor
  No Requerimiento:   4.8 Baja Administrativa...
  Consumidas:         8.50h
  Fase:               Scrum / Desarrollo
  Recurso:            Francisco Javier Torres Siles
  Horas Extras:       —
```

### `--modo resumen`

Una línea por día con horas normales y horas extra. Incluye marcador `⚡` para días con extra y `[FDS]` para fines de semana. Ideal para detectar inconsistencias de un vistazo.

```
  Día                                 Total   Normal    Extra
  2026-06-22 (Lunes)                 13.50h    8.50h    5.00h ⚡
  2026-06-29 (Lunes)                 10.50h    8.50h    2.00h ⚡
```

---

## Clasificación automática de Fase

El script clasifica cada tarea según palabras clave en el título (en orden de prioridad):

| Fase | Palabras clave detectadas |
|------|--------------------------|
| **Scrum** | daily, scrum, standup, sprint review, sprint planning, planning, retrospectiva, refinamiento |
| **Pruebas** | prueba, test, qa, validación, verificación |
| **Análisis** | análisis, levantamiento, blueprint, estrategia, diseño, modelo, definición, alistamiento, enlistado, workspace, refinado |
| **Desarrollo** | todo lo que no clasifica en las anteriores (catch-all) |

Si un día tiene tareas de múltiples fases, se muestran separadas por ` / ` (ej. `Scrum / Desarrollo / Pruebas`).

> Para ajustar la clasificación, editar `FASES_PALABRAS` en la sección de configuración del script.

---

## Exportación a Excel (`--excel`)

Genera un archivo `.xlsx` estilo Bitácora Novacomp con:
- Fila 1: título con nombre completo del recurso (azul oscuro)
- Fila 2: período, proyecto y timestamp de generación
- Fila 3: encabezados con filtros automáticos (▼)
- Filas de datos: alternadas blanco/gris, días con horas extra en **rojo**
- Fila de totales: suma de Consumidas y Horas Extras
- Columnas con `wrap_text` para Tareas y Requerimiento

Si hay múltiples personas, genera una pestaña por persona.

**Requiere:** `openpyxl` instalado. Si no está disponible:
```bash
python3 -m pip install openpyxl --user
```

---

## Ejemplos de uso frecuente

```bash
# Ver el mes actual (modo detalle, persona por defecto)
python3 Marchamo_Digital/scripts/ado_tareas_report.py

# Resumen de junio 2026 (horas normal/extra por día)
python3 Marchamo_Digital/scripts/ado_tareas_report.py \
  --email ftorres@crnova.com --mes 6 --anio 2026 --modo resumen

# Ver detalle de junio en formato de tarjeta institucional
python3 Marchamo_Digital/scripts/ado_tareas_report.py \
  --email ftorres@crnova.com --mes 6 --anio 2026 \
  --modo detalle --submodo plano

# Generar bitácora Excel de junio
python3 Marchamo_Digital/scripts/ado_tareas_report.py \
  --email ftorres@crnova.com --mes 6 --anio 2026 \
  --modo detalle --submodo plano --excel

# Todas las personas del equipo en julio
python3 Marchamo_Digital/scripts/ado_tareas_report.py \
  --email - --mes 7 --anio 2026 --modo resumen

# Usar fecha de creación en vez de fecha de atención
python3 Marchamo_Digital/scripts/ado_tareas_report.py \
  --mes 6 --anio 2026 --campo-fecha creacion
```

---

## Diagnóstico y corrección de horas

### Detectar días con horas extra inesperadas

```bash
python3 Marchamo_Digital/scripts/ado_tareas_report.py \
  --email EMAIL --mes M --anio AAAA --modo resumen
```
Buscar líneas con `⚡`. Si el extra es pequeño (0.25h), probablemente es un error de entrada.

### Ver el detalle de los días con problemas

```bash
python3 Marchamo_Digital/scripts/ado_tareas_report.py \
  --email EMAIL --mes M --anio AAAA --modo detalle 2>/dev/null \
  | grep -A15 "FECHA_DEL_DIA"
```

### Corregir `CompletedWork` de una tarea en ADO

```python
import os, base64, json, urllib.request

pat = os.environ['ADO_PAT'].strip()
auth = "Basic " + base64.b64encode((":" + pat).encode()).decode()

wi_id = 306904   # ID de la tarea a corregir
nuevas_horas = 4.0

url = f"https://dev.azure.com/grupoins/_apis/wit/workitems/{wi_id}?api-version=7.1"
body = [{"op": "add", "path": "/fields/Microsoft.VSTS.Scheduling.CompletedWork", "value": nuevas_horas}]
req = urllib.request.Request(url, data=json.dumps(body).encode(), headers={
    "Authorization": auth, "Content-Type": "application/json-patch+json", "Accept": "application/json"
}, method="PATCH")
with urllib.request.urlopen(req) as r:
    d = json.loads(r.read())
    print(f"✅ {wi_id} → {d['fields']['Microsoft.VSTS.Scheduling.CompletedWork']}h")
```

---

## Diagnóstico del PAT

### Verificar que el PAT funciona

```bash
curl -s -o /dev/null -w "%{http_code}" -u ":$ADO_PAT" \
  "https://dev.azure.com/grupoins/_apis/projects?api-version=7.1"
# 200 = OK | 401 = PAT inválido/expirado
```

### Síntomas comunes

| Síntoma | Causa probable | Solución |
|---------|---------------|----------|
| HTTP 401 con body vacío | PAT incorrecto o expirado | Regenerar en https://dev.azure.com/grupoins/_usersSettings/tokens |
| `ModuleNotFoundError: openpyxl` | Módulo no instalado | `python3 -m pip install openpyxl --user` |
| Campo `fecha desconocida` | Campo de fecha no está seteado en las tareas | Usar `--campo-fecha creacion` como alternativa |
| `0.00h` en todas las tareas | El endpoint batch falló silenciosamente | Verificar el PAT y la conectividad |

### Actualizar el PAT en `.bashrc`

```bash
# Regenerar el token en ADO y luego:
sed -i 's|export ADO_PAT=.*|export ADO_PAT="NUEVO_TOKEN"|' ~/.bashrc
source ~/.bashrc

# Verificar
curl -s -o /dev/null -w "%{http_code}\n" -u ":$ADO_PAT" \
  "https://dev.azure.com/grupoins/_apis/projects?api-version=7.1"
```

---

## Contexto técnico del script

| Constante | Valor | Descripción |
|-----------|-------|-------------|
| `ORG` | `grupoins` | Organización Azure DevOps |
| `PROJECT` | `Seguro Obligatorio Automotor` | Proyecto ADO |
| `JORNADA_NORMAL` | `8.5` | Horas de jornada ordinaria (umbral horas extra) |
| `FECHA_ATENCION_FIELD` | `Custom.ad762953-443e-4988-b93d-f892183f33c3` | Campo "Fecha de atención" |
| `FECHA_CREACION_FIELD` | `System.CreatedDate` | Fecha de creación del work item |
| Batch endpoint | `POST /_apis/wit/workitemsbatch` | Endpoint usado para traer campos de múltiples work items |

---

## Skill Collaboration

Cuando la solicitud requiera capacidades fuera del dominio de este Skill, coordinar con el Skill correspondiente siguiendo `framework/DECISION-TREE.md`.

| Necesidad | Skill |
|---|---|
| Crear tareas diarias en ADO | `registrar-tareas-diarias` |

---

## Completion Checklist

Antes de considerar terminada cualquier tarea:

- □ Se respetó el Framework (Principles, Workflow, GIT-Policy, Conventions).
- □ Se siguió el Workflow.
- □ Se utilizó el Skill correcto.
- □ Se mantuvieron cambios mínimos.
- □ Se mantuvo consistencia con el proyecto.
- □ Se espera aprobación del desarrollador antes de cualquier operación Git.
