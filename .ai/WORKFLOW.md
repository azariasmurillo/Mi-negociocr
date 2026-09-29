# Flujo de trabajo para agentes

## 1. Auditar antes de implementar

1. Reformula el objetivo y los límites autorizados.
2. Identifica producto, módulo, entradas, salidas y archivos candidatos.
3. Usa Graphify cuando aporte valor para descubrir estructura, relaciones, dependencias o impacto. No es necesario para cambios triviales o locales cuyo alcance ya esté claro. Para cambios medios o altos, o investigaciones transversales, prefiérelo como mapa inicial cuando sea útil; el source continúa siendo la autoridad final.
4. Inspecciona el source real y la configuración relevante.
5. Registra hechos extraídos, contradicciones, riesgos y decisiones pendientes.

## 2. Clasificar el riesgo

- **Bajo:** texto; CSS/estilos; cambios visuales aislados; documentación; refactors locales claramente reversibles sin alterar contratos.
- **Medio:** comportamiento local; DTOs; endpoints; servicios; persistencia no sensible; cambios con consumidores conocidos pero impacto acotado.
- **Alto:** pagos; créditos; saldos; cualquier cálculo o dato monetario; `BusinessId`/tenancy; autenticación; autorización; migraciones; operaciones destructivas; secretos; infraestructura/deploy; cambios irreversibles; cambios transversales importantes entre API/frontend/mobile.

La profundidad de la auditoría debe ser proporcional al riesgo; no conviertas riesgo bajo o medio en una ceremonia innecesaria. Para riesgo medio, exige evidencia de impacto, plan de reversión y verificación proporcional. Para riesgo alto, aplica audit-first estricto: investiga impacto, invariantes, consumidores, tests y rollback antes de implementar. Si existe una decisión funcional o arquitectónica importante no resuelta, detén la implementación y pide una decisión humana.

## 3. Implementar con trazabilidad

Usa los puntos de extensión existentes. Mantén el diff pequeño. Conserva compatibilidad con consumidores conocidos. No introduzcas una nueva capa solo para satisfacer una regla genérica si el source del módulo usa otra integración coherente.

## 4. Verificar y entregar

Relee los archivos cambiados, revisa referencias, límites de tenancy y contratos. Ejecuta únicamente las verificaciones autorizadas. Si builds o tests están prohibidos, no los ejecutes ni los presentes como realizados. Reporta comandos, evidencia, incertidumbre y archivos modificados.
