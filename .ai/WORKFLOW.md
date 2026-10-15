# Flujo de trabajo para agentes

## 1. Auditar antes de implementar

1. Reformula el objetivo y los límites autorizados.
2. Identifica producto, módulo, entradas, salidas y archivos candidatos.
3. Consulta el mapa de `.ai/README.md`, lee la documentación canónica del módulo y las decisiones relevantes, distinguiendo referencias vigentes, propuestas e historia. Si hay conflicto con el código, registra la diferencia; no elijas un contrato por suposición.
4. Usa Graphify cuando aporte valor para descubrir estructura, relaciones, dependencias o impacto. No es necesario para cambios triviales o locales cuyo alcance ya esté claro. Para cambios medios o altos, o investigaciones transversales, prefiérelo como mapa inicial cuando sea útil; el source continúa siendo la autoridad final.
5. Inspecciona el source real y la configuración relevante; la documentación no demuestra que una funcionalidad exista, funcione o esté desplegada.
6. Registra hechos extraídos, contradicciones, riesgos y decisiones pendientes.

## 2. Clasificar el riesgo

- **Bajo:** texto; CSS/estilos; cambios visuales aislados; documentación; refactors locales claramente reversibles sin alterar contratos.
- **Medio:** comportamiento local; DTOs; endpoints; servicios; persistencia no sensible; cambios con consumidores conocidos pero impacto acotado.
- **Alto:** pagos; créditos; saldos; cualquier cálculo o dato monetario; `BusinessId`/tenancy; autenticación; autorización; migraciones; operaciones destructivas; secretos; infraestructura/deploy; cambios irreversibles; cambios transversales importantes entre API/frontend/mobile.

La profundidad de la auditoría debe ser proporcional al riesgo; no conviertas riesgo bajo o medio en una ceremonia innecesaria. Para riesgo medio, exige evidencia de impacto, plan de reversión y verificación proporcional. Para riesgo alto, aplica audit-first estricto: investiga impacto, invariantes, consumidores, tests y rollback antes de implementar. Si existe una decisión funcional o arquitectónica importante no resuelta, detén la implementación y pide una decisión humana.

## 3. Implementar con trazabilidad

Usa los puntos de extensión existentes. Mantén el diff pequeño. Conserva compatibilidad con consumidores conocidos. No introduzcas una nueva capa solo para satisfacer una regla genérica si el source del módulo usa otra integración coherente.

Durante la implementación, identifica decisiones y cambios de comportamiento que afecten arquitectura, modelos, contratos API, tenancy, seguridad, inventario/ventas, Marketplace, WhatsApp/IA, migraciones o configuración/despliegues. Anota los documentos afectados para actualizarlos al validar el resultado; no conviertas cada línea de código en una entrada documental.

## 4. Verificar y entregar

Relee los archivos cambiados, revisa referencias, límites de tenancy y contratos. Ejecuta únicamente las verificaciones autorizadas. Si builds o tests están prohibidos, no los ejecutes ni los presentes como realizados. Reporta comandos, evidencia, incertidumbre y archivos modificados. Antes de cerrar una implementación significativa, aplica la sección 5 y enlaza los documentos actualizados o declara por qué quedaron pendientes.

**Deploy a producción (push/merge a `main`/`master`):** orden fijo — (1) unit tests completos de cada repo tocado, (2) actualizar documentación del cambio, (3) commit y push. Detalle: `.cursor/rules/deploy-pipeline.mdc` y `DEPLOY.md` §3.

## 5. Memoria técnica permanente

Vigente desde el 15 de octubre de 2026. Este procedimiento mantiene la documentación existente como memoria compartida. Aplica a cualquier herramienta que cargue estas instrucciones y conserva AUDIT-FIRST y los límites de [SAFETY.md](./SAFETY.md); no garantiza carga automática desde un checkout aislado.

### Cuándo actualizar

- Un cambio es significativo si altera un comportamiento documentado, una decisión técnica, arquitectura, datos/contratos, consumidores, aislamiento, seguridad, migraciones o la operación de una función. Actualizar los documentos correspondientes cuando la implementación esté terminada y validada con las comprobaciones autorizadas, antes del cierre del trabajo; no esperar al deploy para registrar código implementado.
- Texto, formato, ajustes visuales menores y refactors internos sin impacto contractual no requieren notas extensas ni un historial nuevo. Si cambian un comportamiento o una decisión ya documentados, corregir esa referencia de forma breve.
- Si la validación falla, está prohibida o no pudo ejecutarse, no declarar el trabajo validado: registrar el resultado disponible, el motivo y el pendiente en la nota pertinente o en el informe, según el alcance autorizado.

### Dónde y cómo registrar

1. Elegir la referencia del módulo mediante el [mapa canónico](./README.md). Actualizar el comportamiento vigente y añadir una entrada breve por fecha/módulo en el registro o nota existente más cercano al código. Usar el mapa, los índices y registros existentes; no crear un changelog global ni otro índice mientras cubran la necesidad. Si falta una referencia, justificar el ajuste mínimo de navegación dentro del alcance autorizado, evitando estructuras paralelas.
2. Documentar lo que otra IA necesita para continuar: objetivo, antes/después, decisiones y motivos, archivos/símbolos y consumidores relevantes, contratos y migraciones afectados, verificaciones, limitaciones y pendientes. Omitir campos que no apliquen, con explicación breve si puede haber duda.
3. Conservar las decisiones anteriores con su fecha y contexto. Si quedan superadas, identificarlas como históricas y enlazar la referencia vigente; no borrar motivos ni sustituir cuerpos de copias con notas propias.
4. Mantener una única entrada principal del cambio. En otras copias o módulos consumidores, añadir solo la referencia o la información específica necesaria. Actualizar índices únicamente si cambió la ruta de navegación o se añadió una referencia pertinente.
5. Identificar la revisión: repo, rama y commit si existe; si son cambios sin commit, indicarlo. No crear commits para completar la nota. El historial de conversaciones y el informe final no sustituyen la actualización documental de un cambio significativo terminado.
6. Separar evidencia de implementación, integración y despliegue según [Estados y evidencia](./README.md). Para migraciones, distinguir existente en código, aplicada en desarrollo y aplicada en producción con evidencia por entorno. No asumir aplicación por Git, configuración o arranque previsto.

Formato breve orientativo dentro del documento existente, solo con datos aplicables:

- Fecha / módulo / objetivo.
- Antes → después; decisión y motivo.
- Archivos, símbolos y consumidores; contrato/migración/configuración afectada.
- Verificación: comando o comprobación realmente ejecutada, resultado y alcance; si no se ejecutó, motivo. No inventar conteos ni atribuir resultados históricos a esta revisión.
- Revisión: repo/rama/commit existente o «sin commit».
- Estado: implementación; integración y alcance comprobado; despliegue con evidencia o «no verificado»; aplicación de migraciones por entorno si corresponde.
- Limitaciones / pendientes y referencia a la decisión todavía necesaria.

### Comprobación antes de entregar

Releer la nota contra el source y la revisión indicados; verificar enlaces locales añadidos, separar planes de hechos, comprobar que no se perdieron notas ajenas y que los estados no atribuyen producción sin evidencia. Reportar la documentación editada y las validaciones realizadas; para un cambio trivial, basta indicar que no requiere actualización extensa. Si la documentación quedó pendiente, decir cuál y por qué, sin presentar la memoria como completa.

El mantenimiento documental relacionado se incluye en la implementación autorizada. No amplía el permiso para ejecutar tests, migraciones, operaciones Git, acceso a producción ni otras acciones restringidas. Si el usuario pide solo auditoría o prohíbe editar documentación, respetar ese límite y dejar los cambios documentales propuestos en el informe. Nunca registrar secretos, credenciales ni datos personales.
