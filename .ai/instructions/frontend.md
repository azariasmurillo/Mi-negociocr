# Instrucciones neutrales para mi-negociocr-frontend

- Confirma el árbol bajo `src/app` antes de editar. El source actual usa Angular standalone y organiza funcionalidades bajo `core`, `features` y `features/shared`.
- Mantén componentes centrados en presentación y coordinación de UI; coloca llamadas HTTP y lógica de integración en servicios de feature o servicios core existentes.
- Reutiliza modelos/DTOs y servicios existentes. No dupliques `HttpClient`, resolución de URL o reglas de sesión.
- Conserva el flujo de autenticación y el manejo de 401 existente.
- Para contratos autenticados que exigen `BusinessId`, resuélvelo mediante la sesión/configuración existente y usa el formato de cada endpoint (ruta, query o body); falla explícitamente si falta. Otros contratos lo resuelven del JWT en la API. En el catálogo público se envía el slug y la API resuelve el negocio con `PublicStoreResolver`; no añadir un `BusinessId` arbitrario ni exigir sesión del comprador. Conserva siempre el aislamiento indicado en `.ai/TENANCY.md`.
- No guardes datos de un negocio bajo una clave compartida por varios negocios; si hay caché/local storage, incluye el contexto de negocio como ya hacen los servicios existentes.
- Antes de cambiar un contrato, inspecciona el controlador/caso de uso API correspondiente y sus consumidores.
- Las reglas antiguas que describen `src/app/shared` o una estructura distinta son documentación a conciliar, no evidencia suficiente para crear carpetas.

