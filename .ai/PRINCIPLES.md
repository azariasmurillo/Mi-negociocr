# Principios de trabajo

- **Fuente primero.** Confirma en el source actual antes de describir arquitectura, comportamiento, contratos o tenancy.
- **Verificar antes de afirmar.** Cada afirmación importante debe tener una ruta de archivo (y, cuando ayude, símbolo o línea) o marcarse como pendiente.
- **Auditoría antes de cambio.** Antes de modificar, delimita alcance, riesgos, consumidores y estado actual.
- **Reutilizar.** Busca componentes, casos de uso, servicios, repositorios y utilidades existentes antes de crear otros.
- **Cambio mínimo.** Cambia solo lo necesario para el objetivo autorizado; no aproveches la tarea para limpiar o renombrar sin autorización.
- **Sin expansión de alcance.** No conviertas una decisión local en una re-arquitectura, migración de datos o cambio de infraestructura.
- **Simplicidad.** Prefiere la solución más pequeña que preserve la evidencia, la seguridad y la claridad.
- **Consistencia.** Sigue convenciones reales del área tocada; si hay excepciones históricas, documéntalas en vez de ocultarlas.
- **Separación.** Mantén UI, aplicación, dominio, persistencia, integraciones y configuración en sus límites observables; no impongas una pureza que el source no tenga.
- **Control humano.** Las operaciones irreversibles o externas requieren autorización explícita y un plan verificable.
- **Evidencia.** Separa hechos `EXTRACTED` de inferencias, recomendaciones y decisiones pendientes.
- **Incertidumbre honesta.** Si falta evidencia o hay conflicto entre documentación y source, dilo y no rellenes el vacío con una suposición.

