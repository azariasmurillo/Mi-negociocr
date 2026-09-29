# Instrucciones neutrales para MiNegocioCR.Api

## Estructura observada

- `API`: composición, middleware, autenticación/autorización, controladores y contratos HTTP.
- `Application`: interfaces, DTOs, casos de uso y reglas de aplicación.
- `Domain`: entidades y conceptos de dominio.
- `Infrastructure`: EF Core, PostgreSQL/Npgsql, repositorios, servicios externos y persistencia.

## Reglas de trabajo

- Un controlador debe delegar en el caso de uso apropiado y conservar validación/HTTP que sea propia del borde; no muevas reglas de negocio al controlador.
- Reutiliza interfaces, DTOs, helpers y registros DI existentes antes de crear tipos paralelos.
- Verifica `Program.cs` y el contrato de la interfaz antes de añadir o cambiar una dependencia.
- Usa validación defensiva para entradas, nulls, identificadores y estados; devuelve el error del contrato existente sin filtrar detalles sensibles.
- Mantén autenticación JWT, autorización y `BusinessId` coherentes en cada endpoint.
- En persistencia, sigue el patrón real del módulo. Algunos módulos usan repositorios registrados; muchos casos de uso, incluido créditos, usan `IAppDbContext` directamente. No reemplaces uno por otro por preferencia abstracta.
- Para operaciones financieras, créditos, pagos o transacciones, identifica límites de transacción, idempotencia, concurrencia, redondeo y auditoría antes de cambiar código.
- Las migraciones existentes no son una invitación a ejecutar o recrear migraciones. Cualquier cambio de esquema requiere autorización y revisión.

## Evidencia de créditos

La ruta observada es `API/Controllers/CreditAccountsController.cs` → interfaces en `Application/Interfaces/CreditAccounts` → casos de uso en `Application/UseCases/CreditAccounts` → `IAppDbContext` en `Application/Interfaces/IAppDbContext.cs` → `Infrastructure/Persistence/AppDbContext.cs`. DI se registra en `API/Program.cs`. Esta traza es evidencia de este módulo, no una regla para todos los módulos.

