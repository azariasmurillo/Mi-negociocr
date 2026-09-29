# Tenancy y aislamiento por negocio

## Hechos `EXTRACTED`

- La API recibe y valida `BusinessId` en numerosos contratos y rutas; las consultas y mutaciones de los módulos deben mantener ese contexto.
- `Program.cs` habilita JWT y autorización. Las claims y la resolución concreta de tenant deben verificarse en el controlador/caso de uso que se toque, no inferirse de una sola ruta.
- La API de créditos tiene rutas por negocio y casos de uso para cuentas, cargos, pagos, comunicaciones y cierres de órdenes. `IAppDbContext` expone las entidades de crédito y sus transacciones.
- El frontend prioriza el `BusinessId` de la sesión y usa configuración runtime como fallback explícito; `InventoryConfigService.requireBusinessId` falla si no puede resolverlo.
- El frontend propaga `BusinessId` en rutas, query params o body según el contrato concreto del servicio. No uniformes esos formatos sin verificar el endpoint.
- La aplicación móvil persiste `business_id` junto con la sesión y construye repositorios con ese identificador; algunos endpoints también lo envían en ruta o body.

## Reglas operativas

1. Nunca mezcles datos de dos negocios en memoria, consultas, cachés, almacenamiento local o respuestas.
2. No confíes solo en un `BusinessId` suministrado por UI: comprueba autenticación, autorización y validación del lado API.
3. Antes de cambiar una ruta, DTO, claim o repositorio, traza quién produce y consume el tenant.
4. Ante discrepancia entre claim, ruta, query o body, detente y documenta el conflicto; no elijas silenciosamente una fuente.
5. Trata créditos, pagos, inventario y órdenes como riesgo alto: conserva aislamiento y trazabilidad transaccional.

