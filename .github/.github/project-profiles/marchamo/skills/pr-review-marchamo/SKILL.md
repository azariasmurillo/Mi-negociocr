---
name: pr-review-marchamo
description: "Skill especializada para revisar Pull Requests de cualquier repositorio del ecosistema MarchamoDigital (Grupo-INS). Aplica los estándares canónicos de MarchamoDigital.Base: arquitectura por capas (Clean Architecture), .NET 10, FastEndpoints, MediatR/CQRS, Autofac, FluentValidation, Dapper/CosmosDB, xUnit, 22 controles de seguridad automatizados. USE WHEN: revisar un PR, hacer code review, validar cambios antes de merge, analizar calidad de código en MarchamoDigital.*."
argument-hint: "Indica el repo y número de PR a revisar, por ejemplo: 'revisa el PR #14 de MarchamoDigital.Auditoria.Correlacion'"
user-invocable: true
---

# PR Review — Ecosistema MarchamoDigital

---

## Framework Integration

Este Skill pertenece al Project Profile Marchamo del AI Engineering Framework.

Antes de ejecutar cualquier acción debe seguir obligatoriamente:

- `framework/PRINCIPLES.md` — Principios globales que gobiernan todo el ecosistema.
- `framework/WORKFLOW.md` — Flujo de trabajo oficial: comprender, analizar, planificar, implementar, validar, esperar aprobación.
- `framework/DECISION-TREE.md` — Árbol de decisión para selección de Skills.
- `framework/GIT-POLICY.md` — Política oficial para todas las operaciones Git.
- `framework/CONVENTIONS.md` — Convenciones de nombres, estructura y organización.

---

## Framework Role

**Responsabilidad de este Skill:**
Revisar Pull Requests de cualquier repositorio del ecosistema `Grupo-INS/MarchamoDigital.*` aplicando los estándares canónicos y generar un reporte local para revisión del desarrollador.

**Problema que resuelve:**
Los PRs en el ecosistema deben cumplir más de 20 reglas de arquitectura, API, seguridad, tests y CI/CD — este Skill las aplica sistemáticamente y produce un reporte estructurado.

**Responsabilidades que NO tiene este Skill:**
- Solo genera un reporte local (`docs/90_borradores/pr-review-*.md`) — NO publica comentarios, reviews ni aprobaciones en GitHub automáticamente.
- La publicación en GitHub requiere autorización explícita y se delega a `pr-review-publisher`.
- Ver `framework/GIT-POLICY.md` para la política completa de operaciones Git.

---

## Propósito

Revisar Pull Requests de **cualquier repositorio** del ecosistema `Grupo-INS/MarchamoDigital.*` aplicando
los estándares canónicos definidos en:

> **Alcance de esta skill (local, no publica en GitHub):**
> Esta skill SOLO genera un reporte local (`docs/90_borradores/pr-review-*.md`) para que el usuario lo lea, negocie y ajuste antes de cualquier acción sobre el PR. **NO debe** publicar comentarios, reviews ni aprobaciones en el PR del compañero. La publicación en GitHub se delegará a una skill separada (a definir más adelante).


- **`MarchamoDigital.Base/ESTANDAR_API_MARCHAMO.md`** — convenciones obligatorias de servicio
- **`MarchamoDigital.Base/ESTANDAR_SEGURIDAD_MARCHAMO.md`** — 22 controles de seguridad
- **`MarchamoDigital.Base/.github/instructions/*.instructions.md`** — reglas por capa
- **`MarchamoDigital.Base/README.md`** — arquitectura de referencia, ficha técnica y relación CI/CD
- `bSide_INS_Marchamo_Development Specification_V1.0` (§14 checklist desarrollador, §15 checklist revisor)
- Experiencia consolidada de revisiones previas

> **Principio rector**: el revisor NO debe inventar estándares que no existan en el proyecto. Antes de marcar un patrón como "no estándar", verificar al menos 3 implementaciones existentes en el repositorio bajo revisión.

El output es una revisión estructurada por categorías con severidad, descripción y acción concreta para cada hallazgo.

---

## Repositorios cubiertos

Este skill aplica a TODOS los repos creados desde `MarchamoDigital.Base`:

| Repositorio | Tipo |
|-------------|------|
| MarchamoDigital.Validaciones | HTTP API |
| MarchamoDigital.Orquestador.Interno | HTTP API |
| MarchamoDigital.Orquestador.Externo | HTTP API |
| MarchamoDigital.Integrador.COSEVI | HTTP API |
| MarchamoDigital.Integrador.Recaudadores | HTTP API |
| MarchamoDigital.Integrador.SICSOA | API + Worker |
| MarchamoDigital.Negocio.EstadosTransaccionales | HTTP API |
| MarchamoDigital.Negocio.Inventario | HTTP API |
| MarchamoDigital.Integrador.ITV | HTTP API |
| MarchamoDigital.Sincronizador.Desacople | Worker |
| MarchamoDigital.Sincronizador.Control | Worker |
| MarchamoDigital.Auditoria.Correlacion | API + Worker |

---

## Cuándo usar este skill

- Cuando el usuario dice: _"revisa el PR #XXX"_, _"haz un code review"_, _"valida este PR"_
- Antes de aprobar un PR en cualquier repositorio `MarchamoDigital.*`
- Cuando se detectan dudas sobre conformidad con arquitectura o estándares

---

## Procedimiento paso a paso

### PASO 0 — Comprensión del PR (ANTES de revisar el código)

Antes de aplicar cualquier regla técnica, leer y resumir:

1. **Objetivo del PR:** qué problema resuelve, qué funcionalidad agrega o modifica.
2. **Alcance declarado:** qué capas / módulos / servicios se ven involucrados según la descripción.
3. **Decisiones técnicas justificadas:** si el autor justifica una decisión inusual (ej: skipear validación, patrón diferente), registrarla y evaluarla en contexto.
4. **Restricciones mencionadas:** limitaciones de tiempo, dependencias bloqueantes, trabajo parcial intencional.
5. **Riesgo funcional inicial:**
   - **Bajo** — cambio aislado, sin impacto en flujos críticos (pago, validación vehicular, auditoría)
   - **Medio** — afecta flujos de negocio pero con cobertura de tests o feature flag
   - **Alto** — afecta flujos críticos, sin tests suficientes, o cambio en contratos externos

> **Regla de oro:** toda observación debe evaluarse contra el objetivo declarado del PR. No generar hallazgos fuera del alcance real si no representan riesgo técnico relevante. Si el código cambiado no está relacionado con la descripción del PR, anotarlo como posible cambio fuera de alcance (🟡).

Incluir esta sección al inicio del reporte de salida (ver formato más adelante).

---

### PASO 1 — Obtener el diff del PR

1. Usar `mcp_io_github_git_get_file_contents` o las herramientas de GitHub MCP para obtener:
   - Metadata del PR (título, descripción, autor, reviewers, checks)
   - Los archivos modificados en el PR (lista + patches/diff)
   - El estado de los CI checks
2. Identificar qué capas están involucradas (Core, Domain, Application, Infrastructure, API, Test)
3. Revisar el título y descripción del PR para validar que cumpla Conventional Commits

### PASO 1.5 — Verificación profunda (PRs con merge history complejo)

Si el PR tiene >5 commits o múltiples merges desde `dev`:

1. **Verificar archivos perdidos en merges:** comparar los archivos del HEAD del branch contra commits clave del historial. Si un commit introdujo archivos de seguridad/auth y no sobreviven en HEAD, es regresión bloqueante.
2. **Grep end-to-end de auth:** ejecutar `grep -iE 'AddAuthentication|UseAuthentication|AddAuthorization|UseAuthorization|jwt|bearer|AllowAnonymous'` en `Program.cs` y en los endpoints. Si hay `AllowAnonymous()` pero NO hay middleware de auth registrado, escalar a 🔴.
3. **Verificar que archivos `.cs` referenciados en commits recientes existen en `git ls-tree HEAD`:** archivos de policies, scopes, auth handlers que pudieron perderse.

### PASO 2 — Aplicar el checklist de revisión

Aplicar las secciones del checklist en el orden indicado. Reportar cada hallazgo con:

```
🔴 BLOQUEANTE | 🟡 ADVERTENCIA | 🟢 SUGERENCIA
[CATEGORÍA] Descripción del hallazgo
Archivo: path/to/file.cs (línea aproximada)
Impacto: consecuencia técnica/operativa concreta
Acción: qué debe cambiar el autor
Complejidad de corrección: Baja | Media | Alta  ← solo para 🔴 BLOQUEANTE
Ejemplo: bloque de código cuando aplique  ← requerido para 🔴 BLOQUEANTE, opcional para 🟡
```

**IMPORTANTE — Evitar falsos positivos:**
- Antes de reportar un hallazgo sobre lifetime/DI, verificar cómo se registra en el Autofac module del repo.
- Antes de reportar un hallazgo sobre estilo de entidades (sealed, init, etc.), verificar al menos 3 entidades existentes en el repo.
- Antes de reportar un hallazgo sobre organización de archivos, verificar el patrón existente en la carpeta.
- Si no puedes verificar (no tienes acceso al código de producción), marcar como 🟢 SUGERENCIA con nota "verificar contra implementación existente".

### PASO 3 — Guardar el reporte como archivo .md

**OBLIGATORIO:** Al finalizar la revisión, guardar el reporte completo en un archivo Markdown con la ruta:

```
docs/90_borradores/pr-review-{número}-{scope}-{YYYY-MM-DD}.md
```

Ejemplo: `docs/90_borradores/pr-review-63-HU261529-2026-05-12.md`

El archivo debe incluir:
- Encabezado con metadatos (repo, rama, fecha, autor, revisores)
- Tabla de resumen ejecutivo con veredicto
- Tabla de archivos revisados
- Sección de puntos positivos
- Hallazgos detallados con código de ejemplo cuando aplique
- Tabla de estado de CI checks (la cantidad varía por repo)
- Condición explícita para aprobación
- Tabla de referencia normativa

---

## Checklist de revisión

### A. Arquitectura — Capas y Dependencias

Verificar que el PR respeta la dependencia unidireccional:
`00.Core` ← `01.Domain` ← `02.Application` ← `03.Infrastructure` ← `04.API` ← `05.Test`

**Reglas obligatorias:**

| Regla | Descripción | Severidad |
|-------|-------------|-----------|
| A1 | Los puertos (`I*` interfaces) deben vivir en `01.Domain/.../Interface.BD/` o `02.Application/Ports/` según el repo | 🔴 |
| A2 | Los adaptadores (implementaciones concretas) deben vivir en `03.Infrastructure` | 🔴 |
| A3 | El dominio (`01.Domain`) NO puede referenciar Application ni Infrastructure | 🔴 |
| A4 | `INS.Core.Base` debe consumirse ÚNICAMENTE como paquete NuGet (`PackageReference`) — nunca como `ProjectReference` ni copiar el 00.Core localmente | 🔴 |
| A5 | Los endpoints FastEndpoints deben vivir en `04.API/INS.API/FastEndPoint/{Contexto}/{Nombre}/V{N}/` | 🟡 |
| A6 | Los endpoints deben heredar de `EndpointBase<TRequest, TResponse>` (definida en `04.API/INS.API/Base/EndPoint/`) | 🔴 |
| A7 | Los UserCases DEBEN heredar de `UserCaseBase` e implementar `ICommandController<Cmd, Response<T>>` | 🔴 |
| A8 | Commands que **modifican datos** DEBEN implementar `IRequireTransaction` (activa `TransactionBehavior`) | 🔴 |
| A9 | Commands de **solo lectura** NO deben implementar `IRequireTransaction` | 🟡 |
| A10 | Repositorios DEBEN heredar de `DBRepositoryBase<TEntity>` e implementar la interfaz `I{Nombre}Repo<TEntity>` del dominio | 🔴 |

**Convenciones de nombrado (fuente: `.github/instructions/*.instructions.md`):**

| Artefacto | Convención | Ejemplo |
|-----------|-----------|---------|
| Comando | `{Accion}{Entidad}Cmd` | `EvaluarMotorReglasCmd` |
| UseCase | `{Accion}{Entidad}UserCase` | `EvaluarMotorReglasUserCase` |
| Validator | `{Entidad}RequestValidator` | `EvaluarMotorReglasRequestValidator` |
| Endpoint | `{Accion}{Entidad}Endpoint` | `AddDemoTaskEndpoint` |
| Puerto / Interfaz BD | `I{Entidad}Repo<TEntity>` | `IDemoTaskRepo<Entity>` |
| UnitOfWork | `I{Entidad}UnitOfWork` | `IDemoTaskUnitOfWork` |
| Repositorio impl | `{Entidad}Repo<TEntity>` | `DemoTaskRepo<TEntity>` |
| Autofac module (App) | `{Contexto}AppModule` | `ExamplesAppModule` |
| Autofac module (Infra) | `ProyectoRepositoryModule` | — |
| Autofac module (Validation) | `Validation{Contexto}Module` | `ValidationExamplesModule` |
| AutoMapper profile | `{Entidad}Profile` | `DemoTaskProfile` |
| Test class | `{Nombre}Tests` | `AddDemoTaskUserCaseTests` |
| Test method | `{Método}_Should{Resultado}_When{Condición}` | `Handle_ShouldReturn_Success_When_RequestIsValid` |

---

### B. Estándar API — Convenciones de Servicio (fuente: `ESTANDAR_API_MARCHAMO.md`)

| Regla | Descripción | Severidad |
|-------|-------------|-----------|
| B1 | **Sin `RoutePrefix` en `UseFastEndpoints`:** El prefijo `/api/<servicio>` lo inyecta el HTTPRoute del AGC. Si el servicio pone `c.Endpoints.RoutePrefix = "api"`, la URL pública queda con doble `/api/api/...` | 🔴 |
| B2 | **Endpoint `/health` obligatorio:** `app.MapHealthChecks("/health")` respondiendo `200 OK`. Lo consumen `HealthCheckPolicy` del AGC y `livenessProbe`/`readinessProbe` del pod | 🔴 |
| B3 | **Puerto 8080:** el contenedor SIEMPRE expone 8080. No usar otros puertos — rompe el `readinessProbe` y el routing del AGC | 🔴 |
| B4 | **OpenAPI/Swagger:** todo servicio expone Swagger en `/swagger` y JSON en `/swagger/v1/swagger.json` | 🟡 |
| B5 | **Logging estructurado (Serilog):** output JSON a `stdout`, con CorrelationId en cada log. No loguear secretos ni PII | 🔴 |
| B6 | **TLS mínimo 1.2:** el servicio no puede bajar la versión TLS. `UseHsts()` en entornos no-Dev | 🟡 |
| B7 | **Cultura fija `es-CR`:** para formato de fechas y números | 🟡 |
| B8 | **Consume `INS.Core.Base` via NuGet:** `PackageReference`, nunca `ProjectReference` ni copia local del 00.Core | 🔴 |
| B9 | **`Directory.Packages.props` con CPM:** si el repo usa Central Package Management, debe tener `<CentralPackageTransitivePinningEnabled>true</CentralPackageTransitivePinningEnabled>` | 🔴 |

---

### C. Código — Calidad y Patrones

**Reglas obligatorias:**

| Regla | Descripción | Severidad |
|-------|-------------|-----------|
| C1 | **No usar el operador null-forgiving (`!`)** para suprimir warnings legítimos en valores que pueden ser null realmente. Usar `?? fallback` o manejar el null explícitamente. | 🔴 |
| C2 | **Validación de cadena de conexión:** usar `string.IsNullOrWhiteSpace(connString)` con fail-fast (throw), NO `?? throw` si el placeholder es `""` (vacío) en lugar de `null` | 🔴 |
| C3 | **Secretos en `Program.cs`:** usar `AddCanonicalKeyVaultSecrets("CONNECTIONSTRINGS--XXX")`. Anti-patrones prohibidos: hardcodear CS en `appsettings.*.json`, k8s secrets via `secretKeyRef`, `DefaultAzureCredential` directo, invocar manualmente `AddDaprSecretStore` | 🔴 |
| C4 | **Doble serialización JSON:** si el tipo ya es `JsonObject` o `JsonElement`, NO volver a serializar/deserializar. Usar pattern matching para short-circuit | 🔴 |
| C5 | **Polly (si aplica):** `ResiliencePipeline` debe ser `static readonly` (thread-safe). NO crear instancias por request. 3 retries, exponential backoff + jitter | 🔴 |
| C6 | **CosmosDB — proyección mínima:** queries deben proyectar solo los campos necesarios. No usar `Find<T>` para traer documentos completos si solo se necesitan algunos campos | 🟡 |
| C7 | **Primary constructors:** usar primary constructor en UserCases y Repositories (C# 12) | 🟢 |
| C8 | **Namespaces:** deben iniciar con `INS.{Capa}.MarchamoDigital.{Modulo}` y reflejar la estructura de carpetas | 🟡 |
| C9 | **Entidades de dominio:** DEBEN heredar de `EntityBase`. NO incluir lógica de negocio. Propiedades de auditoría se heredan, no redeclararlas. Usar atributos SQL: `[Table]`, `[Column]`, `[Identity]`, `[NotSaved]`, `[NotUpdated]`, `[IsLogicalRemoved]` | 🔴 |
| C10 | **Respuesta uniforme `Response<T>`:** usar el envoltorio `Response<T>` con `Success()`, `FluentValidationError()`, `CriticalError()`. NO retornar excepciones directamente al cliente | 🔴 |
| C11 | **Flujo estándar de un UseCase:** (1) Validar request no nulo → `SendCriticalError()`, (2) FluentValidation → `SendFluentValidationError()`, (3) Mapear Request→Entity, (4) Repo via UoW, (5) Mapear Entity→Response, (6) Retornar `Result.Success<T>()` | 🔴 |
| C12 | **Validadores — registro DUAL obligatorio:** todo validador DEBE registrarse en (a) Autofac module Y (b) Microsoft DI en `Program.cs` con `builder.Services.AddScoped<IValidator<T>, TValidator>()`. Si solo se registra en Autofac, FastEndpoints no lo encuentra → error 500 | 🔴 |
| C13 | **No ocultar miembros base con `new`:** usar `protected new` para redeclarar un campo/propiedad que la clase base ya provee es code smell. En particular, `protected new ILogger Logger` oculta el `ILogger<TEndpoint>` tipado que FastEndpoints inyecta automáticamente — se pierde la categoría en logs. Preferir usar el Logger heredado o inyectar `ILogger<T>` con el tipo concreto. | 🟡 |
| C14 | **Eficiencia en lookups por clave única:** si se busca un documento por `correlationId` u otra clave única, NO usar un método genérico que aplica `Limit(N)` con N > 1 o multiplicadores artificiales. Crear un método dedicado `ObtenerPorIdAsync(id)` que haga `Find({_id: id}).Limit(1)` para minimizar consumo de RUs/DTUs. | 🟡 |
| C15 | **Validator error codes en el response:** si `WithErrorCode("RN-XX")` se usa en FluentValidation y los códigos son contractuales para el cliente, verificar que `EndpointResponseHelper` / `WriteErrorsAsync` los propague en el payload. Si solo emite `ErrorMessage` y descarta `ErrorCode`, los clientes no pueden reaccionar programáticamente. | 🟡 |

**Registro de dependencias (DI) — lifetimes canónicas:**

| Tipo de servicio | Lifetime | Fuente |
|-----------------|----------|--------|
| UserCases (Handlers) | `InstancePerLifetimeScope()` (Scoped) | Autofac module |
| Validators (FluentValidation) | `Scoped` (en ambos: Autofac + MS DI) | Dual |
| Repositorios SQL (con `ITransactionService`) | `InstancePerLifetimeScope()` | Autofac module |
| Repositorios CosmosDB (consumen `IMongoDatabase`) | `InstancePerLifetimeScope()` | Autofac module |
| `ITransactionService` | `InstancePerLifetimeScope()` | Autofac module |
| `IMongoDatabase` | `Singleton` | Autofac module |
| Polly `ResiliencePipeline` | `static readonly` field | Código |
| AutoMapper `IMapper` | `SingleInstance` | Autofac module |

> **CRÍTICO — Evitar falsos positivos en DI:** Antes de marcar un lifetime como incorrecto, verificar el `*Module.cs` del repo. TODOS los repos del proyecto registran repositorios como `InstancePerLifetimeScope()`. Solo `IMongoDatabase` y `IMapper` son Singleton.

---

### D. Tests — Cobertura y Calidad

**Reglas obligatorias:**

| Regla | Descripción | Severidad |
|-------|-------------|-----------|
| D1 | **Sin tests placeholder:** todo test debe tener al menos una aserción real. `Assert.True(true)` o métodos vacíos son BLOQUEANTES | 🔴 |
| D2 | **Tests unitarios sin I/O:** los tests unitarios NO deben conectar a bases de datos reales. Usar mocks de interfaces | 🔴 |
| D3 | **Patrón `CreateSut()` recomendado:** método privado que construye el SUT con todos sus mocks, retorna tupla `(sut, mock1, mock2, ...)` | 🟡 |
| D4 | **Tests de adaptadores:** instanciar el adaptador REAL, no mockear la interfaz que implementa. Probar comportamiento real con fakes de las dependencias inyectadas | 🔴 |
| D5 | **1 archivo = 1 clase de test principal:** nombre del archivo debe coincidir con la clase de test principal | 🟡 |
| D6 | **Cobertura mínima por UseCase:** (1) Request nulo → error/excepción, (2) Validación fallida → `FluentValidationError`, (3) Caso exitoso → `Success` con datos correctos | 🔴 |
| D7 | **Cobertura mínima gates:** QA ≥ 80%, UAT/PRD ≥ 90% | 🟡 |
| D8 | **Estructura AAA:** Arrange / Act / Assert claramente separados | 🟢 |
| D9 | **Framework:** xUnit con `[Fact]` (simples) y `[Theory]+[InlineData]` (parametrizados). Moq para mocking. FluentAssertions o `Assert.*` para aserciones — no mezclar estilos en el mismo archivo | 🟡 |
| D10 | **Nombrado de métodos de test:** `{Método}_Should{Resultado}_When{Condición}` (ej: `Handle_ShouldThrow_When_RequestIsNull`) | 🟡 |

**Traits de xUnit permitidos (repos con CosmosDB):**

```csharp
[Trait("Category", "Integration")]   // Tests de integración sin Cosmos (CI canónico)
[Trait("Category", "LiveCosmos")]     // Tests que requieren Cosmos real (excluidos del CI)
[Trait("Feature", "NombreFeature")]  // Subtipo de LiveCosmos para filtro fino
```

---

### E. Infraestructura — SQL MI (Dapper) y CosmosDB (MongoDB API)

#### E.1 SQL MI (repos que usan Dapper)

| Regla | Descripción | Severidad |
|-------|-------------|-----------|
| E1 | Repos heredan de `DBRepositoryBase<TEntity>` con `ITransactionService` | 🔴 |
| E2 | Unit of Work implementa `IUnitOfWorkBase` y expone repos + `ITransactionService` | 🔴 |
| E3 | Stored Procedures anotados con `[StoredProcedure]` y `[StoredProcedureParameter]` | 🟡 |
| E4 | **Sin SQL Injection:** TODAS las queries deben usar parámetros. Nunca concatenar strings en SQL | 🔴 |
| E5 | Connection string registrada desde `configuration.GetConnectionString("NombreConexion")` | 🟡 |

#### E.2 CosmosDB MongoDB API (repos que usan MongoDB.Driver)

| Regla | Descripción | Severidad |
|-------|-------------|-----------|
| E6 | Registrar repository Cosmos como `InstancePerLifetimeScope()` (estándar del proyecto). Solo `IMongoDatabase` y `IMapper` son `Singleton`. Verificar siempre el `*Module.cs` de Autofac antes de marcar un lifetime como incorrecto. | 🔴 |
| E7 | Fail-fast si `string.IsNullOrWhiteSpace(connectionString)` en constructor del repository | 🔴 |
| E8 | `static readonly ResiliencePipeline` para reintentos de Mongo | 🔴 |
| E9 | Proyectar solo campos necesarios en queries | 🟡 |
| E10 | Manejar excepciones transientes en Polly: **MongoDB.Driver v3.x** → `MongoConnectionException` + `MongoExecutionTimeoutException`. **NO usar** `MongoTimeoutException` (eliminado en v3.x) | 🟡 |

---

### F. Seguridad — 22 Controles (fuente: `ESTANDAR_SEGURIDAD_MARCHAMO.md`)

| Regla | Descripción | Severidad |
|-------|-------------|-----------|
| F1 | **Sin secretos en código:** ni cadenas de conexión, passwords, API keys, tokens. Solo placeholder `""` en `appsettings.json`. Usar `AddCanonicalKeyVaultSecrets()` | 🔴 |
| F2 | **Sin credenciales en logs:** no loguear connection strings, tokens, PII ni datos sensibles | 🔴 |
| F3 | **Validación de entrada:** todos los endpoints DEBEN tener `IValidator<T>` con FluentValidation. Nunca confiar en input sin validar | 🔴 |
| F4 | **Sin SQL Injection:** queries parametrizadas siempre (Dapper usa `@param`) | 🔴 |
| F5 | **Actions pinned by SHA:** todo workflow YAML debe referenciar Actions por SHA, no por tag/branch | 🔴 |
| F6 | **Dockerfile non-root:** la imagen final debe correr como usuario no-root y exponer solo puerto 8080 | 🔴 |
| F7 | **Sin dependencias con CVE Critical/High:** la Dependency Review del CI bloquea PRs que agreguen deps vulnerables | 🔴 |
| F8 | **Sin licencias prohibidas:** GPL, AGPL y similares no se permiten en código comercial (Licensee check) | 🔴 |
| F9 | **`SECURITY.md` presente:** todo repo debe tener un archivo SECURITY.md en la raíz | 🟡 |
| F10 | **Event Hub solo con Managed Identity:** SAS deshabilitado (`disableLocalAuth=true`). NO usar Connection String para Event Hub | 🔴 |
| F11 | **Headers de seguridad:** `X-Frame-Options: DENY`, CSP restrictivo (ya incluido en template) | 🟡 |
| F12 | **Ningún PR se mergea con findings `Critical` o `High`** sin override explícito del Security Champion + 1 code owner. Override máximo 14 días | 🔴 |
| F13 | **Auth middleware obligatorio en producción:** `AddAuthentication`/`AddAuthorization`/`UseAuthentication`/`UseAuthorization` DEBEN estar registrados en `Program.cs` para cualquier endpoint que exponga datos de negocio. Si `AllowAnonymous()` se usa sin auth middleware registrado, el PR es BLOQUEANTE salvo que se documente formalmente (en el body del PR + work item con prioridad bloqueante) que auth vendrá en PR posterior y exista un gate que impida deploy a staging/prod sin auth. Verificar con `grep -iE 'auth|jwt|bearer|scope'` en todo el diff y `Program.cs`. | 🔴 |
| F14 | **Commits de seguridad perdidos en merges:** en PRs con múltiples merges desde `dev`, verificar que archivos de seguridad introducidos en commits anteriores (auth handlers, policies, scopes) sigan presentes en el HEAD del branch. Usar `git ls-tree HEAD -- <path>` si hay duda. Si se detecta regresión, es BLOQUEANTE. | 🔴 |

---

### G. CI/CD y PR Metadata

| Regla | Descripción | Severidad |
|-------|-------------|-----------|
| G1 | **Título del PR:** debe seguir Conventional Commits: `feat|fix|refactor|docs|test|chore(scope): descripción` | 🟡 |
| G2 | **Tamaño del PR:** objetivo ≤ 600 líneas de diff. Si supera 800 líneas, emitir 🟡 ADVERTENCIA (NO bloqueante) recordando que lo ideal es generar PRs por cada avance significativo, sin esperar al final del trabajo. No contar archivos auto-generados, migraciones triviales ni snapshots de prueba en el conteo | 🟡 |
| G3 | **Branch name:** `feature/{ticket-id}-{descripcion-corta}` | 🟢 |
| G4 | **CI checks deben pasar antes de merge** (la cantidad varía por repo: Semgrep, DevSkim, Gitleaks, TruffleHog, Hadolint, Dockle, NuGet Audit, Dependency Review, License Check, Integration Tests, Build, PR Validate, PR Size Check, CodeQL, etc.) | 🔴 |
| G5 | **No usar `--no-verify`** ni forzar merge con checks fallando | 🔴 |
| G6 | **Workflows CI/CD:** deben usar reusable workflows de `Marchamo.Digital.Pipelines` (no CD hecho a mano) | 🔴 |
| G7 | **`deploy-manifest.yaml` presente:** define servicios desplegables, recursos, scaling y Helm chart | 🟡 |
| G8 | **`ci.yml` caller:** debe declarar `permissions: { packages: read, actions: read }` | 🟡 |

---

### H. Correlación y Auditoría

| Regla | Descripción | Severidad |
|-------|-------------|-----------|
| H1 | **`AddMarchamoCorrelation()`** registrado en `Program.cs` si el servicio usa Event Hub para auditoría | 🔴 |
| H2 | **`UseMarchamoCorrelation()`** en el pipeline de middleware (extrae `X-Correlation-Id` del header) | 🔴 |
| H3 | **Propagación a Service Bus:** si el servicio envía mensajes, propagar CorrelationId con `ServiceBusCorrelationHandler.PropagateToMessage()` | 🟡 |
| H4 | **Variables de configuración:** `MarchamoCorrelation__EventHubNamespace`, `MarchamoCorrelation__AuditEventHubName`, `MarchamoCorrelation__AzureTenantId` | 🟡 |

---

### I. DDD y SOLID — Diseño orientado al dominio y principios de diseño

Revisar los cambios del PR contra principios de **DDD** (capa de dominio rica y aislada) y **SOLID**. Aplicar con criterio: el objetivo es señalar deuda de diseño, no forzar refactors masivos.

#### I.1 DDD — Domain-Driven Design

| Regla | Descripción | Severidad |
|-------|-------------|-----------|
| I1 | **Dominio puro:** `01.Domain` NO debe depender de `02.Application`, `03.Infrastructure`, `04.API`, ni de librerías de infraestructura (EF, Dapper, Mongo, HttpClient, ASP.NET, etc.). Solo `00.Core` y librerías de soporte (`INS.Core.Base`). | 🔴 |
| I2 | **Lógica de negocio en el dominio:** invariantes, reglas y validaciones de negocio viven en entidades / value objects / domain services. Si un UserCase concentra cálculos, decisiones y reglas que pertenecen a la entidad, marcar como advertencia. | 🟡 |
| I3 | **Entidades anémicas:** una entidad que solo tiene getters/setters sin comportamiento NO es defecto automático en este proyecto (BSON/Dapper requiere setters), pero si la regla de negocio asociada vive dispersa en varios UserCases, sugerir extraer a método del dominio o domain service. | 🟢 |
| I4 | **Value Objects para conceptos sin identidad:** Placa, NumeroMarchamo, Monto, Periodo, etc. deben modelarse como VO inmutables con validación en el constructor cuando aplique. Pasar strings/decimales primitivos por todas las capas es _primitive obsession_. | 🟡 |
| I5 | **Lenguaje ubicuo:** nombres de clases, métodos y propiedades deben usar el vocabulario del negocio (ver `Analisis Épicas/shared/glossary.yaml`). Nombres genéricos tipo `Manager`, `Helper`, `Processor`, `Util` sin contexto del dominio son señal de fuga del modelo. | 🟡 |
| I6 | **Límites de contexto (Bounded Context):** un repo no debe importar entidades de dominio de otro repo `MarchamoDigital.*` directamente. La integración entre contextos es vía contratos/DTOs por HTTP o Service Bus. | 🔴 |
| I7 | **Agregados con raíz clara:** modificaciones a entidades hijas pasan por la raíz del agregado. Evitar repositorios separados para entidades que son parte de un agregado mayor. | 🟡 |
| I8 | **Eventos de dominio (cuando aplique):** cambios de estado relevantes para auditoría/integración deben emitir eventos, no asumir efectos colaterales implícitos. | 🟢 |

#### I.2 SOLID

| Regla | Descripción | Severidad |
|-------|-------------|-----------|
| S1 | **SRP — Single Responsibility:** una clase / UserCase / método con múltiples responsabilidades no relacionadas. Si un método tiene >40 líneas efectivas o mezcla validación + orquestación + mapeo + persistencia + logging detallado, sugerir extracción. | 🟡 |
| S2 | **Método muy grande ("god method"):** método con muchos pasos secuenciales (>7 pasos lógicos, anidamiento >3 niveles, o ciclomática estimada >10) → 🟡 ADVERTENCIA con sugerencia de extraer métodos privados o domain services. NO bloqueante. | 🟡 |
| S3 | **Clase muy grande ("god class"):** clase >400 líneas o >15 métodos públicos → 🟡 ADVERTENCIA y sugerir separación por responsabilidades. | 🟡 |
| S4 | **OCP — Open/Closed:** preferir extensión por composición / estrategia / polimorfismo sobre `switch`/`if-else` largos que crecen con cada nuevo caso del dominio. | 🟢 |
| S5 | **LSP — Liskov Substitution:** una subclase no debe lanzar `NotImplementedException` ni cambiar el contrato (pre/post condiciones) de la base. | 🔴 |
| S6 | **ISP — Interface Segregation:** evitar interfaces "fat" con muchos métodos no relacionados que obligan a implementaciones a dejar métodos vacíos / `throw`. Preferir interfaces pequeñas y cohesivas. | 🟡 |
| S7 | **DIP — Dependency Inversion:** UserCases y servicios de aplicación dependen de **abstracciones** (`IXxxRepo`, `IXxxService`) declaradas en el dominio/aplicación, NUNCA de implementaciones concretas de infraestructura. `new HttpClient()`, `new SqlConnection()`, `DateTime.Now` dentro de la lógica son banderas. | 🔴 |
| S8 | **Inyección por constructor:** todas las dependencias se reciben por constructor (primary constructor de C# 12 preferido). Prohibido service locator (`IServiceProvider.GetService` dentro de la lógica). | 🔴 |
| S9 | **Acoplamiento temporal:** secuencias de llamadas que deben hacerse en un orden estricto y no validado son señal de mal diseño. Sugerir encapsular el flujo. | 🟢 |
| S10 | **Duplicación de lógica (DRY adjunto):** si el PR introduce 2+ copias de la misma regla de negocio en distintos UserCases, marcar advertencia para extraer al dominio. | 🟡 |

> **Criterio práctico:** los hallazgos de I/S son normalmente 🟡 o 🟢. Solo escalar a 🔴 cuando se viola un límite arquitectónico duro del proyecto (dependencia inválida entre capas, dependencia concreta en lugar de abstracción, ruptura de bounded context).

---

## Formato de reporte de salida

```markdown
## Revisión PR #XXX — {Título del PR}

### 0. Comprensión del PR

**Objetivo:** (qué problema resuelve / funcionalidad agrega)

**Alcance declarado:** (capas/módulos afectados según la descripción del autor)

**Decisiones técnicas justificadas en la descripción:** (listar o indicar "ninguna declarada")

**Riesgo funcional:** Bajo | Medio | Alto — (justificación breve)

**Cambios fuera de alcance detectados:** (archivos modificados que no tienen relación aparente con el objetivo) o "Ninguno detectado"

---

### Resumen ejecutivo
- Repositorio: MarchamoDigital.{Modulo}
- Archivos modificados: N
- Capas afectadas: [Core | Domain | Application | Infrastructure | API | Test]
- Hallazgos: X bloqueantes, Y advertencias, Z sugerencias
- Veredicto: ✅ APROBADO | ⚠️ APROBADO CON CONDICIONES | 🚫 REQUIERE CAMBIOS

---

### Puntos positivos
(listar lo que el PR hace bien — no solo criticar)

### Hallazgos detallados

#### 🔴 BLOQUEANTES (deben corregirse antes del merge)

1. **[C4] Doble serialización JSON**
   Archivo: `02.Application/.../EvaluarMotorReglasUserCase.cs` línea 45
   Descripción: `return JsonSerializer.Serialize(datos)` cuando `datos` ya es `JsonObject`
   Impacto técnico: serialización doble introduce latencia y posible corrupción de estructura JSON
   Acción: Agregar pattern matching para short-circuit
   Complejidad de corrección: Baja | Media | Alta
   ```csharp
   // ❌ Actual
   return JsonSerializer.Serialize(datos);
   // ✅ Propuesta
   return datos is JsonObject jo ? jo.ToJsonString() : JsonSerializer.Serialize(datos);
   ```

#### 🟡 ADVERTENCIAS (corregir en este PR o en chore inmediato)

1. **[D5] Filename no coincide con clase principal**
   Archivo: `05.Test/.../EtiquetaRfidServicioTests.cs`
   Descripción: El archivo contiene `EtiquetaRfidCosmosServiceTests` como clase principal
   Acción: Renombrar archivo en chore PR

#### 🟢 SUGERENCIAS (opcionales, mejoran calidad)

1. **[D8] Estructura AAA**
   Descripción: Agregar comentarios `// Arrange / Act / Assert` para legibilidad

---

### Tabla consolidada de hallazgos

| ID | Categoría | Hallazgo | Severidad | Impacto | Acción recomendada | ¿Bloquea merge? |
|----|-----------|----------|-----------|---------|-------------------|-----------------|
| H01 | Arquitectura | (descripción corta) | 🔴 Crítico | (impacto) | (acción) | Sí |
| H02 | Testing | (descripción corta) | 🟡 Medio | (impacto) | (acción) | No |
| H03 | Clean Code | (descripción corta) | 🟢 Bajo | (impacto) | (acción) | No |

Las categorías posibles: Arquitectura · Performance · Seguridad · Observabilidad · Testing · Mantenibilidad · Resiliencia · Diseño · Concurrencia · Integridad de Datos · Clean Code · DDD · SOLID

---

### Aspectos diferibles (no bloquean merge)

Ítems que no representan riesgo inmediato pero deben trackearse:

| Ítem | Prioridad | Impacto estimado | Riesgo de postergarlo |
|------|-----------|------------------|-----------------------|
| (descripción) | Alta/Media/Baja | (impacto) | (consecuencia) |

Si no hay ítems diferibles: _No se identifican ítems diferibles en este PR._

---

### Evaluación arquitectónica general

- **Nivel de madurez técnica del cambio:** Básico | Intermedio | Avanzado
- **Coherencia con el objetivo del PR:** Alineado | Parcialmente alineado | No alineado
- **Riesgo general del merge:** Bajo | Medio | Alto
- **Calidad general de implementación:** (párrafo breve, perspectiva senior)

---

### CI Checks
| Check | Estado |
|-------|--------|
| Build | ✅ |
| ... | ... |

---

### 🚫 Blockers Before Merge

(Si existen bloqueantes:)
- [ ] **[F13] Auth middleware ausente** — Riesgo: endpoint expuesto públicamente en prod. Impacto: cualquier cliente sin credenciales puede invocar el API. Recomendación: agregar `UseAuthentication()` + `UseAuthorization()` en `Program.cs`.
- [ ] **[C3] Cadena de conexión hardcodeada** — Riesgo: credencial expuesta en repositorio. Impacto: breach de seguridad. Recomendación: reemplazar con `AddCanonicalKeyVaultSecrets("CONNECTIONSTRINGS--XXX")`.

(Si no existen bloqueantes:)
✅ No se identifican bloqueantes críticos para merge.

---

### Recomendación final

**Decisión:** Approve | Approve with Minor Improvements | Request Changes | Reject Temporarily

**Justificación:** (párrafo técnico que sustenta la decisión considerando el objetivo del PR, los hallazgos encontrados y el riesgo operativo)

### Condición de aprobación
(qué debe pasar exactamente para que el PR se apruebe — solo cuando hay bloqueantes o condiciones)

### Referencia normativa
| Fuente | Ubicación |
|--------|-----------|
| ESTANDAR_API_MARCHAMO.md | MarchamoDigital.Base (raíz) |
| ESTANDAR_SEGURIDAD_MARCHAMO.md | MarchamoDigital.Base (raíz) |
| .github/instructions/ | MarchamoDigital.Base |
| Development Specification V1.0 | Spec externa |
```

---

## Patrones de código de referencia

### Endpoint correcto (04.API)

```csharp
public class AddDemoTaskEndpoint(IMediator mediator)
    : EndpointBase<DemoTaskRequest, Response<DemoTaskResponse>>(mediator, RouteFastEndpoint.AddDemoTask)
{
    public override void Configure()
    {
        Post(RouteFastEndpoint.AddDemoTask);
        Version(0);
        Tags(Constants.Versions.VERSION_0);
        Group<ExampleGrup>();
        Description(x => x.WithName("Add DemoTask").WithSummary("Crea una tarea demo"));
        AllowAnonymous();
    }

    public override async Task HandleAsync(DemoTaskRequest request, CancellationToken ct)
        => await ValidateResult(
            await SendMediatorCommand<AddDemoTaskCmd, DemoTaskResponse>(
                req => new AddDemoTaskCmd(req), request, ct), ct);
}
```

### UserCase correcto (02.Application)

```csharp
public class AddDemoTaskUserCase(
    IMapper mapper,
    ILogger<AddDemoTaskUserCase> logger,
    IDemoTaskUnitOfWork unitOfWork,
    IValidator<DemoTaskRequest> validator)
    : UserCaseBase(logger, nameof(AddDemoTaskUserCase)),
      ICommandController<AddDemoTaskCmd, DemoTaskResponse>
{
    public async Task<Response<DemoTaskResponse>> Handle(AddDemoTaskCmd cmd, CancellationToken ct)
    {
        if (cmd?.Request is null) return SendCriticalError("Request nulo");

        var validation = await validator.ValidateAsync(cmd.Request, ct);
        if (!validation.IsValid) return SendFluentValidationError(validation);

        var entity = mapper.Map<DemoTaskEntity>(cmd.Request);
        await unitOfWork.DemoTaskRepo.Add(entity);

        var response = mapper.Map<DemoTaskResponse>(entity);
        return Result.Success(response);
    }
}
```

### Command correcto (02.Application)

```csharp
// ✅ Modifica datos → implementa IRequireTransaction
public record AddDemoTaskCmd(DemoTaskRequest Request) : ICommand<Response<DemoTaskResponse>>, IRequireTransaction;

// ✅ Solo lectura → NO implementa IRequireTransaction
public record GetDemoTaskByIdCmd(int Id) : ICommand<Response<DemoTaskResponse>>;
```

### Polly correcto (03.Infrastructure)

```csharp
private static readonly ResiliencePipeline _pipeline = new ResiliencePipelineBuilder()
    .AddRetry(new RetryStrategyOptions
    {
        MaxRetryAttempts = 3,
        BackoffType = DelayBackoffType.Exponential,
        UseJitter = true,
        ShouldHandle = new PredicateBuilder().Handle<MongoConnectionException>()
                                             .Handle<MongoExecutionTimeoutException>()
    })
    .Build();
```

### Validador con registro dual (02.Application + Program.cs)

```csharp
// En 02.Application/Validation/
public class DemoTaskRequestValidator : AbstractValidator<DemoTaskRequest>
{
    public DemoTaskRequestValidator()
    {
        RuleFor(x => x.TaskName).NotEmpty().MaximumLength(255);
        RuleFor(x => x.TaskDate).NotEmpty();
    }
}

// En Program.cs — OBLIGATORIO para que FastEndpoints lo encuentre:
builder.Services.AddScoped<IValidator<DemoTaskRequest>, DemoTaskRequestValidator>();
```

### Repositorio correcto (03.Infrastructure)

```csharp
public class DemoTaskRepo<TEntity>(
    ITransactionService transactionService,
    ILogger<DBRepositoryBase<TEntity>> logger)
    : DBRepositoryBase<TEntity>(transactionService, logger),
      IDemoTaskRepo<TEntity>
    where TEntity : EntityBase, new()
{
    // Métodos custom...
}
```

### Test correcto con CreateSut() (05.Test)

```csharp
public class AddDemoTaskUserCaseTests
{
    private (AddDemoTaskUserCase sut, Mock<IMapper> mapper, Mock<IDemoTaskUnitOfWork> uow) CreateSut()
    {
        var mapper = new Mock<IMapper>(MockBehavior.Strict);
        var logger = new Mock<ILogger<AddDemoTaskUserCase>>();
        var uow = new Mock<IDemoTaskUnitOfWork>(MockBehavior.Strict);
        var validator = new Mock<IValidator<DemoTaskRequest>>();
        validator.Setup(v => v.ValidateAsync(It.IsAny<DemoTaskRequest>(), default))
            .ReturnsAsync(new ValidationResult());

        var sut = new AddDemoTaskUserCase(mapper.Object, logger.Object, uow.Object, validator.Object);
        return (sut, mapper, uow);
    }

    [Fact]
    public async Task Handle_ShouldReturn_Success_When_RequestIsValid()
    {
        // Arrange
        var (sut, mapper, uow) = CreateSut();
        var request = new DemoTaskRequest { TaskName = "Test", TaskDate = DateTime.Now };
        // Setup mocks...

        // Act
        var result = await sut.Handle(new AddDemoTaskCmd(request), CancellationToken.None);

        // Assert
        Assert.True(result.IsSuccess);
        Assert.Equal(ResponseState.Ok, result.State);
    }

    [Fact]
    public async Task Handle_ShouldReturn_CriticalError_When_RequestIsNull()
    {
        // Arrange
        var (sut, _, _) = CreateSut();

        // Act
        var result = await sut.Handle(new AddDemoTaskCmd(null!), CancellationToken.None);

        // Assert
        Assert.Equal(ResponseState.CriticalError, result.State);
    }
}
```

### Fail-fast connection string correcto (03.Infrastructure)

```csharp
// ✅ CORRECTO: IsNullOrWhiteSpace porque el placeholder es "" (vacío, no null)
if (string.IsNullOrWhiteSpace(connectionString))
    throw new InvalidOperationException("La cadena de conexión no está configurada.");

// ❌ INCORRECTO: ?? throw solo captura null, no ""
var cs = connectionString ?? throw new InvalidOperationException("...");
```

### Program.cs canónico — secretos con Key Vault

```csharp
using INS.CORE.Base.Infrastructure.Extensions;

var builder = WebApplication.CreateBuilder(args);

// Secretos via Dapr + Key Vault (OBLIGATORIO)
builder.AddCanonicalKeyVaultSecrets("CONNECTIONSTRINGS--COSMOSDBMONGODB");

// Correlación + auditoría
builder.Services.AddMarchamoCorrelation(builder.Configuration);

var app = builder.Build();
app.UseMarchamoCorrelation();
// NO: app.UseFastEndpoints(c => { c.Endpoints.RoutePrefix = "api"; }); ← PROHIBIDO
app.UseFastEndpoints(c =>
{
    c.Versioning.Prefix = "v";
    c.Versioning.PrependToRoute = true;
});
app.MapHealthChecks("/health"); // OBLIGATORIO
app.Run();
```

---

## Guardrails anti-falso-positivo (lecciones aprendidas)

> Estas reglas previenen findings inválidos que desprestigian la revisión:

1. **Lifetime DI — verificar antes de criticar:** TODOS los repos del proyecto usan `InstancePerLifetimeScope()`. Solo `IMongoDatabase` e `IMapper` son Singleton. NO asumir que repos deben ser Singleton por thread-safety teórica sin verificar el módulo Autofac existente.

2. **Entidades sealed/init — verificar patrón existente:** El template NO exige `sealed` ni `init`-only en entidades. El BSON/Dapper mapper necesita setters. Verificar al menos 3 entidades del repo antes de sugerir sealed/init.

3. **1 clase = 1 archivo — verificar patrón existente:** Algunas constantes coexisten en un mismo archivo (ej: `DomainConstants.cs` con múltiples clases). Verificar si es patrón establecido antes de pedir separación.

4. **NO inventar estándares teóricos:** Si algo no está explícitamente en `ESTANDAR_API_MARCHAMO.md`, `ESTANDAR_SEGURIDAD_MARCHAMO.md` o `.github/instructions/`, NO marcarlo como BLOQUEANTE. Máximo 🟢 SUGERENCIA con justificación de best practice.

5. **AllowAnonymous ≠ "advertencia menor":** si NO existe middleware de auth registrado (`AddAuthentication`/`UseAuthorization` en `Program.cs`), `AllowAnonymous()` no es simplemente "deuda técnica" — es un endpoint público real sin ninguna capa de protección. Esto es 🔴 BLOQUEANTE (ver F13). Solo baja a 🟡 si el middleware existe y `AllowAnonymous` es explícito/temporal con gate documentado.

6. **Verificar multiplicadores artificiales en queries:** patrones como `Limit(filtro.ElementosPorPagina * 10)` que desperdician RUs cuando el filtro busca por clave única. Reportar como 🟡 con sugerencia de método dedicado.

7. **`EndpointBase<TRequest, object>` — serialización vacía:** si una base class usa `object` como `TResponse`, ojo que responses vacíos serializan como `{}` en lugar de ser omitidos. Verificar si afecta el contrato HTTP.

---

## Fuentes normativas del proyecto

| Documento | Ubicación | Contenido relevante |
|-----------|-----------|---------------------|
| ESTANDAR_API_MARCHAMO.md | MarchamoDigital.Base (raíz) | Convenciones obligatorias de servicio: FastEndpoints, /health, puerto 8080, OpenAPI, NuGet, CPM |
| ESTANDAR_SEGURIDAD_MARCHAMO.md | MarchamoDigital.Base (raíz) | 22 controles de seguridad, gates por severidad, responsabilidades |
| .github/instructions/ | MarchamoDigital.Base | Reglas por capa: endpoints, entities, repos, tests, usecases, validators |
| README.md | MarchamoDigital.Base | Arquitectura, ficha técnica, correlación, relación CI/CD, NuGet canónico |
| .editorconfig | MarchamoDigital.Base | Formateo: UTF-8, spaces-4, CA suppressions justificadas |
| Development Specification V1.0 | Spec externa | §14 checklist dev, §15 checklist revisor |
| documentation/BLUEPRINT.MD | Cada repo | Guía de implementación de features del módulo |
| deploy-manifest.yaml | Cada repo | Manifiesto declarativo de despliegue |
