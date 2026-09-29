---
name: localhost-api-testing
description: 'Levanta un microservicio MarchamoDigital en localhost conectado a Cosmos DB MongoDB real y ejecuta pruebas manuales contra endpoints. USA CUANDO: probar endpoint en local, levantar API local, test manual localhost, POST local contra Cosmos, verificar endpoint, prueba E2E local, correr API con MongoDB real, debug endpoint local.'
argument-hint: 'Especifica el microservicio a levantar (ej: estados-transaccionales, validaciones, integrador-recaudadores) y qué endpoint probar.'
---

# Localhost API Testing — MarchamoDigital

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
Levantar cualquier microservicio del ecosistema localmente conectado a Cosmos DB MongoDB real y ejecutar requests manuales para validación E2E.

**Problema que resuelve:**
Probar un endpoint localmente requiere conocer cómo configurar la variable de conexión, qué puerto usa cada servicio y cómo verificar que arranó correctamente.

**Responsabilidades que NO tiene este Skill:**
- No ejecuta pruebas automáticas (eso es `dotnet test`).
- No gestiona el ambiente cloud.
- No realiza operaciones Git.

---

## Propósito
Levantar cualquier microservicio del ecosistema localmente conectado a **Cosmos DB MongoDB real** (ambiente dev) y ejecutar requests manuales para validación E2E.

## Prerrequisitos

1. **VPN activa** — La conexión a Cosmos DB requiere VPN corporativa.
2. **Variable de entorno** `CONNECTIONSTRINGS__CosmosDbMongoDB` configurada con el connection string de Cosmos DB.
3. **.NET 10 SDK** instalado.

## Procedimiento paso a paso

### 1. Verificar variable de entorno

```bash
# Verificar que existe (NO imprimir el valor — contiene credenciales)
if [ -n "$CONNECTIONSTRINGS__CosmosDbMongoDB" ]; then echo "OK: presente"; else echo "ERROR: falta"; fi
```

```powershell
if ($env:CONNECTIONSTRINGS__CosmosDbMongoDB) { "OK: presente" } else { "ERROR: falta" }
```

### 2. Identificar el proyecto API del microservicio

Navegar al directorio `04.API/` del microservicio activo:
```bash
cd {repo-root}/04.API/{nombre-proyecto-api}
```

### 3. Levantar la API

```bash
# Linux/macOS
ASPNETCORE_ENVIRONMENT=Development \
ConnectionStrings__CosmosDbMongoDb="$CONNECTIONSTRINGS__CosmosDbMongoDB" \
dotnet run --urls "http://localhost:{puerto}"
```

```powershell
# Windows PowerShell
$env:ASPNETCORE_ENVIRONMENT = "Development"
$env:ConnectionStrings__CosmosDbMongoDb = $env:CONNECTIONSTRINGS__CosmosDbMongoDB
dotnet run --urls "http://localhost:{puerto}"
```

> La variable intermedia `ConnectionStrings__CosmosDbMongoDb` es necesaria porque .NET espera esa clave exacta de configuración.

### 4. Verificar que arrancó correctamente

Señales de éxito en los logs:
```
[Startup] Cosmos configuration: connectionString=present databaseName=marchamo_digital
Registered N endpoints in X milliseconds.
```

Confirmar Swagger:
```bash
curl -s -o /dev/null -w "%{http_code}" http://localhost:{puerto}/swagger
# Esperado: 200
```

### 5. Ejecutar requests de prueba

```bash
curl -X POST http://localhost:{puerto}/v0/{grupo}/{accion} \
  -H "Content-Type: application/json" \
  -H "X-Correlation-ID: test-$(date +%s)" \
  -d '{ ... body ... }'
```

## Puertos por servicio (docker-compose.local.yml)

| Servicio | Puerto local |
|---|---|
| Validaciones | 8101 |
| EstadosTransaccionales | 8102 |

## Alternativa: docker-compose multi-servicio

Desde la raíz del workspace:
```bash
docker compose -f docker-compose.local.yml up -d
curl http://localhost:8101/v0/health   # Validaciones
curl http://localhost:8102/health      # EstadosTransaccionales
```

## Notas
- Si ya compilaste antes, añadir `--no-build` para arrancar más rápido.
- Event Hub deshabilitado en local (no-op en INS.Core.Base).
- Cosmos DB real se usa directamente — los cambios afectan el ambiente DEV.

---

## Skill Collaboration

Cuando la solicitud requiera capacidades fuera del dominio de este Skill, coordinar con el Skill correspondiente siguiendo `framework/DECISION-TREE.md`.

| Necesidad | Skill |
|---|---|
| Explorar datos en Cosmos DB | `cosmos-marchamo-explorer` |
| Generar plan de pruebas QA | `qa-api-testing` |

---

## Completion Checklist

Antes de considerar terminada cualquier tarea:

- □ Se respetó el Framework (Principles, Workflow, GIT-Policy, Conventions).
- □ Se siguió el Workflow.
- □ Se utilizó el Skill correcto.
- □ Se mantuvieron cambios mínimos.
- □ Se mantuvo consistencia con el proyecto.
- □ Se espera aprobación del desarrollador antes de cualquier operación Git.
