# Framework Workflow

## Objetivo

Este documento define el flujo oficial de trabajo que el agente debe seguir dentro del AI Engineering Framework.

Todos los Agent Skills deben respetar este flujo.

Nunca deben omitir pasos.

---

# Flujo General

Toda solicitud debe seguir el siguiente proceso.

```
Comprender

↓

Analizar

↓

Planificar

↓

Buscar Skills

↓

Buscar Instructions

↓

Buscar implementaciones similares

↓

Implementar

↓

Validar

↓

Revisar

↓

Esperar aprobación del desarrollador

↓

Operaciones Git (únicamente si fueron solicitadas)
```

---

# Paso 1 - Comprender

Antes de generar código debes comprender completamente la solicitud.

Si existe información insuficiente:

Realiza únicamente una pregunta clara.

No inventes información.

---

# Paso 2 - Analizar

Analiza:

- alcance
- arquitectura
- módulos involucrados
- dependencias
- riesgos

No escribas código durante este paso.

---

# Paso 3 - Planificar

Antes de implementar:

Define mentalmente la solución.

Determina:

- qué archivos cambiar
- qué capas modificar
- qué patrones utilizar

---

# Paso 4 - Buscar Agent Skills

Determina la intención de la solicitud mediante `framework/DECISION-TREE.md`.

Resuelve la intención al Skill concreto usando el skill-map del Project Profile y los Packs instalados.

Si existe un Skill especializado:

Debe utilizarse.

Nunca responder únicamente utilizando conocimiento general.

---

# Paso 5 - Buscar Instructions

Determina qué Instructions aplican.

Ejemplo:

- Endpoint
- Repository
- Entity
- Validator
- Tests

Consulta esas instrucciones antes de escribir código.

---

# Paso 6 - Buscar implementaciones similares

Siempre buscar:

- módulos equivalentes
- endpoints equivalentes
- validadores
- repositorios
- pruebas
- mapeos

El proyecto ya contiene la mayoría de patrones necesarios.

Reutilízalos.

---

# Paso 7 - Implementar

Implementa únicamente lo solicitado.

No agregues funcionalidades adicionales.

No hagas refactors innecesarios.

No cambies código sin relación.

---

# Paso 8 - Validar

Antes de finalizar revisa:

Arquitectura

↓

Convenciones

↓

Compilación

↓

Tests

↓

Consistencia

↓

Cambios mínimos

---

# Paso 9 - Revisión

Antes de dar por terminado el trabajo:

Resume:

- cambios realizados
- archivos modificados
- decisiones importantes
- riesgos

---

# Paso 10 - Esperar aprobación

Siempre esperar que el desarrollador revise.

Nunca asumir que el trabajo está terminado.

---

# Paso 11 - Operaciones Git

Las operaciones Git son el último paso.

Nunca ejecutar automáticamente:

- git commit
- git push
- Pull Request
- Merge
- Release

Únicamente cuando exista autorización explícita.

---

# Flujo para solicitudes simples

```
Analizar

↓

Implementar

↓

Validar

↓

Esperar aprobación
```

---

# Flujo para nuevas funcionalidades

```
Analizar

↓

Skill Arquitectura

↓

Skill Implementación

↓

Validar

↓

Review

↓

Esperar aprobación
```

---

# Flujo para errores

```
Analizar

↓

Diagnosticar

↓

Corregir

↓

Validar

↓

Esperar aprobación
```

---

# Flujo para documentación

```
Analizar

↓

Documentar

↓

Validar

↓

Esperar aprobación
```

---

# Regla principal

La calidad siempre tiene prioridad sobre la velocidad.

Una respuesta correcta es mejor que una respuesta rápida.
