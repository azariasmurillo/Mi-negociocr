# Proyecto Mi-NegocioCR

## Hechos verificados

- Workspace auditado: `C:\Mi-negociocr`.
- El workspace contiene tres productos relacionados, cada uno con su propio código y configuración:
  - `MiNegocioCR.Api`: API ASP.NET Core sobre .NET 8; el proyecto referencia Entity Framework Core 8 y PostgreSQL/Npgsql.
  - `mi-negociocr-frontend`: frontend Angular; `package.json` y `angular.json` indican Angular 21 y Material/SignalR en el stack usado.
  - `mi-negociocr-app`: aplicación Flutter; `pubspec.yaml` declara Flutter 3.x, Riverpod y Dio.
- También existen documentación, artefactos, copias/salidas históricas y `graphify-out/`. Esas colecciones no son automáticamente arquitectura productiva.
- La API configura autenticación JWT, autorización, EF Core y registros de dependencias en `MiNegocioCR.Api/API/Program.cs`.

## Exclusiones de arquitectura

No uses como fuente de arquitectura real ni mezcles en conclusiones resultados de:

`_api_test_copy_20260902`, `_api_test_isolated`, `_api_out`, `_build_out`, `_build_out_test`, `_testout`, `_test_out`, `_tmp`, `.build-tmp`, `.build-tmp2`.

El perfil documental de Marchamo, cuando aparezca en documentación histórica o en el framework de agentes, tampoco describe Mi-NegocioCR.

