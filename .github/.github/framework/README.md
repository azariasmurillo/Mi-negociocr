# AI Engineering Framework (AEF)

## Objetivo

El AI Engineering Framework define la forma oficial en que un agente de IA debe trabajar dentro de un repositorio.

Su propósito NO es enseñar a programar.

Su propósito es enseñar cómo trabaja el proyecto que lo consume.

El Framework centraliza reglas, principios y convenciones para que todos los Agent Skills compartan el mismo comportamiento.

---

# Filosofía

El Framework busca que el agente actúe como un miembro más del equipo de desarrollo.

Antes de escribir código debe:

1. Comprender el problema.
2. Consultar los Skills apropiados.
3. Consultar las Instructions correspondientes.
4. Buscar implementaciones similares.
5. Implementar utilizando los patrones existentes.
6. Validar el resultado.
7. Esperar aprobación humana antes de cualquier operación sobre Git.

---

# Arquitectura del producto

El AEF se organiza en tres capas. No se introducen conceptos fuera de este alcance.

## Core

Reglas genéricas e independientes de proyecto y de herramienta de IA.

Ubicación:

`framework/`

Incluye Principles, Workflow, Decision Tree, Git Policy, Conventions y Roles.

## Packs

Conocimiento específico de tecnologías o integraciones.

Ubicación:

`packs/`

Se instalan según el stack del proyecto. No pertenecen al Core.

## Project Profiles

Configuración específica de cada proyecto consumidor.

Ubicación:

`project-profiles/`

Incluye stack-profile, skill-map, estándares, dominio e infraestructura del proyecto.

Cada proyecto consumidor aporta su Project Profile; no forma parte del Core.

---

# Componentes del Core

## AGENTS.md

Punto de entrada oficial del Framework.

Contiene las reglas globales que siempre aplican.

Nunca contiene lógica específica de un Skill ni de un proyecto.

---

## Skills

Ubicación:

`skills/` (Core) y `packs/*/skills/` (Packs)

Cada Skill resuelve un único problema específico.

---

## Instructions

Ubicación:

`.github/instructions/` (en el repositorio consumidor)

Definen cómo implementar una capa específica del sistema.

---

## Framework

Ubicación:

`framework/`

Define principios comunes.

Los Skills deben reutilizar estos documentos en lugar de duplicar reglas.

---

## Prompts

Ubicación:

`.github/prompts/` (en el repositorio consumidor)

Prompts reutilizables para tareas repetitivas.

---

# Principios

Todos los componentes del Framework deben cumplir:

- Simplicidad.
- Consistencia.
- Reutilización.
- Mínimo mantenimiento.
- Mínima duplicación.
- Alta legibilidad.

---

# Regla principal

Si una mejora hace el Framework más complejo sin aportar un beneficio claro, no debe implementarse.

---

# Prioridad

Cuando exista información en varios lugares:

1. Código existente.
2. Framework (Core).
3. Project Profile.
4. Skills (Core y Packs instalados).
5. Instructions.
6. Documentación.
7. Conocimiento general.

---

# Objetivo final

Lograr que cualquier asistente de IA produzca soluciones consistentes con el proyecto consumidor, minimizando la necesidad de instrucciones manuales del desarrollador.
