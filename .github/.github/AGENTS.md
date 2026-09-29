# AI Engineering Framework — Agent Instructions

Este documento es el punto de entrada oficial del AI Engineering Framework (AEF).

Define cómo el agente debe operar dentro de cualquier repositorio que utilice este Framework.

Es independiente de la herramienta de IA (Cursor, Claude Code, OpenAI Codex, Gemini CLI, GitHub Copilot u otras).

---

# Framework

Este repositorio utiliza el AI Engineering Framework ubicado en:

`.github/framework/` (o la raíz del paquete AEF)

El Framework gobierna el comportamiento global del agente.

Todos los Roles, Agent Skills e Instructions deben respetar las reglas definidas por el Framework antes de ejecutar cualquier tarea.

Documentos del Framework:

- `framework/PRINCIPLES.md` — Principios globales del Framework.
- `framework/WORKFLOW.md` — Flujo oficial del proceso de desarrollo.
- `framework/DECISION-TREE.md` — Árbol de decisión por intención.
- `framework/GIT-POLICY.md` — Política oficial para operaciones Git y plataformas.
- `framework/CONVENTIONS.md` — Convenciones de nombres, organización y estilo.

Arquitectura del producto:

- **Core** — `framework/` + `skills/`
- **Packs** — `packs/`
- **Project Profiles** — `project-profiles/`

En caso de contradicción entre este documento y el Framework:

**El Framework tiene prioridad.**

---

# Flujo del Framework

Para cualquier solicitud, el agente debe seguir siempre este flujo:

1. Identificar el Rol solicitado por el usuario.
2. Cargar el documento correspondiente ubicado en `framework/roles/`.
3. Consultar el Framework.
4. Cargar el Project Profile activo (si existe).
5. Determinar la **intención** mediante `framework/DECISION-TREE.md`.
6. Resolver la intención al Agent Skill mediante el skill-map del Project Profile y los Packs instalados.
7. Cargar únicamente los Agent Skills necesarios.
8. Cargar únicamente las Instructions necesarias para el repositorio.
9. Ejecutar la tarea respetando el Framework.

Nunca omitas este flujo.

---

# Separación de responsabilidades

El Framework distingue claramente estos conceptos.

## Roles

Definen el comportamiento que debe seguir el agente.

Los Roles gobiernan:

- comportamiento
- responsabilidades
- restricciones
- flujo de trabajo
- estados
- entregables

Ubicación:

`framework/roles/`

---

## Agent Skills

Los Agent Skills contienen conocimiento especializado para resolver problemas específicos.

Un mismo Rol puede utilizar distintos Skills.

Un mismo Skill puede ser utilizado por distintos Roles.

Ubicaciones:

- Core: `skills/`
- Packs: `packs/<pack>/skills/`
- Project Profile: `project-profiles/<project>/skills/` (solo si es exclusivo del proyecto)

---

## Instructions

Las Instructions contienen reglas específicas del repositorio, arquitectura o capas del sistema.

Ubicación:

`.github/instructions/` (repositorio consumidor)

Nunca confundas Roles, Skills e Instructions.

---

## Packs

Conocimiento específico de tecnologías o integraciones.

No forman parte del Core.

---

## Project Profiles

Configuración específica del proyecto consumidor:

- stack-profile
- skill-map
- estándares
- dominio
- infraestructura

---

# Roles

Antes de iniciar cualquier tarea identifica el Rol solicitado por el usuario.

Si el usuario solicita actuar como alguno de los siguientes Roles:

- Tech Lead
- Analyst
- Architect
- Senior Developer
- Code Reviewer
- QA

Debes cargar primero el documento correspondiente ubicado en:

`framework/roles/`

y seguir estrictamente:

- misión
- responsabilidades
- entradas requeridas
- flujo obligatorio
- restricciones
- estados permitidos
- entregables
- checklist final

Los Roles constituyen la fuente oficial del comportamiento del agente.

---

# Orden de prioridad

Cuando exista información en varios lugares utiliza siempre el siguiente orden:

1. Código existente.
2. Framework (`framework/`)
3. Roles (`framework/roles/`)
4. Project Profile (`project-profiles/<project>/`)
5. Agent Skills (Core, Packs, Profile)
6. Instructions (`.github/instructions/`)
7. Documentación del repositorio.
8. Conocimiento general.

Si existe contradicción:

El código existente siempre es la fuente de verdad.

---

# Uso obligatorio de Agent Skills

Antes de iniciar cualquier tarea:

1. Consulta `framework/DECISION-TREE.md` y determina la intención.
2. Resuelve la intención con el skill-map del Project Profile y los Packs instalados.
3. Determina si existe un Skill especializado.
4. Carga únicamente los Skills necesarios.
5. Sigue completamente las instrucciones del Skill.
6. Si el Skill requiere colaboración con otros Skills, sigue dicho flujo.

Nunca ignores un Skill especializado para responder utilizando únicamente conocimiento general.

---

# Uso obligatorio de Instructions

Antes de modificar cualquier componente del sistema debes consultar las Instructions correspondientes.

Ejemplos:

- Entidades
- Endpoints
- Repositorios
- Casos de uso
- Validadores
- Infraestructura
- Tests
- Arquitectura específica del repositorio

Si existe una Instruction aplicable, debes seguirla.

---

# Principios generales

El agente debe:

- Priorizar el Framework antes que el conocimiento general.
- Respetar el Rol solicitado.
- Respetar el Project Profile activo.
- Utilizar únicamente los Agent Skills necesarios.
- Evitar duplicación de responsabilidades.
- Mantener cambios mínimos.
- Respetar la arquitectura existente.
- Solicitar aclaraciones cuando falte información crítica.
- Nunca asumir requisitos inexistentes.
- Nunca modificar el alcance aprobado.
- Nunca omitir el flujo definido por el Framework.

---

# Regla de Oro

Todo comportamiento del agente debe surgir del siguiente orden:

Framework

↓

Rol

↓

Project Profile

↓

Agent Skills

↓

Instructions

↓

Código existente

↓

Respuesta
