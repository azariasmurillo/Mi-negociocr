# Framework Principles

## Objetivo

Este documento define los principios que TODOS los Agent Skills, Instructions y Prompts del AI Engineering Framework deben seguir.

Estos principios son globales y no deben repetirse dentro de cada Skill.

---

# Principio 1 - El código es la fuente de verdad

Nunca asumir.

Nunca inventar.

Siempre verificar primero el código existente.

Cuando exista una contradicción entre documentación y código:

El código siempre tiene prioridad.

---

# Principio 2 - Reutilizar antes de crear

Antes de crear cualquier solución nueva siempre debes:

- Buscar implementaciones similares.
- Buscar patrones existentes.
- Buscar clases equivalentes.
- Buscar endpoints equivalentes.
- Buscar validadores equivalentes.
- Buscar pruebas equivalentes.

Nunca reinventes una solución que ya existe.

---

# Principio 3 - Consistencia

La consistencia tiene prioridad sobre las preferencias personales.

Respeta siempre:

- Arquitectura
- Naming
- Organización
- Estructura
- Convenciones existentes

---

# Principio 4 - Cambios mínimos

Realiza únicamente los cambios necesarios para resolver la solicitud.

Nunca:

- refactorices código no solicitado
- cambies nombres sin necesidad
- reformatees archivos completos
- modifiques código no relacionado

Cada línea modificada debe tener una razón clara.

---

# Principio 5 - Una responsabilidad

Cada componente debe tener una única responsabilidad.

Cada:

- Skill
- Prompt
- Instruction

debe resolver solamente un problema.

---

# Principio 6 - Skills especializados

Si existe un Skill especializado para resolver una tarea:

Debe utilizarse.

No responder únicamente utilizando conocimiento general.

---

# Principio 7 - Instructions especializadas

Si existe una Instruction para la capa que será modificada:

Debe consultarse antes de generar código.

---

# Principio 8 - Framework primero

Los Skills no deben duplicar reglas globales.

Si una regla aplica a todos los Skills debe pertenecer al Framework.

---

# Principio 9 - Calidad según el Project Profile

Todo código generado debe cumplir el stack y los estándares definidos en el Project Profile del proyecto consumidor.

El Core no fija un stack tecnológico.

Si no existe Project Profile o stack-profile:

1. Inferir el stack desde el código existente.
2. Solicitar aclaración cuando falte información crítica.
3. Nunca imponer un stack ajeno al proyecto.

---

# Principio 10 - Seguridad

Nunca ejecutar automáticamente acciones sobre Git.

Nunca:

- Commit
- Push
- Pull Request
- Merge
- Tag
- Release

Sin autorización explícita del desarrollador.

---

# Principio 11 - Validación

Antes de considerar terminada una tarea verificar:

□ Arquitectura

□ Convenciones

□ Compilación

□ Tests

□ Cambios mínimos

□ Consistencia

---

# Principio 12 - Simplicidad

Siempre preferir la solución más simple.

Una solución más compleja solamente es válida si aporta un beneficio claro.

---

# Principio 13 - Evolución continua

El Framework debe mejorar continuamente.

Las mejoras deben:

- Reducir mantenimiento.
- Reducir duplicación.
- Mejorar la calidad.
- Facilitar futuras modificaciones.

Nunca aumentar complejidad sin una razón clara.

---

# Principio 14 - Control humano

El desarrollador siempre tiene la decisión final.

El agente asiste.

Nunca reemplaza el criterio del desarrollador.
