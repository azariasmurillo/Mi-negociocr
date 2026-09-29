---
name: implement-from-audit
description: "Adaptador del Skill neutral implement-from-audit de Mi-NegocioCR para agentes compatibles."
---

# implement-from-audit — adaptador

Activa este Skill únicamente cuando exista un `AUDIT RESULT` suficientemente
claro y la persona usuaria autorice explícitamente implementar la decisión,
por ejemplo: «procede con la solución recomendada», «implementa lo que
encontramos» o «aplica la opción 2».

Un `AUDIT RESULT` por sí solo **NO** autoriza implementación. Si no existe
auditoría aprobada, aplica el workflow correspondiente y `audit-first` según
riesgo e incertidumbre. Si la auditoría quedó obsoleta o aparece una
dependencia que cambia el alcance, detente y pide una nueva decisión humana.

Lee y sigue el Skill neutral ubicado en:

`.ai/skills/implement-from-audit/SKILL.md`

Ese archivo es la única fuente de verdad del procedimiento, sus controles de
riesgo y su formato `IMPLEMENTATION RESULT`. Este archivo solo hace
descubrible el Skill; no duplica su procedimiento.
