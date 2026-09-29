---
name: create-module
description: "Workflow completo para crear un módulo nuevo en proyectos del ecosistema MarchamoDigital. Usar cuando se necesite crear una funcionalidad completa que atraviese todas las capas (dominio, aplicación, infraestructura, API, tests). Guía paso a paso usando DemoTask de MarchamoDigital.Base como template canónico."
---
# Crear Módulo Nuevo — Workflow Completo

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
Guiar paso a paso la creación de un módulo nuevo que atraviesa todas las capas del proyecto (dominio, aplicación, infraestructura, API, tests).

**Problema que resuelve:**
Crear un CRUD completo en Clean Architecture requiere conocer el orden correcto, las convenciones de nomenclatura, las instrucciones por capa y el template canónico `DemoTask`.

**Responsabilidades que NO tiene este Skill:**
- No agrega una operación a un módulo existente (solo módulos nuevos).
- No realiza operaciones Git.

---

## Cuándo usar
- Al agregar una nueva funcionalidad/recurso al proyecto.
- Para crear un CRUD completo que atraviese todas las capas.
- Cuando se necesite una guía paso a paso con archivos de referencia concretos.

## Prerrequisitos
- Nombre del módulo/recurso (ej: `Product`)
- Campos con tipos (ej: `Name: string`, `Price: decimal`)
- Tabla SQL destino si aplica
- Operaciones requeridas (Add, GetById, Update, Delete, queries custom)

## Procedimiento (seguir en orden)

### Paso 1: Entidad de dominio
Leer `.github/instructions/csharp-entities.instructions.md`.
- Heredar de `EntityBase`
- Anotar con `[Table(...)]` y `[Column(...)]`
- PK con `[Identity]` y `[NotSaved]`

### Paso 2: Request DTO
Leer `.github/instructions/csharp-usecases.instructions.md`.
- Heredar de `RequestBase`
- Solo campos editables por el usuario

### Paso 3: Response DTO
- Heredar de `ResponseBase`

### Paso 4: Interfaz de repositorio
- `I{Nombre}Repo<TEntity>` heredando de `IDBRepositoryBase<{Nombre}Entity>`
- `I{Nombre}UnitOfWork` heredando de `IUnitOfWorkBase`

### Paso 5: Commands (uno por operación)
Leer `.github/instructions/csharp-usecases.instructions.md`.
- `{Acción}{Nombre}Cmd` como `record`
- Implementar `ICommand<Response<{Nombre}Response>>`
- Si modifica datos: implementar `IRequireTransaction`

### Paso 6: Use Cases (uno por operación)
- Heredar de `UserCaseBase`
- Implementar `ICommandController<TCmd, TResponse>`
- Flujo: validar nulo → FluentValidation → mapear → repo → mapear → `Response.Success()`

### Paso 7: Validadores
Leer `.github/instructions/csharp-validators.instructions.md`.
- `AbstractValidator<TRequest>`
- Registrar en Autofac Y en `Program.cs` (Microsoft DI) ← CRÍTICO

### Paso 8: Repositorio
Leer `.github/instructions/csharp-repos.instructions.md`.
- `{Nombre}Repo<TEntity>` heredando de `DBRepositoryBase<TEntity>`
- `{Nombre}UnitOfWork` implementando `I{Nombre}UnitOfWork`
- `{Nombre}Profile` para AutoMapper
- Actualizar `ProyectoRepositoryModule` en Autofac

### Paso 9: Endpoint
Leer `.github/instructions/csharp-endpoints.instructions.md`.
- Un `{Acción}{Nombre}Endpoint` por operación en `V0/`
- Agregar constantes en `RouteFastEndpoint.cs`

### Paso 10: Tests
Leer `.github/instructions/csharp-tests.instructions.md`.
- Un archivo de test por use case
- Patrón `CreateSut()` + AAA
- Cobertura: null, validación fallida, caso exitoso

### Paso 11: Verificar
```bash
dotnet build *.sln && dotnet test *.sln
```

## Referencia canónica
El módulo `DemoTask` en `MarchamoDigital.Base` es la plantilla exacta para todos los pasos.

---

## Skill Collaboration

Cuando la solicitud requiera capacidades fuera del dominio de este Skill, coordinar con el Skill correspondiente siguiendo `framework/DECISION-TREE.md`.

| Necesidad | Skill |
|---|---|
| Análisis técnico y diseño del módulo | `arquitecto-de-software` |
| Revisar el PR del módulo creado | `pr-review-marchamo` |
| Probar el endpoint en local | `localhost-api-testing` |

---

## Completion Checklist

Antes de considerar terminada cualquier tarea:

- □ Se respetó el Framework (Principles, Workflow, GIT-Policy, Conventions).
- □ Se siguió el Workflow.
- □ Se utilizó el Skill correcto.
- □ Se mantuvieron cambios mínimos.
- □ Se mantuvo consistencia con el proyecto.
- □ Se espera aprobación del desarrollador antes de cualquier operación Git.
