---
name: audit-first
description: "Audita cambios de Mi-NegocioCR antes de implementar cuando el riesgo, la incertidumbre o el impacto lo justifican."
---

# audit-first

## Propósito

Convierte la política de `.ai/WORKFLOW.md` en un procedimiento operativo para investigar antes de implementar. Este Skill no reemplaza ni copia la fuente neutral: sus decisiones deben seguir `.ai/README.md`, `.ai/PRINCIPLES.md`, `.ai/PROJECT.md`, `.ai/ARCHITECTURE.md`, `.ai/TENANCY.md`, `.ai/SAFETY.md` y la instrucción de área correspondiente.

## Cuándo aplicarlo

Actívalo cuando el origen de un bug no esté claro, haya cambios de comportamiento, varios consumidores, contratos, persistencia, impacto transversal o documentación desalineada con el source. También aplícalo a pagos, créditos, saldos, cálculos monetarios, `BusinessId`/tenancy, autenticación, autorización, migraciones, seguridad, infraestructura o cambios irreversibles.

Para texto, CSS/estilos, cambios visuales simples, documentación o cambios locales evidentes y reversibles, sigue directamente el workflow proporcional de `.ai/WORKFLOW.md`; no conviertas un cambio bajo en una auditoría pesada.

## Autoridad y preparación

1. Lee `.ai/README.md` y respeta su orden de lectura.
2. Identifica el producto y carga solo la instrucción de área necesaria: `.ai/instructions/api.md`, `.ai/instructions/frontend.md` o `.ai/instructions/mobile.md`.
3. Trata el source actual como autoridad final. La documentación confirmada ayuda; Graphify es un mapa; las inferencias deben declararse.
4. Define qué acciones están autorizadas. Auditar no autoriza implementar, borrar, migrar, desplegar ni modificar secretos.

## Procedimiento

### 1. ENTENDER

Reformula el objetivo, el comportamiento esperado, el alcance autorizado y los productos afectados. No aceptes la solución sugerida como causa confirmada.

### 2. CLASIFICAR

Clasifica el riesgo como `BAJO`, `MEDIO` o `ALTO` según `.ai/WORKFLOW.md`. La profundidad debe ser proporcional: medio exige evidencia de impacto y verificación; alto exige el procedimiento estricto de este Skill.

### 3. ORIENTARSE

Empieza por el módulo estrechamente relacionado. Identifica símbolos, endpoints, DTOs, entidades, servicios, consumidores, persistencia y pruebas relevantes. Usa Graphify cuando reduzca el espacio de búsqueda o revele relaciones, dependencias, comunidades o caminos entre componentes; usa `query`, `path` o `explain` según corresponda.

### 4. VERIFICAR EL SOURCE

Abre los archivos relevantes y confirma cada hallazgo importante. Graphify no es evidencia final. Etiqueta los hallazgos como:

- `EXTRACTED`: observado directamente en un archivo o configuración.
- `INFERRED`: conclusión razonable derivada de evidencia, pero no declarada literalmente.
- `AMBIGUOUS`: evidencia incompleta o contradictoria.

Si documentación y source difieren, registra la discrepancia y prioriza el source.

### 5. TRAZAR EL FLUJO

Reconstruye solo el flujo real que corresponda. Puede incluir, por ejemplo, `Frontend → service → HTTP → Controller → UseCase → Persistence` o `Mobile → Repository → ApiClient → API → UseCase → Persistence`. No impongas esas capas si el módulo usa otro patrón; documenta la variante observada.

### 6. EVALUAR IMPACTO

Identifica productores, consumidores, contratos, persistencia, tenancy, efectos secundarios, concurrencia, transacciones, compatibilidad y posibles regresiones. Comprueba qué otros productos o módulos podrían depender del cambio.

### 7. REVISAR PRUEBAS

Busca pruebas existentes relacionadas y determina qué comportamiento ya protegen, qué escenario falta y qué pruebas deberían ejecutarse después. No ejecutes tests sin autorización explícita; reporta la recomendación, no una ejecución inexistente.

### 8. RESOLVER INCERTIDUMBRES

Si una incertidumbre puede cambiar significativamente el diseño, detén la implementación. Presenta qué se sabe, qué no se sabe, las opciones y el impacto de cada una. No elijas silenciosamente una decisión funcional o arquitectónica.

### 9. PROPONER EL CAMBIO MÍNIMO

Recomienda únicamente el cambio necesario para el objetivo, respetando patrones reales, preservando contratos cuando sea posible y evitando scope creep. `Recommended change` es una propuesta técnica basada en evidencia, no autorización para implementarla.

### 10. HANDOFF

Termina con esta salida compacta:

```text
AUDIT RESULT

Scope:
Risk: LOW | MEDIUM | HIGH
Current behavior:
Relevant flow:
Evidence:
Dependencies:
Risks:
Tests:
Uncertainties:
Recommended change:
Files likely affected:
Ready to implement: YES | NO
```

## Revisión financiera

Si el cambio afecta dinero, pagos, créditos, saldos, totales, descuentos, impuestos, redondeo o transacciones financieras, revisa los aspectos aplicables: unidades monetarias, precisión, redondeo, signos, idempotencia, transacciones, concurrencia, duplicación, rollback, auditoría/trazabilidad e impacto histórico. No asumas que todos aplican; indica cuáles verificaste y cuáles no corresponden.

## Revisión de tenancy

Para cambios relacionados con `BusinessId`, traza cómo se produce, transporta, valida y consume. Nunca trates el `BusinessId` de la UI como autorización suficiente: verifica la API. Cualquier posibilidad de cruce entre negocios es riesgo `ALTO` y debe quedar explícita en `Risks`.

## Uso de Graphify

Graphify es un mapa estructural, no la fuente final. Úsalo proporcionalmente para localizar símbolos, relaciones, dependencias, comunidades o caminos. Después verifica el source. No ejecutes `graphify update` durante una auditoría salvo autorización explícita y necesidad real.

## Eficiencia

- Empieza por el módulo y los símbolos directamente relacionados.
- Amplía la búsqueda solo cuando la evidencia lo requiera.
- Evita leer todo el repositorio o lanzar búsquedas gigantes sin una pregunta concreta.
- No uses Graphify para cambios triviales cuyo alcance ya está claro.
- No repitas el contenido completo de `.ai/`; referencia el documento aplicable.
- No investigues módulos sin relación demostrable con el objetivo.

## Control humano

No implementes automáticamente después de auditar cuando el usuario pidió solo auditoría, exista incertidumbre importante, un cambio de riesgo alto carezca de una decisión funcional o arquitectónica, o la acción esté prohibida por `.ai/SAFETY.md`. En esos casos entrega el `AUDIT RESULT` y espera instrucciones.
