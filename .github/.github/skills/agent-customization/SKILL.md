---
name: agent-customization
description: "Reglas y comportamiento por defecto para agentes que generan y organizan entregables por historia/PBI en el repositorio. La ruta base y las plantillas las define el Project Profile."
user-invocable: true
---

# Agent Customization — Reglas Operativas

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
Definir la estructura estándar de directorio y el protocolo de inicio para organizar artefactos por historia/PBI en el repositorio.

**Problema que resuelve:**
Los agentes necesitan una guía clara sobre cómo crear, nombrar y organizar los entregables de cada historia para mantener trazabilidad y consistencia.

**Responsabilidades que NO tiene este Skill:**
- No implementa código de negocio.
- No interactúa con Azure DevOps ni con plataformas cloud.
- No realiza operaciones Git.
- No define la taxonomía absoluta del repo: eso vive en el Project Profile (`docs.layout`).

---

## Propósito
Definir la forma estándar en que los agentes deben crear, organizar y mantener artefactos (diagramas, entregables, snapshots) por historia/PBI, evitando desorden y asegurando trazabilidad.

La **ruta base** y las **plantillas** se toman del Project Profile activo. Si el profile no define `docs.layout`, usar la estructura relativa de este Skill bajo el directorio de diseños del proyecto.

---

## Estructura estándar de directorio por historia/PBI

Ruta base (por defecto, sobreescribible por Project Profile):

`<docs.designs_root>/PBI-<id>-<short-name>/`

```
PBI-<id>-<short-name>/
├── 00-readme.md              # OBLIGATORIO — Punto de entrada: metadatos, estado, artefactos, CAs, open questions
├── changelog.md              # OBLIGATORIO — Historial de cambios con fecha y autor
├── questions.md              # RECOMENDADO — Preguntas abiertas activas, bloqueantes e impedimentos
├── data_dictionary.md        # CONDICIONAL — Sólo si la historia involucra modelo de datos/entidades
├── [documentos-trabajo.md]   # OPCIONALES — Archivos de trabajo intermedios durante el desarrollo
├── diagrams/                 # OBLIGATORIO si hay diagramas
│   ├── README.md             # Inventario de diagramas con rutas y estado
│   ├── *.drawio              # Fuente editable (draw.io)
│   ├── *-mermaid.md          # Diagramas en texto (Mermaid)
│   └── *.svg                 # Exportación SVG
├── snapshots/                # OBLIGATORIO si hay diagramas
│   ├── README.md             # Registro de snapshots con fecha y propósito
│   └── YYYY-MM-DD-<desc>.png # Snapshots versionados con fecha
├── sources/                  # RECOMENDADO — Materiales de insumo (PDFs procesados, YAMLs fuente, etc.)
│   └── *.md
├── ddl/                      # CONDICIONAL — Solo si la historia genera modelo físico de BD
│   └── *.sql
└── Entregable/               # OBLIGATORIO — Entregable formal revisable/aprobable
    └── PBI-<id>-<titulo>.docx  (o equivalente .md para entregables ligeros)
```

### Clasificación de subdirectorios

| Subdirectorio | Cuándo aplica |
|---|---|
| `diagrams/` | Historia produce diagramas (arquitectura, ER, flujos, estados) |
| `snapshots/` | Siempre que exista `diagrams/` |
| `ddl/` | Historia incluye modelo de datos con persistencia (tablas, DDL, SQL) |
| `sources/` | Se usan documentos fuente externos como insumo (PDFs, YAMLs, specs) |
| `Entregable/` | Toda historia con entregable formal revisable por stakeholders |
| `data_dictionary.md` | Historia define o refina entidades de dominio |
| `questions.md` | Siempre — registrar impedimentos y preguntas desde el inicio |

---

## Protocolo de inicio — Al comenzar a trabajar una nueva historia

Al recibir el contexto de una historia nueva, el agente DEBE:

1. **Leer** la fuente de historia definida por el Project Profile (backlog local, ADO, etc.).
2. **Crear** la carpeta `<docs.designs_root>/PBI-<id>-<short-name>/` si no existe.
3. **Crear** `00-readme.md` (usando la plantilla de borrador del Project Profile, si existe) con metadatos completos:
   - Estado: borrador
   - Autor, Fecha, Resumen del objetivo
   - Referencia al Work Item (URL del Project Profile / herramienta de trabajo)
   - Lista de artefactos esperados con links relativos
   - Extracto de criterios de aceptación
4. **Crear** `changelog.md` con entrada inicial de fecha y autor.
5. **Crear** `questions.md` con las preguntas detectadas al revisar la historia.
6. **Crear** subdirectorios aplicables según la tabla de clasificación anterior, cada uno con su `README.md`.
7. **Actualizar** el índice documental del proyecto (si el Project Profile lo define).
8. **Reportar** al desarrollador el árbol creado y las preguntas detectadas.

---

## Convenciones de nombrado

- Directorio PBI: `PBI-<id>-<short-name>` — short-name en kebab-case, máx. 4 palabras clave.
- Snapshots y entregables intermedios: `YYYY-MM-DD-<descripcion>.ext`
- Archivos de trabajo en raíz: `<tema>_<aspecto>.md`
- Entregable formal en `Entregable/`: `PBI <id> - <Titulo Largo>.docx` (con espacios, para uso humano)
- DDL: `<dominio>_<aspecto>.sql`

---

## Reglas de trazabilidad

- Todo `00-readme.md` debe listar **todos los artefactos** del PBI con links relativos.
- Incluir siempre referencia al Work Item y a historias relacionadas.
- Usar plantillas de borrador/vigente definidas por el Project Profile.
- Toda decisión de diseño relevante se registra en `changelog.md`.

---

## Permisos y límites

- El agente puede **crear y editar** archivos en el árbol documental del proyecto y en Skills del Framework instalado.
- El agente **NO moverá** archivos binarios críticos (`.drawio`, `.docx`, `.pdf`) sin confirmación explícita del autor; en su lugar referenciará y propondrá la acción.
- El agente puede **actualizar** el índice documental y `00-readme.md` en la misma sesión de trabajo.

---

## Referencia

No hay una referencia canónica de un proyecto concreto en el Core.

El Project Profile puede apuntar a un ejemplo interno del propio proyecto consumidor.

---

## Calidad y auditoría

- Declarar `Estado: borrador` o `Estado: vigente` en cada documento.
- Registrar en `changelog.md` toda modificación con formato: `YYYY-MM-DD — Autor — Descripción`.
- Las `questions.md` deben tener estado por pregunta: `[ ] pendiente` / `[x] resuelta`.

---

## Skill Collaboration

Cuando la solicitud requiera capacidades fuera del dominio de este Skill, coordinar con el Skill correspondiente siguiendo `framework/DECISION-TREE.md` y el skill-map del Project Profile.

| Necesidad | Intención |
|---|---|
| Exportar entregable Markdown a Word | según skill-map (`exportar-md-docx` si está instalado) |
| Refinar historia de usuario | `work-management` |
| Generar blueprint técnico de la historia | `generate-blueprint` |

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
