# Framework Conventions

## Objetivo

Este documento define las convenciones oficiales del AI Engineering Framework (AEF).

Todos los Agent Skills, Instructions, Prompts, Packs, Project Profiles y futuros componentes deben seguir estas convenciones.

---

# Principio

Una única forma de hacer las cosas.

Si existe una convención oficial:

Debe respetarse.

No crear nuevas convenciones.

---

# Estructura del producto

```
AEF/

AGENTS.md

framework/

skills/

packs/

project-profiles/

instructions/          # en el repositorio consumidor

prompts/               # en el repositorio consumidor

agents/                # en el repositorio consumidor
```

Cada carpeta tiene una única responsabilidad.

---

# Core (`framework/`)

Contiene:

- Principios
- Workflow
- Decision Tree
- Políticas
- Convenciones
- Roles

Nunca contiene:

- lógica específica de un proyecto
- stack tecnológico fijo
- conocimiento de una herramienta de IA concreta

---

# Skills del Core (`skills/`)

Skills genéricas e independientes de stack y de proyecto.

Cada Skill debe resolver un único problema.

No debe contener:

- reglas globales
- políticas Git
- principios generales
- IDs, orgs o dominio de un proyecto concreto

Debe reutilizar el Framework.

---

# Packs (`packs/`)

Contienen conocimiento específico de tecnologías o integraciones.

Ejemplos de packs:

- dotnet
- azure
- ado
- docs
- android
- cosmos
- github-review

Un Pack NO es parte del Core.

Un Pack NO debe hardcodear un Project Profile concreto.

---

# Project Profiles (`project-profiles/`)

Contienen la configuración específica de un proyecto consumidor.

Incluyen:

- identidad del proyecto
- stack-profile
- skill-map (intención → Skill)
- estándares
- dominio
- infraestructura
- skills exclusivas del proyecto (si aplica)

Un Project Profile consume el Core y los Packs; no los redefine.

---

# Instructions

Cada Instruction explica cómo trabajar sobre una capa específica.

Ejemplos:

- Endpoints
- Repositories
- Entities
- Validators
- Tests

Viven en el repositorio consumidor.

---

# Prompts

Los Prompts contienen texto reutilizable.

Nunca contienen reglas del Framework.

Nunca contienen lógica de negocio.

---

# Agents

Los Agents son orquestadores.

Su responsabilidad es decidir.

No duplicar lógica existente en Skills.

---

# Nombres

Utilizar nombres simples.

Buenos ejemplos:

- add-operation
- review-code
- fix-build
- new-module

Evitar nombres demasiado largos.

Evitar nombres de proyecto o de herramienta de IA en Skills del Core o de Packs.

---

# Estructura de un Skill

Cada Skill debe contener únicamente:

Objetivo

↓

Cuándo utilizarlo

↓

Entradas

↓

Proceso

↓

Resultado

↓

Restricciones

No agregar documentación innecesaria.

---

# Estructura de una Instruction

Cada Instruction debe contener:

Objetivo

↓

Buenas prácticas

↓

Proceso

↓

Errores comunes

↓

Checklist

---

# Estructura de un Prompt

Cada Prompt debe contener:

Contexto

↓

Objetivo

↓

Resultado esperado

↓

Restricciones

---

# Reutilización

Antes de crear:

- un Skill
- una Instruction
- un Prompt
- un Pack
- un Project Profile

verificar si ya existe uno equivalente.

---

# Evolución

Cada nuevo componente debe:

Reducir mantenimiento.

Reducir duplicación.

Mantener consistencia.

Aumentar reutilización.

Nunca aumentar complejidad sin un beneficio claro.
