# Auditoría final — Validación de visión AEF (re-auditoría)

**Estado:** Re-auditoría post-generalización  
**Audiencia:** Tech Lead  
**Fecha:** 2026-07-13  
**Afirmación bajo prueba:**  
*“Puede utilizarse con cualquier IA y en cualquier proyecto sin modificar el Framework.”*

**Alcance auditado:** Core (`AGENTS.md`, `framework/`, `skills/`) + Packs (`packs/`)  
**Fuera de alcance (correcto):** `project-profiles/marchamo/` (consumidor) y documentos históricos `ANALISIS-*.md`

---

## Veredicto

**CUMPLE.**

El AI Engineering Framework puede utilizarse con cualquier asistente de IA y en cualquier proyecto **sin modificar el Core ni los Packs**.

La configuración específica se aporta exclusivamente mediante un **Project Profile** nuevo (y la selección de Packs opcionales).

---

## 1. ¿Existe dependencia de una IA específica?

**Respuesta: No.**

| Componente | Hallazgo |
|---|---|
| `AGENTS.md` | Actor genérico (“el agente”). Menciona herramientas solo como ejemplos de independencia |
| `framework/*` | Sin vendor |
| `skills/` (Core) | Sin vendor |
| `packs/` | Sin vendor |

**Bloqueantes:** ninguno.  
**Mejoras:** adapters de entry point (`CLAUDE.md`, `GEMINI.md`, etc.) en el repo consumidor — fuera del Framework.

---

## 2. ¿Existe dependencia de un proyecto específico?

**Respuesta: No en Core ni Packs.**

| Zona | Dependencia de Marchamo/INS/otro proyecto | Clasificación |
|---|---|---|
| `framework/` | Ninguna | Cumple |
| `skills/` (Core) | Ninguna | Cumple |
| `packs/` | Ninguna | Cumple |
| `project-profiles/marchamo/` | Sí (esperado) | No es incumplimiento |
| `README.md` | Cita `marchamo` solo como ejemplo de profile | Mejora cosmética |

**Bloqueantes:** ninguno.

---

## 3. ¿Qué no puede reutilizarse en un proyecto completamente diferente?

| Elemento | ¿Reutilizable sin cambios? |
|---|---|
| Roles | **Sí** |
| Workflow | **Sí** |
| Principles | **Sí** |
| Git Policy | **Sí** |
| Decision Tree (intenciones) | **Sí** |
| `AGENTS.md` | **Sí** |
| Core skills | **Sí** |
| Packs | **Sí** (conocimiento de stack/integración, sin proyecto hardcodeado) |
| Skills del profile Marchamo | **No** (correcto: no deben instalarse en otro proyecto) |

---

## 4. ¿Qué debe parametrizarse para un proyecto nuevo?

Todo en un **Project Profile** nuevo (sin tocar Core/Packs):

| Parámetro | Dónde |
|---|---|
| Identidad / org / repos | Profile |
| `stack-profile` | Profile |
| `skill-map` | Profile |
| `docs.layout` / plantillas | Profile |
| `standards.root` | Profile |
| ADO / infra | Profile |
| Skills de dominio | Profile `skills/` (opcional) |

Instalación válida:

1. Instalar Core sin cambios  
2. Instalar Packs necesarios sin cambios  
3. Crear `project-profiles/<nuevo-proyecto>/`  
4. No copiar `project-profiles/marchamo/` salvo que el consumidor sea Marchamo

---

## 5. Simulación: proyecto nuevo + Claude/Gemini

**Escenario:** proyecto “Acme”, stack Node, asistente Claude Code.

| Paso | ¿Requiere modificar el Framework? |
|---|---|
| Usar Roles / Workflow / Principles / Git Policy / Decision Tree | **No** |
| Usar `AGENTS.md` | **No** |
| Apuntar Claude a `AGENTS.md` | **No** (config del repo consumidor) |
| Crear Project Profile `acme` | **No** (se añade profile; Core/Packs intactos) |
| Usar Core skills (`arquitecto-de-software`, `agent-customization`, `faro-del-conocimiento`) | **No** — resuelven vía skill-map / `docs.layout` |
| Usar Packs instalados | **No** — sin referencias a Marchamo/INS |

**Qué lo impedía antes (ya corregido):**

1. ~~Core skills con nombres/paths Marchamo~~ → generalizados  
2. ~~Packs con org/estándares/repos Marchamo~~ → generalizados  
3. ~~Cross-refs a skills del profile Marchamo desde Core/Packs~~ → reemplazados por intenciones + skill-map  

---

## Conclusión frente a la visión

| Dimensión | ¿Cumple? |
|---|---|
| Cualquier asistente de IA | **Sí** |
| Cualquier proyecto **sin modificar el Framework** | **Sí** |

### Bloqueantes

Ninguno.

### Mejoras (no impiden la visión)

1. Mencionar `marchamo` como ejemplo de profile en `README.md` (opcional quitar).  
2. Adapters de entry point por herramienta en repos consumidores.  
3. Archivar o aislar `ANALISIS-*.md` / auditoría previa para no confundirlos con el producto.

---

## Respuesta final a la afirmación

**Sí.** El AI Engineering Framework puede utilizarse con cualquier IA y en cualquier proyecto sin modificar el Framework.

La arquitectura queda **validada y lista para congelarse**. El producto puede pasar a fase de implementación y evolución.
