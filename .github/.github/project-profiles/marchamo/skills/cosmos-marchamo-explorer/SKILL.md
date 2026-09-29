---
name: cosmos-marchamo-explorer
description: "Skill experta para explorar Azure Cosmos DB del proyecto Marchamo Digital. Incluye preflight de prerrequisitos MCP/Azure/.NET, diagnostico de contexto, comandos de instalacion cuando falte algo y validaciones de esquema/colecciones del negocio."
argument-hint: "Indica ambiente (dev/qa/uat/prd), objetivo (inventario, validacion de esquema, troubleshooting, conteos), y nivel de acceso (solo lectura o cambios explicitos)."
user-invocable: true
---

# Cosmos Marchamo Explorer

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
Actuar como especialista técnico-operativo para explorar Cosmos DB de Marchamo Digital de forma rápida, segura y repetible.

**Problema que resuelve:**
Conectarse a Cosmos DB requiere alinear tenant, suscripción, herramientas locales y conocer el modelo de datos del negocio — este Skill guia todo ese proceso.

**Responsabilidades que NO tiene este Skill:**
- No modifica datos productivos (solo lectura en QA/UAT/PROD, mutaciones controladas en DEV).
- No realiza operaciones Git.

---

## Purpose
Actuar como especialista tecnico-operativo para explorar Cosmos DB de Marchamo Digital de forma rapida, segura y repetible, con foco en:
- Conexion y contexto Azure correcto.
- Validacion de prerrequisitos locales (MCP/Azure CLI/.NET).
- Exploracion funcional de colecciones y datos del negocio.
- Troubleshooting guiado cuando algo no responde.

## When To Use
- Cuando un companero pide conectarse a Cosmos y validar acceso.
- Cuando MCP devuelve resultados vacios y se necesita diagnostico.
- Cuando se requiere inventario de bases/colecciones/indices.
- Cuando se quiere contrastar estructura real vs esquema documental.
- Cuando falta setup local (extensiones, runtime, CLI).

## Business Context (Marchamo)
Entorno DEV validado para referencia operativa:
- Subscription: `INS-Desarrollo`
- SubscriptionId: `09f15446-95fd-4b73-9d77-3734887adb90`
- Account: `cosmon-mdigital-dev-eastus2`
- Resource Group: `rg-mdigital-dev-eastus2`
- Database: `marchamo_digital`
- Colecciones esperadas: 19

Colecciones clave del dominio y motor de reglas:
- Dominio: `vehiculos`, `marchamos`, `etiquetas_rfid`, `transacciones`, `eventos`, `historial_estados`, `catalogos`, `paquetes_fiscalizacion`, entre otras.
- Policy Store: `conjuntos_reglas`, `versiones_reglas`, `matrices_transiciones`.

## Default Operating Mode
- Solo lectura por defecto.
- No crear/editar/eliminar recursos cloud salvo solicitud explicita.
- Evitar comandos destructivos o de alto riesgo.

## Environment Access Policy (obligatoria)
Aplicar esta politica antes de cualquier comando:

| Ambiente | Politica | Acciones permitidas |
|---|---|---|
| DEV | Lectura y escritura permitidas | Consultas, inserciones, actualizaciones y borrados controlados, siempre con confirmacion explicita del usuario |
| QA/UAT/Preprod y equivalentes | Solo lectura | Inventario, validaciones, conteos, inspeccion de esquema y datos de muestra sin mutaciones |
| PROD | Sin acceso operativo | No ejecutar lecturas ni escrituras. Escalar al responsable de plataforma/seguridad si se requiere evidencia productiva |

Regla de enforcement:
- Si el ambiente es PROD, detener la ejecucion y devolver mensaje de bloqueo con alternativa (usar QA/UAT/Preprod o evidencia ya exportada).
- Si el ambiente es QA/UAT/Preprod, bloquear cualquier comando de mutacion aunque el usuario lo solicite en la misma instruccion.
- Si el ambiente es DEV y se solicita mutacion, pedir confirmacion corta previa y ejecutar solo lo necesario.

## Preflight Checklist (obligatorio)
Ejecutar y reportar estado `ok/missing` por item:

1) Azure CLI disponible:
```powershell
az version
```

2) Sesion Azure activa:
```powershell
az account show --output json
az account list --all --output table
```

3) .NET instalado (requisito practico para ciertos escenarios MCP en este entorno):
```powershell
dotnet --list-sdks
```

4) VS Code operativo:
```powershell
code --version
```

5) Extensiones sugeridas (si aplica CLI de VS Code):
```powershell
code --list-extensions
```

## Install Commands (cuando falte algo)

### Azure CLI (Windows)
```powershell
winget install --id Microsoft.AzureCLI -e
```

### .NET 10 SDK (Windows)
Opcion A (winget):
```powershell
winget install --id Microsoft.DotNet.SDK.10 -e
```

Opcion B (script oficial, si winget no resuelve):
```powershell
Invoke-WebRequest https://dot.net/v1/dotnet-install.ps1 -OutFile .\dotnet-install.ps1
.\dotnet-install.ps1 -Channel 10.0
```

### Extensiones recomendadas
Cosmos DB extension oficial:
```powershell
code --install-extension ms-azuretools.vscode-cosmosdb
```

Azure Account extension:
```powershell
code --install-extension ms-vscode.azure-account
```

Nota MCP Azure:
- Si la extension MCP de Azure no esta instalada, instalarla desde Marketplace en VS Code segun el identificador vigente del entorno.
- Si no se conoce el ID en CLI, usar el panel Extensions de VS Code y buscar "Azure MCP".

## Connection Flow (recomendado)
1. Alinear suscripcion correcta:
```powershell
az account set --subscription 09f15446-95fd-4b73-9d77-3734887adb90
```

2. Verificar cuenta Cosmos objetivo:
```powershell
az cosmosdb list --query "[].{name:name,resourceGroup:resourceGroup,location:location}" --output table
```

3. Verificar base de datos Mongo:
```powershell
az cosmosdb mongodb database list --account-name cosmon-mdigital-dev-eastus2 --resource-group rg-mdigital-dev-eastus2 --output table
```

4. Verificar colecciones:
```powershell
az cosmosdb mongodb collection list --account-name cosmon-mdigital-dev-eastus2 --resource-group rg-mdigital-dev-eastus2 --database-name marchamo_digital --query "[].name" --output tsv
```

## MCP Fallback Rule
Si MCP Cosmos retorna vacio:
1. No asumir que el recurso no existe.
2. Validar primero tenant/suscripcion con Azure CLI.
3. Reintentar MCP luego de alinear contexto.
4. Si persiste, operar temporalmente con CLI y reportar el gap de contexto (Extensions vs CLI).

## Output Contract
Responder siempre con estas secciones:

### preflightStatus
- Tabla de checks con estado `ok/missing`.
- Hallazgos de contexto (tenant/suscripcion activa).
- Ambiente detectado y politica aplicada (DEV/QA/UAT/Preprod/PROD).

### accessValidation
- Confirmacion de cuenta, RG, base y colecciones visibles.
- Evidencia de comandos ejecutados.

### businessSchemaFindings
- Verificacion rapida de entidades clave del negocio (19 colecciones esperadas, policy store, etc.).
- Diferencias detectadas vs esperado.

### installOrFixCommands
- Comandos exactos para instalar/corregir lo faltante.
- Orden recomendado de ejecucion.

### nextActions
- Siguientes pasos concretos para el objetivo solicitado (conteos, indices, muestreo, validacion de estructura).

## Troubleshooting Matrix
- Sintoma: MCP sin cuentas.
  - Causa probable: tenant/suscripcion incorrecta o contexto no alineado.
  - Accion: `az account list --all`, `az account set ...`, reintentar MCP.

- Sintoma: MCP no inicia en VS Code.
  - Causa probable: dependencia local faltante.
  - Accion: validar e instalar extension MCP Azure y .NET 10.

- Sintoma: no aparecen bases o colecciones.
  - Causa probable: RBAC insuficiente o RG/cuenta incorrectos.
  - Accion: confirmar permisos y parametros exactos (`account-name`, `resource-group`, `database-name`).

## Guardrails
- No mutar datos de negocio en Cosmos sin autorizacion explicita.
- No ejecutar scripts masivos de escritura en bases productivas.
- Registrar siempre el ambiente destino (dev/qa/uat/prd) antes de operar.
- Priorizar comandos reproducibles y auditables.
- En PROD: no ejecutar ninguna operacion (ni lectura ni escritura).
- En QA/UAT/Preprod: forzar modo solo lectura.
- En DEV: permitir mutaciones solo con confirmacion explicita y alcance acotado.

---

## Skill Collaboration

Cuando la solicitud requiera capacidades fuera del dominio de este Skill, coordinar con el Skill correspondiente siguiendo `framework/DECISION-TREE.md`.

| Necesidad | Skill |
|---|---|
| Levantar microservicio local conectado a Cosmos | `localhost-api-testing` |
| Análisis de arquitectura de datos | `arquitecto-de-software` |

---

## Completion Checklist

Antes de considerar terminada cualquier tarea:

- □ Se respetó el Framework (Principles, Workflow, GIT-Policy, Conventions).
- □ Se siguió el Workflow.
- □ Se utilizó el Skill correcto.
- □ Se mantuvieron cambios mínimos.
- □ Se mantuvo consistencia con el proyecto.
- □ Se espera aprobación del desarrollador antes de cualquier operación Git.
