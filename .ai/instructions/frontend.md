# Instrucciones neutrales para mi-negociocr-frontend

- Confirma el árbol bajo `src/app` antes de editar. El source actual usa Angular standalone y organiza funcionalidades bajo `core`, `features` y `features/shared`.
- Mantén componentes centrados en presentación y coordinación de UI; coloca llamadas HTTP y lógica de integración en servicios de feature o servicios core existentes.
- Reutiliza modelos/DTOs y servicios existentes. No dupliques `HttpClient`, resolución de URL o reglas de sesión.
- Conserva el flujo de autenticación y el manejo de 401 existente.
- Resuelve `BusinessId` mediante la sesión/configuración existente y usa el formato que exige cada endpoint (ruta, query o body). Falla de forma explícita si falta.
- No guardes datos de un negocio bajo una clave compartida por varios negocios; si hay caché/local storage, incluye el contexto de negocio como ya hacen los servicios existentes.
- Antes de cambiar un contrato, inspecciona el controlador/caso de uso API correspondiente y sus consumidores.
- Las reglas antiguas que describen `src/app/shared` o una estructura distinta son documentación a conciliar, no evidencia suficiente para crear carpetas.

