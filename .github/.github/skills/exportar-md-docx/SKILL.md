---
name: exportar-md-docx
description: "Exporta documentos Markdown a Word (.docx) con diagramas Mermaid convertidos a imágenes PNG embebidas. USE WHEN: el usuario quiere convertir un .md a Word, exportar entregable formal, generar .docx desde markdown, compartir documento con stakeholders, exportar con diagramas, o cuando se genera un entregable en Entregable/ que contiene bloques Mermaid o imágenes."
argument-hint: "Ruta del archivo .md a convertir. Opcional: ruta de salida, plantilla Word de referencia, escala Mermaid."
user-invocable: true
---

# Exportar Markdown a DOCX (con Mermaid → PNG)

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
Convertir archivos Markdown del repositorio a Word (.docx) renderizando automáticamente los diagramas Mermaid como imágenes PNG embebidas.

**Problema que resuelve:**
Los stakeholders necesitan entregables en Word con diagramas visibles — este Skill automatiza la conversión sin pérdida de contenido visual.

**Responsabilidades que NO tiene este Skill:**
- No genera el contenido del documento Markdown (eso es `doc-tecnica`, `doc-funcional`, etc.).
- No realiza operaciones Git.

---

## Propósito
Convertir archivos Markdown del repositorio a Word (.docx) de forma automática, renderizando todos los bloques ` ```mermaid ` como imágenes PNG embebidas en el documento final.

---

## Cuándo sugerir este flujo

El agente DEBE sugerir esta opción de forma proactiva cuando:

- Se genera o edita un archivo en `Entregable/` que contiene bloques ` ```mermaid `.
- El usuario pide "entregar", "exportar", "compartir con el equipo", "generar Word" o "preparar para revisión".
- El entregable formal de una historia (según `agent-customization`) está en Markdown y el PO o un stakeholder lo va a revisar.
- Se crea un entregable que cumple el patrón `Entregable/PBI-<id>-*.md`.

**Mensaje sugerido al usuario:**
> "Este documento contiene diagramas Mermaid. ¿Querés convertirlo a Word con las imágenes embebidas? Podés ejecutar:
> `./convert-md-to-docx.ps1 -InputPath \"<ruta-del-archivo>\"`"

---

## Herramientas requeridas

| Herramienta | Instalación | Verificar |
|---|---|---|
| **Pandoc** | https://pandoc.org/installing.html — instalar **para todos los usuarios** | `pandoc --version` |
| **Mermaid CLI** | `npm install -g @mermaid-js/mermaid-cli` | `mmdc --version` |
| **Node.js** | https://nodejs.org/ (prerrequisito de Mermaid CLI) | `node --version` |

> ⚠️ Pandoc **debe instalarse para todos los usuarios** (no solo el usuario actual) para que quede disponible globalmente en PATH.

---

## Scripts del repositorio

Los scripts viven en la raíz del repo:

| Archivo | Propósito |
|---|---|
| `convert-md-to-docx.ps1` | Script principal PowerShell — parámetros completos |
| `convert-md-to-docx.bat` | Wrapper para drag-drop o ejecución desde explorador |

---

## Uso básico

```powershell
# Desde la raíz del repo
./convert-md-to-docx.ps1 -InputPath "<docs.designs_root>/PBI-XXXXX-Nombre/Entregable/PBI-XXXXX-Titulo.md"
```

Genera el `.docx` en la misma carpeta con el mismo nombre base.

---

## Parámetros del script

| Parámetro | Tipo | Requerido | Default | Descripción |
|---|---|---|---|---|
| `-InputPath` | string | ✅ | — | Ruta al `.md` a convertir |
| `-OutputPath` | string | ❌ | Mismo dir y nombre | Ruta de salida `.docx` |
| `-ReferenceDoc` | string | ❌ | — | Plantilla Word para aplicar estilos corporativos |
| `-PandocPath` | string | ❌ | Auto-detectado | Ruta manual a `pandoc.exe` si no está en PATH |
| `-MmcdPath` | string | ❌ | Auto-detectado | Ruta manual a `mmdc` |
| `-MermaidScale` | int | ❌ | `2` | Factor de escala para renderizado PNG |
| `-MermaidWidth` | int | ❌ | `1600` | Ancho en píxeles del PNG generado |
| `-UseTemplate` | string | ❌ | — | Ruta a la plantilla oficial INS. Fusiona portada/estilos INS con el contenido del .md. El entregable hereda portada, footer corporativo y TOC. |
| `-AllowLocalLinks` | switch | ❌ | false | Permitir hipervínculos a archivos locales (default deshabilitado para evitar referencias rotas en Word) |
| `-KeepIntermediate` | switch | ❌ | false | Conserva los `.mmd`, `.png` y `.processed.md` temporales |

---

## Ejemplos comunes

### Conversión simple
```powershell
./convert-md-to-docx.ps1 -InputPath "<docs.designs_root>/PBI-12345-ejemplo/Entregable/PBI-12345-Titulo.md"
```

### Con plantilla corporativa
```powershell
./convert-md-to-docx.ps1 `
  -InputPath "docs/.../Entregable/PBI-XXXXX-Titulo.md" `
  -ReferenceDoc "docs/00_plantillas/plantilla_word_corporativa.docx"
```

### Conservar PNGs generados (para revisión)
```powershell
./convert-md-to-docx.ps1 -InputPath "docs/.../archivo.md" -KeepIntermediate
```

### Pandoc no está en PATH (workaround)
```powershell
./convert-md-to-docx.ps1 `
  -InputPath "docs/.../archivo.md" `
  -PandocPath "C:\Program Files\Pandoc\pandoc.exe"
```

---

## Flujo interno del script

```
entrada.md
  ↓
1. Detecta bloques ```mermaid``` con regex
  ↓
2. Extrae cada bloque a un archivo .mmd temporal
  ↓
3. Renderiza cada .mmd → .png (via mmdc con escala configurada)
  ↓
4. Reemplaza bloques Mermaid por referencias ![](imagen.png)
  ↓
5. Pandoc convierte el .processed.md → .docx (con imágenes embebidas)
  ↓
6. Limpia archivos temporales (salvo -KeepIntermediate)
  ↓
salida.docx ✅
```

---

## Diagnóstico de errores comunes

### `pandoc: command not found`
Pandoc está instalado pero no está en PATH.
- **Causa más frecuente:** se instaló solo para el usuario actual, no para todos.
- **Solución A:** Reinstalar Pandoc marcando "Install for all users".
- **Solución B:** Usar `-PandocPath "C:\Program Files\Pandoc\pandoc.exe"`.
- **Verificar:** `where.exe pandoc` en cmd.

### `mmdc: command not found`
```powershell
npm install -g @mermaid-js/mermaid-cli
# Reiniciar PowerShell
mmdc --version
```

### `mmdc fallo al renderizar`
- Verifica que el bloque Mermaid sea sintácticamente válido en https://mermaid.live
- Usa `-MermaidScale 1` si hay problemas de memoria con diagramas muy grandes.

### Word no muestra las imágenes
- Asegúrate de que el `.docx` no está abierto en Word al generar.
- Usa `-KeepIntermediate` para confirmar que los `.png` se generaron correctamente.

---

## Convenciones del repositorio

- Los `.docx` generados van en la misma carpeta del `.md` fuente (`Entregable/`).
- **No commitear** los `.docx` generados automáticamente —son artefactos derivados.
- Los archivos temporales se guardan en `Entregable/.md2docx/` (ignorado por git).
- Agrega `.md2docx/` al `.gitignore` si aún no está.

---

## Referencia de archivos clave

| Archivo | Ruta |
|---|---|
| Script PowerShell | `convert-md-to-docx.ps1` |
| Script Batch (drag-drop) | `convert-md-to-docx.bat` |
| Guía completa de uso | `CONVERTIR-MD-A-DOCX-README.md` |

---

## Skill Collaboration

Cuando la solicitud requiera capacidades fuera del dominio de este Skill, coordinar con el Skill correspondiente siguiendo `framework/DECISION-TREE.md`.

| Necesidad | Skill |
|---|---|
| Generar el documento Markdown a convertir | `doc-tecnica` / `doc-funcional` / `doc-arquitectura` |

---

## Completion Checklist

Antes de considerar terminada cualquier tarea:

- □ Se respetó el Framework (Principles, Workflow, GIT-Policy, Conventions).
- □ Se siguió el Workflow.
- □ Se utilizó el Skill correcto.
- □ Se mantuvieron cambios mínimos.
- □ Se mantuvo consistencia con el proyecto.
- □ Se espera aprobación del desarrollador antes de cualquier operación Git.
