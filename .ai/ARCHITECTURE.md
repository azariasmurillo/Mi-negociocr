# Arquitectura observada

Este documento resume hechos `EXTRACTED` del source actual. No es un mandato para rediseñar los productos.

## API

`MiNegocioCR.Api` contiene `API`, `Application`, `Domain`, `Infrastructure` y `Migrations`. `API/Program.cs` configura ASP.NET Core, JWT, autorización, EF Core/Npgsql, SignalR y DI. Los controladores reciben casos de uso de `Application.Interfaces`; los casos de uso viven bajo `Application/UseCases` y los DTOs bajo `Application/DTOs`; las entidades están bajo `Domain/Entities`; la persistencia EF está en `Infrastructure/Persistence`.

El patrón no es uniforme en todos los módulos. Hay interfaces e implementaciones de repositorio registradas en DI, pero muchos casos de uso acceden a `IAppDbContext`, que expone `DbSet` y `SaveChangesAsync`. El módulo de créditos es evidencia explícita de esa coexistencia: `API/Controllers/CreditAccountsController.cs` delega a interfaces de casos de uso; `Application/UseCases/CreditAccounts/*.cs` usa `IAppDbContext`; `Infrastructure/Persistence/AppDbContext.cs` implementa ese contrato y expone cuentas, transacciones y comunicaciones. No impongas “todo debe pasar por repositorios” como regla universal sin revisar el módulo.

## Frontend

`mi-negociocr-frontend/src/app` usa Angular standalone. El source separa `core`, `features` y utilidades compartidas dentro de `features/shared`; los componentes de pantalla coordinan UI y los servicios de feature encapsulan llamadas `HttpClient` y transformaciones. `core/auth/auth-token.store.ts`, `core/config/runtime-env.ts` y `features/inventory/services/inventory-config.service.ts` participan en sesión, URL de API y resolución de `BusinessId`. Las reglas de Cursor que mencionan `src/app/shared` no coinciden exactamente con esa ruta actual; confirma el árbol real antes de crear archivos.

## Móvil

`mi-negociocr-app/lib` tiene `core`, `features`, modelos, pantallas y repositorios de datos por feature. `main.dart` inicializa Flutter/Riverpod y Sentry cuando está habilitado; `app.dart` decide entre login y shell según `sessionProvider`. `ApiClient` centraliza Dio, Bearer, timeouts, refresh y respuesta ante 401. `SessionStorage` usa almacenamiento seguro para token, URL, usuario, `BusinessId` y, cuando se solicita recordar acceso, la contraseña guardada; los repositorios reciben el cliente y, cuando corresponde, el `BusinessId`. La navegación observada usa `MaterialApp`/`Navigator`; no se debe asumir un router adicional sin verificarlo.

## Regla de lectura

Los documentos históricos y reglas de proveedor son contexto. Si contradicen el source actual, conserva el hecho del source, registra la discrepancia y evita “corregir” documentación fuera del alcance autorizado.
