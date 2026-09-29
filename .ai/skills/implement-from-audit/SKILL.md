---
name: implement-from-audit
description: "Implementa de forma mínima y verificable una decisión humana aprobada a partir de un AUDIT RESULT de Mi-NegocioCR. Úsalo cuando el usuario autorice proceder con la solución o el alcance definido por una auditoría; no se activa solo por la existencia de una auditoría."
---

# implement-from-audit

## Propósito

Convierte una auditoría técnica aprobada y una autorización humana explícita
en un cambio mínimo, trazable y verificable. Complementa `audit-first`; no lo
reemplaza ni duplica su procedimiento. La fuente de autoridad continúa siendo
`.ai/` y, para las reglas de auditoría, `.ai/skills/audit-first/SKILL.md`.

## Cuándo aplicarlo

Aplicarlo solo cuando estén presentes:

1. una auditoría previa suficientemente clara;
2. una recomendación o decisión aprobada;
3. un alcance de archivos/componentes identificable; y
4. autorización humana para implementar.

Son señales válidas frases como «implementa lo encontrado en la auditoría»,
«procede con la solución propuesta» o «aplica la opción 2». Un `AUDIT RESULT`
por sí solo no autoriza cambios.

Si el usuario pidió solo investigar, revisar o auditar, detenerse sin editar.
La autorización para implementar no autoriza commit, push, merge, PR, deploy,
migraciones aplicadas, cambios de infraestructura, secretos ni operaciones
destructivas.

## Fuente neutral y entrada esperada

Seguir `.ai/README.md`, `.ai/PRINCIPLES.md`, `.ai/WORKFLOW.md`,
`.ai/PROJECT.md`, `.ai/ARCHITECTURE.md`, `.ai/TENANCY.md`, `.ai/SAFETY.md` y
la instrucción de área aplicable. Usar la auditoría como contrato de alcance.

Recuperar, cuando estén disponibles:

`Scope`, `Risk`, `Current behavior`, `Relevant flow`, `Evidence`,
`Dependencies`, `Risks`, `Tests`, `Uncertainties`, `Recommended change`,
`Files likely affected` y `Ready to implement`.

Identificar exactamente qué decisión humana fue aprobada. No inventar
aprobación, alcance, evidencia ni resolución de incertidumbres.

## Pre-flight obligatorio

Antes de editar:

1. Confirmar que `Ready to implement` permite proceder.
2. Confirmar que las incertidumbres bloqueantes fueron resueltas.
3. Confirmar la recomendación u opción concreta autorizada.
4. Confirmar que la autorización cubre todas las acciones necesarias.
5. Releer solo los archivos fuente relevantes si hace falta comprobar que no
   cambiaron desde la auditoría.
6. Si el source cambió de forma que invalida la auditoría, detenerse y
   solicitar una nueva auditoría o decisión.
7. Para riesgo `ALTO`, comprobar antes de editar las invariantes señaladas
   por la auditoría.

## No re-auditar todo

No repetir `audit-first`, no volver a mapear el sistema completo, ni producir
otra auditoría extensa. Confiar en la auditoría aprobada salvo que el source
haya cambiado, falte evidencia necesaria, exista una contradicción o el cambio
real exceda el alcance auditado.

Usar Graphify solo si aparece una duda estructural nueva que afecte
directamente la implementación. No ejecutar `graphify update` como parte
automática de este Skill.

## Implementación controlada

1. Mantener el diff mínimo y limitado al alcance aprobado.
2. Seguir los patrones reales del módulo.
3. Reutilizar tipos, servicios, casos de uso, repositorios, helpers y
   utilidades existentes.
4. No introducir capas, refactors, renombres, migraciones ni mejoras no
   autorizadas.
5. Preservar contratos y consumidores identificados.
6. Modificar archivos fuera del alcance solo si existe una dependencia
   técnicamente necesaria y está cubierta por la autorización.
7. Si aparece una dependencia nueva que cambia materialmente el alcance,
   detenerse, reportarla y pedir nueva autorización.

## Riesgo alto e invariantes

Para `ALTO`, conservar explícitamente las invariantes identificadas por la
auditoría. Según corresponda, verificar:

- finanzas: precisión, redondeo, signos, idempotencia, transacciones,
  concurrencia, duplicación, rollback, trazabilidad e historial;
- tenancy: producción, transporte, validación y consumo de `BusinessId`, sin
  cruces entre negocios;
- auth/security: autenticación, autorización, validaciones y exposición de
  información.

No asumir que todos los puntos aplican: usar los que la auditoría haya
identificado y declarar los que no se comprobaron.

## Tests, migraciones y acciones externas

Usar la auditoría para seleccionar la verificación más estrecha. Cuando sea
razonable, añadir una prueba de regresión que falle con el comportamiento
anterior y pase con el nuevo. No cambiar pruebas solo para hacerlas pasar.

No ejecutar builds o tests si no están autorizados. Si están autorizados,
ejecutar primero la verificación focalizada y reportar exactamente el comando
y resultado.

Crear código que requiera una migración no autoriza crearla ni aplicarla.
Deploy, cambios de infraestructura, secretos, operaciones destructivas y
operaciones Git de escritura siguen sujetos a `.ai/SAFETY.md` y a
autorización explícita.

## Auto-verificación del diff

Después de implementar:

1. releer los archivos modificados;
2. comprobar que cada cambio corresponde a la decisión aprobada;
3. buscar modificaciones accidentales y scope creep;
4. revisar contratos, consumidores e invariantes afectados;
5. revisar las pruebas agregadas o modificadas;
6. verificar que no se expusieron secretos ni datos sensibles;
7. retirar cualquier cambio innecesario antes de entregar.

## Salida obligatoria

Terminar con:

```text
IMPLEMENTATION RESULT

Audit used:
Approved decision:
Risk:
Implemented:
Files changed:
Tests added/changed:
Verification executed:
Verification result:
Invariants checked:
Deviations from audit:
Remaining risks:
Manual actions required:
Ready for human review: YES | NO
```

`Deviations from audit` debe ser explícito. Usar `NONE` si no hubo
desviaciones. Si una desviación técnicamente necesaria no cambia materialmente
el diseño, explicar qué cambió, por qué, la evidencia y el impacto. Si cambia
materialmente el alcance o diseño, no implementarla sin nueva autorización.

