---
name: modelado-datos-bd
description: "Skill especializada en modelado de datos, diseño relacional y generacion de DDL para el proyecto Marchamo Digital. Use when: diseño o revision de modelo ER, abstraccion al modelo relacional, generacion de scripts DDL con convencion de nombres del proyecto, definicion de catalogos y datos semilla, modelado de estados y transiciones, diccionario de datos, snapshots de diagramas, y cualquier actividad relacionada con entidades, atributos, relaciones y bases de datos del dominio."
argument-hint: "Proporciona el modelo ER fuente, las entidades involucradas, la historia o PBI de referencia, y cualquier restriccion de negocio o convencion de base de datos conocida"
user-invocable: true
---

# Modelado de Datos y Bases de Datos — Marchamo Digital

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
Asistir en el diseño, documentación y generación de artefactos de datos del proyecto Marchamo Digital: modelos ER, modelos relacionales, DDL y diccionarios de datos.

**Problema que resuelve:**
El proyecto tiene un estándar de nomenclatura específico (prefijos `con`, `cod`, `des`, etc.) que debe aplicarse consistentemente en todas las entidades de la base de datos.

**Responsabilidades que NO tiene este Skill:**
- No ejecuta scripts DDL en bases de datos productivas.
- No decide la arquitectura de datos (eso es `arquitecto-de-software`).
- No realiza operaciones Git.

---

## Purpose
Asistir en el diseño, documentación y generación de artefactos de datos del proyecto Marchamo Digital: modelos ER, modelos relacionales, DDL y diccionarios de datos, aplicando el estándar de nomenclatura oficial del proyecto.

## Plataforma objetivo
- **SQL Server / Azure SQL Managed Instance** (T-SQL): para datos transaccionales (marchamos, estados, eventos, inventarios).
- **Cosmos DB**: para datos de alta escala, lectura optimizada o documentos semi-estructurados (evaluar por dominio).
- **Azure Storage**: para binarios; en BD solo se almacena la `rut` (URL/ruta) del recurso.

---

## Estándar de nomenclatura de campos y tablas

### Prefijos obligatorios

| Prefijo | Tipo de dato | Uso | Ejemplo |
|---|---|---|---|
| `con` | BIGINT IDENTITY | **PK**: `con` + nombre de la propia tabla. **FK**: `con` + nombre de la entidad referenciada | `conMarchamo`, `conEmisionTCD` |
| `cod` | VARCHAR | Código alfanumérico, clave legible, identificador externo | `codEstado`, `codPlaca`, `codEPC` |
| `des` | VARCHAR | Descripción explicativa de un valor catalogado | `desEstado`, `desTipoEvento` |
| `nom` | VARCHAR | Nombre propio de un actor, entidad o elemento | `nomRecaudador`, `nomSistema` |
| `ind` | BIT | Indicador booleano — **siempre en sentido positivo** | `indActivo`, `indNotificaPorWhatsapp` |
| `bin` | VARBINARY | Binario — uso excepcional; preferir `rut` + storage externo | `binFotografiaAdjunta` |
| `rut` | VARCHAR | Ruta o URL a recurso en storage o sistema externo | `rutEvidenciaColocacionPlaca` |
| `mon` | DECIMAL(18,2) | Monto monetario | `monTotalAPagar` |
| `por` | DECIMAL(5,4) | Porcentaje | `porImpuestoDeVentas` |
| `fec` | DATETIME2 | Fecha y/o hora *(convención inferida del estándar)* | `fecEmision`, `fecVencimiento` |
| `num` | INT / BIGINT | Número no monetario y no consecutivo *(convención inferida)* | `numAnio`, `numCantidad` |
| `txt` | VARCHAR(MAX) | Texto libre largo, JSON serializado, observaciones *(convención inferida)* | `txtObservaciones`, `txtMetadata` |

### Reglas generales

1. **Tablas**: PascalCase en español, sin abreviaturas innecesarias. Ej: `TarjetaCirculacionDigital`, `EtiquetaRFID`.
2. **PKs**: `con` + nombre de la tabla. Ej: `conMarchamo` en tabla `Marchamo`.
3. **FKs**: `con` + nombre de la entidad referenciada. Ej: `conMarchamo` en tabla `HistorialEstado`.
4. **Indicadores booleanos** (`ind`): siempre redactados en sentido positivo. `indActivo` sí; `indInactivo` no.
5. **Binarios** (`bin`): excepcional. Preferir storage externo y `rut` con la ruta.
6. Los prefijos `fec`, `num` y `txt` son extensión inferida del estándar; pueden formalizarse en futuras versiones.
7. Los campos de auditoría mínimos son: `fecCreacion` (DEFAULT GETDATE()) y `fecActualizacion`.
8. Las **tablas de auditoría** (Evento, HistorialEstado) son **inmutables**: no llevan `fecActualizacion`.

---

## Catálogo de entidades del dominio (referencia)

| Entidad | Tabla relacional | Notas |
|---|---|---|
| MARCHAMO | `Marchamo` | Derecho de circulación; entidad central del dominio |
| VEHICULO | `Vehiculo` | FK a `Dueno` |
| DUENO | `Dueno` | Propietario del vehículo |
| TCD | `TarjetaCirculacionDigital` | PK: `conEmisionTCD`; soporte de reemisiones via auto-referencia |
| RFID_TAG | `EtiquetaRFID` | `codEPC` es el identificador físico único |
| INVENTARIO_ETIQUETAS | `InventarioEtiquetas` | Lote asignado a CEDINS → Recaudador |
| DISPOSITIVO | `Dispositivo` | Asignado a Recaudador y/u Oficial |
| INVENTARIO_DISPOSITIVOS | `InventarioDispositivos` | Lote de dispositivos |
| RECAUDADOR | `Recaudador` | FK a CEDINS |
| OFICIAL | `Oficial` | FK a Usuario |
| USUARIO | `Usuario` | Tabla compartida de identidad |
| CIUDADANO_EMPRESA | `CiudadanoEmpresa` | Actor que paga/recibe el marchamo |
| PERSONAL_SICSOA_BODEGA | `PersonalSICSOABodega` | FK a Usuario |
| CEDINS | `CEDINS` | Unidad distribuidora de etiquetas |
| SISTEMA_EXTERNO | `SistemaExterno` | Referencial; puede moverse a configuración |
| EVENTO | `Evento` | Inmutable; registro de auditoría y trazabilidad |
| TIPO_EVENTO | `TipoEvento` | Catálogo con `indGeneraTransicion` |
| CATALOGO_ESTADOS | `CatalogoEstados` | Estados del derecho de circulación |
| TRANSICION_ESTADO | `TransicionEstado` | Matriz de transiciones válidas |
| HISTORIAL_ESTADO | `HistorialEstado` | Inmutable; log de cambios de estado |

---

## When To Use

- Diseñar o revisar modelos ER del dominio Marchamo Digital.
- Abstraer el modelo ER al modelo relacional aplicando el estándar de nombres del proyecto.
- Generar scripts DDL (T-SQL) para SQL MI con la convención de nomenclatura.
- Definir o ampliar el diccionario de datos (`data_dictionary.md`).
- Modelar catálogos de estados, transiciones e historiales (US 261487, US 277031).
- Generar snapshots de diagramas Mermaid o draw.io para el repositorio.
- Detectar brechas entre el modelo ER y los criterios de aceptación de una historia.
- Modelar entidades nuevas o extender entidades existentes con el estándar vigente.

---

## Inputs

- Modelo ER fuente: `docs/40_disenos_y_propuestas/PBI-276293-Modelado-ER/diagrams/Modelo_ER_mermaid.md`
- Entregable de referencia: `docs/40_disenos_y_propuestas/PBI-276293-Modelado-ER/entregable_modelado_er.md`
- Diccionario de datos: `docs/40_disenos_y_propuestas/PBI-276293-Modelado-ER/data_dictionary.md`
- DDL de referencia: `docs/40_disenos_y_propuestas/PBI-276293-Modelado-ER/ddl/relational_model_ddl.sql`
- Historias relacionadas: US 276293, US 261487, US 277031
- Arquitectura base: `Analisis Épicas/shared/architecture_baseline.yaml`

---

## Procedure

1. Identificar entidades y relaciones del dominio para la tarea en curso.
2. Verificar si la entidad ya existe en el catálogo de entidades del dominio (tabla arriba).
3. Si es una entidad nueva: proponer tabla con prefijos correctos y registrar en el diccionario de datos.
4. Si se extiende una entidad existente: agregar campos con prefijo correcto; actualizar DDL y diccionario.
5. Generar o actualizar el DDL T-SQL respetando el estándar.
6. Actualizar `data_dictionary.md` con los atributos de la entidad.
7. Si hay cambio en el modelo ER, actualizar `Modelo_ER_mermaid.md` y regenerar snapshot PNG.
8. Registrar el cambio en el Historial de cambios del entregable correspondiente.

---

## Output Contract

### entityDefinition
- Nombre de tabla en PascalCase (español).
- Campos con prefijo correcto, tipo T-SQL, restricciones y descripción.
- PKs, FKs y UNIQUE constraints.

### ddlScript
- Script T-SQL completo con `CREATE TABLE`, constraints nombradas (`CONSTRAINT PK_...`, `CONSTRAINT FK_...`, `CONSTRAINT UQ_...`).
- Datos semilla (`INSERT INTO`) para catálogos cuando aplique.
- Encabezado con referencia a historia y convención.

### dataDictionaryEntry
- Entrada en formato Markdown para `data_dictionary.md`.
- Incluye: nombre de entidad, atributos con prefijo, tipo, descripción.

### mermaidUpdate
- Relación nueva o modificada en sintaxis Mermaid ER para agregar al diagrama.

### openQuestions
- Preguntas de negocio que afectan el modelo (cardinalidades, reglas de negocio, tipos de dato).
- Pendientes de validación antes de ejecutar DDL en ambiente de desarrollo.

---

## Quality Rules

- No usar prefijos mixtos ni inventar prefijos fuera del estándar.
- No crear tablas sin PK (`con` + nombre de tabla, IDENTITY).
- No omitir `fecCreacion` en tablas mutables.
- Tablas de auditoría e historial son inmutables: no llevan `fecActualizacion` y sus registros no se eliminan.
- Validar cardinalidades contra el diagrama Mermaid antes de generar FK.
- Indicar como "supuesto" cualquier decisión de diseño no confirmada por el negocio.
- Datos binarios (`bin`) solo si no hay alternativa de storage externo.

---

## Archivos clave del repositorio

| Propósito | Ruta |
|---|---|
| Entregable principal ER | `docs/40_disenos_y_propuestas/PBI-276293-Modelado-ER/entregable_modelado_er.md` |
| Diagrama Mermaid ER | `docs/40_disenos_y_propuestas/PBI-276293-Modelado-ER/diagrams/Modelo_ER_mermaid.md` |
| Drawio fuente | `docs/40_disenos_y_propuestas/PBI-276293-Modelado-ER/diagrams/MarchamoDigital.drawio` |
| Diccionario de datos | `docs/40_disenos_y_propuestas/PBI-276293-Modelado-ER/data_dictionary.md` |
| DDL modelo relacional | `docs/40_disenos_y_propuestas/PBI-276293-Modelado-ER/ddl/relational_model_ddl.sql` |
| DDL modelo de estados | `docs/40_disenos_y_propuestas/PBI-276293-Modelado-ER/ddl/state_model_es.sql` |
| Snapshot PNG | `docs/40_disenos_y_propuestas/PBI-276293-Modelado-ER/snapshots/` |
| Catálogo de estados etiquetas | `docs/40_disenos_y_propuestas/PBI-276293-Modelado-ER/entregable_estado_etiquetas.md` |

---

## Skill Collaboration

Cuando la solicitud requiera capacidades fuera del dominio de este Skill, coordinar con el Skill correspondiente siguiendo `framework/DECISION-TREE.md`.

| Necesidad | Skill |
|---|---|
| Decisiones de arquitectura de datos | `arquitecto-de-software` |
| Documentación técnica del modelo | `doc-tecnica` |

---

## Completion Checklist

Antes de considerar terminada cualquier tarea:

- □ Se respetó el Framework (Principles, Workflow, GIT-Policy, Conventions).
- □ Se siguió el Workflow.
- □ Se utilizó el Skill correcto.
- □ Se mantuvieron cambios mínimos.
- □ Se mantuvo consistencia con el proyecto.
- □ Se espera aprobación del desarrollador antes de cualquier operación Git.
