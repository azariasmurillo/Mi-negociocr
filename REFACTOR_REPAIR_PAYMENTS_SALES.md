# Refactor: Repair Orders + Prepagos + Facturación

> **Memoria del modelo financiero base (mayo 2026):** preservar motivación, decisiones y QA. Para evoluciones consultar [anulación/saldo a favor](./MiNegocioCR.Api/docs/SALE_VOID_STORE_CREDIT_AGO_2026.md), [Créditos v1.2](./MiNegocioCR.Api/docs/CAMBIOS_CREDITOS_MULTI_CUENTA_AGO_2026.md) y [diagnóstico/abonos](./MiNegocioCR.Api/docs/CAMBIOS_REPAIR_INCOME_DIAGNOSTIC_ABONO_AGO_2026.md). RegisterSaleUseCase determina el comportamiento actual; los SQL y checklists de este registro no son autorización para ejecutarlos.

**Fecha:** Mayo 2026  
**Repositorio:** `c:\Mi-negociocr`  
**Stack:** ASP.NET Core 8 (API) + Angular 21 (Frontend) + PostgreSQL (Supabase/Railway)

---

## Índice

1. [Contexto y motivación](#1-contexto-y-motivación)
2. [Arquitectura del sistema](#2-arquitectura-del-sistema)
3. [Problemas que se resolvieron](#3-problemas-que-se-resolvieron)
4. [Nuevo modelo de datos](#4-nuevo-modelo-de-datos)
5. [Lógica financiera correcta](#5-lógica-financiera-correcta)
6. [Flujos de negocio](#6-flujos-de-negocio)
7. [Cambios en la base de datos](#7-cambios-en-la-base-de-datos)
8. [Cambios en el backend](#8-cambios-en-el-backend)
9. [Cambios en el frontend](#9-cambios-en-el-frontend)
10. [Reglas de negocio importantes](#10-reglas-de-negocio-importantes)
11. [Lo que NO se cambió intencionalmente](#11-lo-que-no-se-cambió-intencionalmente)
12. [Bugs adicionales corregidos](#12-bugs-adicionales-corregidos)
13. [Guía QA — casos a probar](#13-guía-qa--casos-a-probar)
14. [Segunda ola — descuentos unificados (Mayo 2026)](#14-segunda-ola--descuentos-unificados-mayo-2026)
15. [Script SQL manual si falla `database update`](#15-script-sql-manual-si-falla-database-update)
16. [Bugs corregidos tras la segunda ola](#16-bugs-corregidos-tras-la-segunda-ola)
17. [Checklist deploy — base de datos](#17-checklist-deploy--base-de-datos)

---

## 1. Contexto y motivación

El sistema gestiona un **taller de reparación** (electrónica, dispositivos). El flujo principal es:

```
Cliente llega → Orden de Reparación → Técnico trabaja → Cliente abona → Facturación final
```

Antes del refactor, el sistema tenía una mezcla conceptual grave:

- Los **prepagos/abonos** del cliente se trataban como si fueran **descuentos** en la factura.
- Los métodos de pago se representaban con **4 booleans** (`PayCash`, `PayTransfer`, `PaySinpe`, `PayCard`) que no soportaban montos reales ni pagos mixtos.
- El POS (Point of Sale) permitía editar ítems, precios y descuentos de una orden de reparación aunque el backend ignoraba esos cambios.
- La ganancia (`TotalProfit`) se calculaba sobre el saldo cobrado hoy, no sobre el valor total de la orden.

---

## 2. Arquitectura del sistema

### Estructura del monorepo

```
c:\Mi-negociocr\
├── MiNegocioCR.Api/          ← ASP.NET Core 8 backend
│   ├── API/Controllers/       ← Controladores HTTP
│   ├── Application/
│   │   ├── DTOs/              ← Data Transfer Objects
│   │   ├── Interfaces/        ← Contratos (use cases, repositorios)
│   │   └── UseCases/          ← Lógica de negocio
│   ├── Domain/Entities/       ← Entidades del dominio
│   ├── Infrastructure/
│   │   └── Persistence/       ← AppDbContext, repositorios, migrations
│   └── Migrations/            ← Migraciones EF Core
│
└── mi-negociocr-frontend/    ← Angular 21 frontend
    └── src/app/features/
        ├── repairs/            ← Módulo de órdenes de reparación
        ├── sales/              ← Módulo de ventas/POS
        └── dashboard/         ← Dashboard y métricas
```

### Autenticación

- JWT-based con `BusinessId` embebido en el token.
- Todos los controladores tienen `[Authorize]`.
- `BusinessId` se extrae del token en el backend; nunca se confía en el frontend para datos de negocio.

### Base de datos

PostgreSQL via EF Core 8. Multi-tenant por `BusinessId` en cada tabla de datos de negocio.

---

## 3. Problemas que se resolvieron

### 3.1 Cálculo incorrecto de descuentos (BUG CRÍTICO)

**Antes (INCORRECTO):**
```csharp
sale.DiscountAmount = discountPercentAmount + totalPagado;
// Los abonos del cliente se sumaban al descuento — contablemente INCORRECTO
```

**Después (CORRECTO — descuento solo al facturar, en `Sales`):**
```csharp
// En RegisterSaleUseCase (POS o reparación): descuento del request de venta
var (discountKind, discountInput, discountAmount) = SaleDiscountCalculator.Resolve(
    subtotal, request.DiscountKind, request.DiscountValue, request.Discount);

// Abonos previos son un campo separado (reparación)
sale.PrepaidAmount = payments.Sum(p => p.Amount);

// Saldo a cobrar hoy
sale.Total = Math.Max(0, totalOrden - prepaidAmount);
```

> **Importante:** `RepairOrders` **ya no tiene** `DiscountPercent`. El descuento se define en el POS al facturar (venta manual o desde reparación), no al crear/editar la orden.

### 3.2 Métodos de pago sin montos reales

**Antes:** 4 booleans en `Sales` y `RepairOrders`  
**Después:** Entidad `SalePaymentMethod` con `Method` + `Amount`, soporte de pagos mixtos reales.

### 3.3 POS editable para ventas de reparación

**Antes:** El POS permitía editar ítems/descuento aunque el backend los ignoraba.  
**Después:** Con `repairOrderId`, ítems y precios son **solo lectura** (vienen de la orden). El **descuento se aplica al facturar** en el mismo POS (% o ₡), igual que en venta manual — **no** existe descuento guardado en `RepairOrders`.

### 3.4 Ganancia mal calculada para órdenes con prepagos

**Antes:** `TotalProfit = sale.Total - TotalCost`  
→ Si una orden vale ₡100,000 y el cliente ya abonó ₡80,000, `sale.Total = ₡20,000`, por lo que la ganancia reportada era solo ₡20,000.

**Después:** `TotalProfit = (TotalOrden - TaxAmount) - TotalCost`  
→ La ganancia se calcula sobre el valor total de la orden, independientemente de los abonos.

---

## 4. Nuevo modelo de datos

### Separación conceptual de entidades

```
RepairOrder          Payment (abonos)       Sale (factura final)
─────────────        ────────────────       ────────────────────
Trabajo técnico      Prepago real           Factura del saldo
Items y servicios    Monto real             Descuento (% o ₡) ← solo aquí
Impuesto % (negocio) Fecha                  Descuenta inventario
                     Método                 Marca orden Invoiced
                     Referencia             SalePaymentMethods
                     Notas
```

### Entidad: `RepairOrder`

- Representa el trabajo técnico.
- Define ítems/servicios e impuesto (% del negocio en el balance).
- **NO tiene descuento** (`DiscountPercent` eliminado — migración `20260526120000`).
- **NO factura**, **NO descuenta stock**, **NO registra pagos finales**.
- Estados: `Pending → InProcess → Processed → Delivered`. `Cancelled` es terminal.

### Entidad: `Payment` (abonos de reparación)

```csharp
public class Payment {
    public Guid Id { get; set; }
    public Guid BusinessId { get; set; }
    public Guid RepairOrderId { get; set; }
    public decimal Amount { get; set; }
    public PaymentMethod Method { get; set; }   // Cash | Transfer | Sinpe | Card
    public PaymentType Type { get; set; }        // Advance | Deposit | Diagnostic | Final
    public string? Reference { get; set; }       // NUEVO: código SINPE, nro transferencia, etc.
    public string? Notes { get; set; }
    public DateTime CreatedAt { get; set; }
}
```

### Entidad: `Sale` (factura final)

```csharp
public class Sale {
    public Guid Id { get; set; }
    public Guid BusinessId { get; set; }
    public Guid? RepairOrderId { get; set; }     // null = venta manual
    public decimal Subtotal { get; set; }
    public decimal DiscountAmount { get; set; }   // descuento al facturar (NO incluye abonos)
    public byte DiscountKind { get; set; }          // None | Percent | FixedAmount
    public decimal DiscountInputValue { get; set; } // valor tipeado (% o ₡)
    public decimal TaxAmount { get; set; }
    public decimal TotalOrden { get; set; }       // NUEVO: total bruto de la orden
    public decimal PrepaidAmount { get; set; }    // NUEVO: suma de abonos previos
    public decimal Total { get; set; }            // saldo cobrado hoy (TotalOrden - PrepaidAmount)
    public ICollection<SalePaymentMethod> PaymentMethods { get; set; }  // NUEVO
    // ... contacto, items, invoice number, etc.
}
```

### Entidad: `SalePaymentMethod` (NUEVA)

```csharp
public class SalePaymentMethod {
    public Guid Id { get; set; }
    public Guid SaleId { get; set; }
    public PaymentMethod Method { get; set; }   // Cash | Transfer | Sinpe | Card
    public decimal Amount { get; set; }
    public Sale Sale { get; set; }
}
```

Permite pagos mixtos reales:
```
Efectivo   ₡10,000
SINPE      ₡25,000
Tarjeta    ₡ 5,000
─────────────────
Total      ₡40,000
```

### Enum: `PaymentMethod`

```csharp
public enum PaymentMethod {
    Cash     = 0,
    Transfer = 1,
    Sinpe    = 2,
    Card     = 3
}
```

---

## 5. Lógica financiera correcta

### Balance de la orden de reparación (sin descuento)

`GET /api/RepairOrders/{id}/balance` — solo ítems + IVA del negocio + abonos:

```
Subtotal      = suma de (cantidad × precio) de todos los ítems
TaxAmount     = Subtotal × (TaxRatePercent / 100)
TotalOrden    = Subtotal + TaxAmount          ← sin descuento en la orden

PrepaidAmount = suma de todos los Payment.Amount para esta RepairOrder
SaldoPendiente = max(0, TotalOrden - PrepaidAmount)
```

El campo `Discount` en el DTO de balance se devuelve en **0** (legacy; no hay descuento en orden).

### Factura / venta (precios con IVA incluido)

Al guardar la venta (`RegisterSaleUseCase`), el descuento se ingresa en el POS:

```
Subtotal bruto  = suma de ítems (precios con IVA incluido)
Descuento       = % o monto fijo (SaleDiscountCalculator)
TotalOrden      = Subtotal bruto − Descuento
TaxAmount       = TotalOrden − (TotalOrden ÷ (1 + rate/100))   ← extraído, no sumado

PrepaidAmount   = abonos previos (solo reparación)
Total (hoy)     = max(0, TotalOrden − PrepaidAmount)
TotalProfit     = (TotalOrden − TaxAmount) − TotalCost
```

### Qué muestra la factura final

```
Subtotal (IVA incl.)          ₡ 85,000
Descuento (15%)             − ₡ 12,750
Impuesto incluido (13%)       ₡  9,778   ← desglose
                            ══════════
Total orden                   ₡ 72,250
Abonos previos              − ₡ 60,000
                            ══════════
Saldo cobrado hoy             ₡ 12,250
```

### Invariantes financieros garantizados

- `DiscountAmount` NUNCA incluye abonos (son categorías distintas).
- `PrepaidAmount` NUNCA es negativo.
- `Total` NUNCA es negativo: `max(0, TotalOrden - PrepaidAmount)`.
- `TotalProfit = (TotalOrden - TaxAmount) - TotalCost` — ganancia neta sin IVA (pass-through).

---

## 6. Flujos de negocio

### Flujo A: Orden de reparación sin prepagos

```
1. Crear RepairOrder (técnico documenta trabajo + ítems)
2. Cambiar estado: Pending → InProcess → Processed
3. Cliente llega, sin abonos previos
4. Abrir POS: /sales?repairOrderId={id}
5. POS: ítems read-only; **descuento editable** (% o ₡) al facturar
6. Elegir método(s) de pago + monto
7. Guardar y generar factura
8. Backend: crea Sale, descuenta stock, marca RepairOrder como Invoiced
9. Estado → Delivered
```

### Flujo B: Orden de reparación con múltiples abonos

```
1. Crear RepairOrder
2. Cliente abona por adelantado → Registrar pago (dialog en la orden)
   - Puede haber N pagos con distintos métodos y referencias
3. Cliente abona nuevamente → Registrar otro pago
4. Orden procesada → Abrir POS
5. POS muestra: Total orden, Total abonado, Saldo pendiente
6. Si saldo = 0 → botón deshabilitado "orden ya pagada"
7. Si saldo > 0 → elegir método para el saldo restante
8. Generar factura con desglose completo
```

### Flujo C: Venta manual (sin orden de reparación)

```
1. Abrir POS en modo manual
2. Agregar productos del catálogo y/o servicios
3. Aplicar descuento % si aplica
4. Elegir método(s) de pago con montos
5. Guardar y/o generar factura
6. Backend: crea Sale, descuenta stock de inventario
```

### Flujo D: Facturación directa desde orden (endpoint `/charge`)

```
POST /api/RepairOrders/{id}/charge
Body: { paymentMethods: [{ method: "Cash", amount: 40000 }] }
```

Delega en `RegisterSaleUseCase` con los datos de la orden como fuente de verdad.

---

## 7. Cambios en la base de datos

### Migración: `20260522120000_RefactorPaymentsAndSalePaymentMethods`

#### Tabla nueva: `SalePaymentMethods`

```sql
CREATE TABLE "SalePaymentMethods" (
    "Id"      uuid           NOT NULL DEFAULT gen_random_uuid(),
    "SaleId"  uuid           NOT NULL,
    "Method"  integer        NOT NULL DEFAULT 0,
    "Amount"  numeric(18,2)  NOT NULL DEFAULT 0,
    CONSTRAINT "PK_SalePaymentMethods" PRIMARY KEY ("Id"),
    CONSTRAINT "FK_SalePaymentMethods_Sales_SaleId"
        FOREIGN KEY ("SaleId") REFERENCES "Sales"("Id") ON DELETE CASCADE
);
CREATE INDEX "IX_SalePaymentMethods_SaleId" ON "SalePaymentMethods"("SaleId");
```

#### Columnas nuevas en `Sales`

```sql
ALTER TABLE "Sales"
    ADD COLUMN "TotalOrden"    numeric(18,2) NOT NULL DEFAULT 0,
    ADD COLUMN "PrepaidAmount" numeric(18,2) NOT NULL DEFAULT 0;

-- Poblar datos iniciales (entorno dev/staging)
UPDATE "Sales" SET "TotalOrden" = "Total", "PrepaidAmount" = 0;
```

#### Columnas eliminadas de `Sales`

```sql
ALTER TABLE "Sales"
    DROP COLUMN "PayCash",
    DROP COLUMN "PayTransfer",
    DROP COLUMN "PaySinpe",
    DROP COLUMN "PayCard";
```

#### Columnas eliminadas de `RepairOrders`

```sql
ALTER TABLE "RepairOrders"
    DROP COLUMN "PayCash",
    DROP COLUMN "PayTransfer",
    DROP COLUMN "PaySinpe",
    DROP COLUMN "PayCard";
```

#### Nueva columna en `Payments`

```sql
ALTER TABLE "Payments"
    ADD COLUMN "Reference" text NULL;
```

#### Registrar migración en EF Core

```sql
INSERT INTO "__EFMigrationsHistory" ("MigrationId", "ProductVersion")
VALUES ('20260522120000_RefactorPaymentsAndSalePaymentMethods', '8.0.8')
ON CONFLICT ("MigrationId") DO NOTHING;
```

### Migración: `20260504220000_AddSaleCostAndProfitMetrics`

Costos en líneas y métricas de ganancia en ventas.

```sql
ALTER TABLE "SaleItems" ADD COLUMN IF NOT EXISTS "CostPrice" numeric(18,2) NOT NULL DEFAULT 0;
ALTER TABLE "Sales" ADD COLUMN IF NOT EXISTS "TotalCost" numeric(18,2) NOT NULL DEFAULT 0;
ALTER TABLE "Sales" ADD COLUMN IF NOT EXISTS "TotalProfit" numeric(18,2) NOT NULL DEFAULT 0;

INSERT INTO "__EFMigrationsHistory" ("MigrationId", "ProductVersion")
SELECT '20260504220000_AddSaleCostAndProfitMetrics', '8.0.8'
WHERE NOT EXISTS (
  SELECT 1 FROM "__EFMigrationsHistory" WHERE "MigrationId" = '20260504220000_AddSaleCostAndProfitMetrics'
);
```

### Migración: `20260526120000_RemoveRepairOrderDiscountPercent`

El descuento **ya no vive en la orden de reparación**; se aplica al facturar en `Sales`.

```sql
ALTER TABLE "RepairOrders" DROP COLUMN IF EXISTS "DiscountPercent";

INSERT INTO "__EFMigrationsHistory" ("MigrationId", "ProductVersion")
SELECT '20260526120000_RemoveRepairOrderDiscountPercent', '8.0.8'
WHERE NOT EXISTS (
  SELECT 1 FROM "__EFMigrationsHistory" WHERE "MigrationId" = '20260526120000_RemoveRepairOrderDiscountPercent'
);
```

### Migración: `20260526130000_AddSaleDiscountMetadata`

Metadata del descuento ingresado en el POS (además de `DiscountAmount` en colones).

```sql
ALTER TABLE "Sales" ADD COLUMN IF NOT EXISTS "DiscountKind" smallint NOT NULL DEFAULT 0;
ALTER TABLE "Sales" ADD COLUMN IF NOT EXISTS "DiscountInputValue" numeric(18,2) NOT NULL DEFAULT 0;

INSERT INTO "__EFMigrationsHistory" ("MigrationId", "ProductVersion")
SELECT '20260526130000_AddSaleDiscountMetadata', '8.0.8'
WHERE NOT EXISTS (
  SELECT 1 FROM "__EFMigrationsHistory" WHERE "MigrationId" = '20260526130000_AddSaleDiscountMetadata'
);
```

| Columna | Tipo | Significado |
|---------|------|-------------|
| `DiscountAmount` | numeric(18,2) | Monto aplicado en ₡ (ya existía) |
| `DiscountKind` | smallint | `0=None`, `1=Percent`, `2=FixedAmount` |
| `DiscountInputValue` | numeric(18,2) | Valor tipeado (% o ₡ según kind) |

### Migraciones con `.Designer.cs` (mayo 2026)

Las migraciones `20260522120000`, `20260526120000` y `20260526130000` tenían archivos `.Designer.cs` faltantes; EF las ignoraba. Se agregaron atributos `[Migration]` para que `dotnet ef database update` y `Database.Migrate()` al arrancar las detecten.

### Script unificado (recomendado si EF falla)

Ver **[`MiNegocioCR.Api/Scripts/apply-pending-migrations.sql`](MiNegocioCR.Api/Scripts/apply-pending-migrations.sql)** — aplica todo lo anterior de forma idempotente.

Verificación: **[`MiNegocioCR.Api/Scripts/verify-schema.sql`](MiNegocioCR.Api/Scripts/verify-schema.sql)**

---

## 8. Cambios en el backend

### 8.1 Entidades modificadas

| Archivo | Cambio |
|---------|--------|
| `Domain/Entities/Sale.cs` | Elimina `PayCash/Transfer/Sinpe/Card`; agrega `TotalOrden`, `PrepaidAmount`, navegación `ICollection<SalePaymentMethod>` |
| `Domain/Entities/RepairOrder.cs` | Elimina `PayCash/Transfer/Sinpe/Card` y **`DiscountPercent`** (descuento solo en `Sale`) |
| `Domain/Entities/Payment.cs` | Agrega `Reference` nullable |
| `Domain/Entities/SalePaymentMethod.cs` | **NUEVO** — `Id`, `SaleId`, `Method (enum)`, `Amount` |

### 8.2 DTOs modificados

| Archivo | Cambio |
|---------|--------|
| `DTOs/CreateSaleRequest.cs` | Elimina 4 booleans; agrega `List<SalePaymentMethodDto> PaymentMethods` |
| `DTOs/CreateSaleFromRepairRequestDto.cs` | `BusinessId` + `PaymentMethods` + descuento opcional (`DiscountKind`, `DiscountValue`) |
| `DTOs/SalePaymentMethodDto.cs` | **NUEVO** — `string Method`, `decimal Amount` |
| `DTOs/ChargeRepairOrderRequestDto.cs` | **NUEVO** — `List<SalePaymentMethodDto> PaymentMethods` |
| `DTOs/SalesListItemDto.cs` | Elimina 4 booleans; agrega `TotalOrden`, `PrepaidAmount`, `List<SalePaymentMethodDto>` |
| `DTOs/CreatePaymentRequestDto.cs` | Agrega `string? Reference` |
| `DTOs/UpdateRepairOrderRequestDto.cs` | Elimina 4 booleans y **descuento** |

### 8.3 Use Cases modificados

#### `RegisterSaleUseCase` (núcleo financiero)

**Ruta con `RepairOrderId`:**
1. Carga `RepairOrder` + `Items` desde BD — **ignora ítems** del request.
2. Carga `Business.TaxRatePercent` desde BD.
3. Descuento desde el **request de venta** (`DiscountKind`, `DiscountValue`, `Discount`) vía `SaleDiscountCalculator` — **no** desde la orden.
4. Calcula: `Subtotal bruto (IVA incl.) → Descuento → TotalOrden`; `TaxAmount` extraído.
5. Suma abonos de la orden: `PrepaidAmount`.
6. `Total = max(0, TotalOrden - PrepaidAmount)`.
7. Guarda `SalePaymentMethod` por cada entrada en `request.PaymentMethods`.
8. `TotalProfit = (TotalOrden - TaxAmount) - TotalCost`.
9. Descuenta stock solo en este momento.

**Ruta manual (sin `RepairOrderId`):**
1. Toma ítems del request.
2. Misma fórmula IVA incluido; `PrepaidAmount = 0`.

**Validaciones:**
- No facturar órdenes canceladas.
- No facturar órdenes ya facturadas (`IsInvoiced = true`).
- No facturar si `PrepaidAmount >= TotalOrden` (ya está completamente pagado).
- `PaymentMethod` desconocido → excepción descriptiva (no silenciar).

**`ParsePaymentMethod`** (case-insensitive, soporta español):
```csharp
"cash" | "efectivo"        → Cash
"transfer" | "transferencia" → Transfer
"sinpe" | "sinpe móvil"    → Sinpe
"card" | "tarjeta"         → Card
otro                       → ArgumentException
```

#### `CreateSaleFromRepairUseCase`

Solo pasa `BusinessId`, `RepairOrderId`, `Source = "Repair"`, `PaymentMethods` y descuento opcional a `RegisterSaleUseCase`. Los ítems vienen de la orden en BD; el descuento del request de venta.

#### `ChargeRepairOrderUseCase`

Acepta `List<SalePaymentMethodDto>?` opcional vía `POST /api/RepairOrders/{id}/charge`. Delega en `RegisterSaleUseCase`.

#### `CreatePaymentUseCase`

Persiste `Reference` (trimmed, nullable) en la entidad `Payment`.

### 8.4 Repositorios modificados

#### `SaleRepository`

- `GetSaleAsync` y `GetSaleByIdAsync`: incluyen `.Include(x => x.PaymentMethods)`.
- `GetPagedSalesAsync`: proyecta `TotalOrden`, `PrepaidAmount`, y mapea `PaymentMethods` al DTO.
- Filtro por método de pago: usa `x.PaymentMethods.Any(pm => (int)pm.Method == method)`.

### 8.5 Interfaces modificadas

- `IAppDbContext`: agrega `DbSet<SalePaymentMethod> SalePaymentMethods`.
- `IChargeRepairOrderUseCase`: agrega parámetro `List<SalePaymentMethodDto>? paymentMethods = null`.

### 8.6 AppDbContext

```csharp
// Sale
builder.Entity<Sale>()
    .HasMany(s => s.PaymentMethods)
    .WithOne(pm => pm.Sale)
    .HasForeignKey(pm => pm.SaleId)
    .OnDelete(DeleteBehavior.Cascade);

builder.Entity<Sale>()
    .Property(s => s.TotalOrden).HasColumnType("numeric(18,2)").HasDefaultValue(0m);

builder.Entity<Sale>()
    .Property(s => s.PrepaidAmount).HasColumnType("numeric(18,2)").HasDefaultValue(0m);

// TotalAmount: columna legacy NOT NULL; alias de Total (cobrado hoy).
// ValueGeneratedNever + sync en SaveChangesAsync — ver FIXES_MAYO_2026 §25.
builder.Entity<Sale>()
    .Property(s => s.TotalAmount)
    .HasColumnType("numeric(18,2)")
    .ValueGeneratedNever();

// SalePaymentMethod
builder.Entity<SalePaymentMethod>()
    .HasIndex(pm => pm.SaleId);
```

---

## 9. Cambios en el frontend

### 9.1 Modelos actualizados

#### `sale-invoice.models.ts`

```typescript
export interface SaleInvoicePaymentMethod {
  method: string;   // 'Cash' | 'Transfer' | 'Sinpe' | 'Card'
  amount: number;
}

export interface SaleInvoiceDetail {
  // Nuevos campos financieros
  totalOrden?: number | null;       // total bruto de la orden
  prepaidAmount?: number | null;    // suma de abonos previos
  paymentMethods?: SaleInvoicePaymentMethod[] | null;
  // ...resto de campos existentes
  /** @deprecated Usar totalOrden */ grossTotal?: number | null;
  /** @deprecated Usar prepaidAmount */ priorPaymentsTotal?: number | null;
}
```

#### `repair-order.models.ts`

- `RepairOrderDetail`: elimina `payCash/payTransfer/paySinpe/payCard`.
- `RepairOrderPayment`: agrega `reference?: string | null`.
- `RepairOrderBalance`: **sin** `discountPercent` — total = subtotal + IVA; descuento solo al facturar.
- `CreateRepairOrderRequest` / `UpdateRepairOrderRequest`: elimina 4 booleans y **elimina descuento**.

#### `dashboard.models.ts`

```typescript
export interface DashboardSaleListItem {
  // Reemplaza los 4 booleans
  paymentMethods?: Array<{ method: string; amount: number }> | null;
}
```

### 9.2 Servicios actualizados

#### `sales.service.ts`

```typescript
export type SalePaymentMethodName = 'Cash' | 'Transfer' | 'Sinpe' | 'Card';

export interface SalePaymentMethod {
  method: SalePaymentMethodName;
  amount: number;
}

// createManualSale → body usa paymentMethods[] + discountKind/discountValue
// createFromRepair → businessId + paymentMethods[] + descuento opcional
```

#### `payments.service.ts` (repair payments)

Agrega `reference?: string | null` en `CreateRepairPaymentRequest` y en el body del POST.

### 9.3 Mappers actualizados

#### `sale-invoice-api.mapper.ts`

- Aplana el objeto `Totals` anidado del response de API antes de mapear.
- Mapea `totalOrden`, `prepaidAmount`, y `paymentMethods[]` con sus montos.

```typescript
const totalsNested = pick<unknown>(root, 'totals', 'Totals');
const flatRoot: JsonObject = isJsonObject(totalsNested)
  ? { ...root, ...totalsNested }
  : root;

return {
  totalOrden: pick(flatRoot, 'totalOrden', 'TotalOrden') != null
    ? Number(pick(flatRoot, 'totalOrden', 'TotalOrden')) : null,
  prepaidAmount: pick(flatRoot, 'prepaidAmount', 'PrepaidAmount') != null
    ? Number(pick(flatRoot, 'prepaidAmount', 'PrepaidAmount')) : null,
  paymentMethods: (() => {
    const pmRaw = pick(root, 'paymentMethods', 'PaymentMethods');
    if (!Array.isArray(pmRaw)) return null;
    return pmRaw.map(pm => ({
      method: String(pm['method'] ?? pm['Method'] ?? ''),
      amount: Number(pm['amount'] ?? pm['Amount'] ?? 0),
    }));
  })(),
};
```

#### `repair-order-api.mapper.ts`

- `normalizeRepairOrderPayment`: agrega mapeo de `reference`/`Reference`.
- `normalizeRepairOrderBalanceFromApi`: subtotal, tax, total, abonos — **sin** descuento en orden.
- `normalizeRepairOrderDetailFromApi`: elimina mapeo de `payCash/payTransfer/paySinpe/payCard`.

#### `sale-invoice-from-repair-draft.ts`

- `mergeSaleInvoiceWithPersistedDraft`: fusiona `totalOrden`, `prepaidAmount`, y `paymentMethods` además de los campos legacy.
- `buildSaleInvoiceDraftFromManualPos`: acepta `paymentMethods?: SalePaymentMethod[]` en lugar de `payment?: ManualSalePaymentSnapshot`.

### 9.4 Componentes UI actualizados

#### `sales-manual.ts` / `sales-manual.html` (POS)

**Modo reparación en POS** (`hasRepairOrderSummary = true`, URL `?repairOrderId=`):

- Panel "Agregar producto" → **oculto**
- Panel "Servicio manual" → **oculto**
- Inputs de cantidad/precio en carrito → **solo lectura**
- Botones de eliminar línea → **ocultos**
- **Descuento (% / ₡)** → **editable** (se aplica al guardar la venta, no en la orden)
- Banner: "Venta desde orden de reparación…"

**UI de métodos de pago con montos:**
```typescript
paymentMethods: SalePaymentMethod[] = [];

addPaymentMethod(method: SalePaymentMethodName): void { ... }
removePaymentMethod(method: SalePaymentMethodName): void { ... }
hasPaymentMethod(method: SalePaymentMethodName): boolean { ... }
```

**`canSubmit`** ahora requiere:
- Al menos un método de pago con `amount > 0`.
- Datos de contacto (nombre + teléfono, o contacto seleccionado).
- Carrito no vacío.
- No en estado "completamente pagado".

**Bug corregido:** `(taxRatePercentFromApi ?? 0) | number` (precedencia de operadores).

#### `repair-order-edit-dialog.ts`

**Resumen financiero** en la sección de pagos (sin fila de descuento — el descuento es al facturar):

```
┌─────────────┬────────────┬──────────────┬────────────┬──────────────────┐
│  Subtotal   │ IVA (13%)  │ Total orden  │  Abonado   │ Saldo pendiente  │
│  ₡ 85,000  │  ₡ 11,050  │  ₡ 96,050   │  ₡ 60,000  │    ₡ 36,050     │
└─────────────┴────────────┴──────────────┴────────────┴──────────────────┘
```

**Tabla de pagos** muestra: Fecha | Método | Tipo | **Referencia** | Monto

Eliminado: checkboxes de `payCash/payTransfer/paySinpe/payCard`.

#### `register-repair-payment-dialog.ts`

Nuevo campo **Referencia** (código SINPE, número de transferencia, etc.) — opcional, máx 200 caracteres.

#### `sales-invoice-print.html` / `sales-invoice-print.ts`

- `effectiveGrossInvoiceTotal()`: prefiere `totalOrden`, fallback a `grossTotal`.
- `effectivePriorPayments()`: prefiere `prepaidAmount`, fallback a `priorPaymentsTotal`.
- Template muestra: `Total orden → Abonos previos → Saldo cobrado hoy`.
- Sección de métodos de pago: lista dinámica con montos (no checkboxes).
- Form `extras` simplificado: solo campo `notes`.

#### `sales-email-html.ts`

- `paymentList()`: lista métodos desde `sale.paymentMethods[]` con montos formateados.
- Totales de email: prefiere `totalOrden`/`prepaidAmount`; muestra desglose correcto.

#### `repair-order-print.html`

- Reemplaza checkboxes de `payCash/etc.` con lista de pagos registrados (`payments[]`).

#### `repair-order-email-html.ts`

- Muestra métodos de pago derivados de los pagos registrados en la orden.

#### `dashboard.ts` / `dashboard.service.ts`

- `paymentLabel()` usa `paymentMethods[]` con etiquetas en español.
- Normalización de `paymentMethods` desde API en el servicio.

---

## 10. Reglas de negocio importantes

### Inventario

El stock se descuenta **ÚNICAMENTE** cuando se genera la `Sale` final.  
**No** al crear la `RepairOrder`.  
**No** al cambiar estados de la orden.  
**No** al registrar abonos.

### Backend es la fuente de verdad

Cuando `repairOrderId != null` en una venta:
- El backend lee `Items` y `TaxRatePercent` de la orden / negocio en BD.
- El **descuento** se toma del request de venta (`DiscountKind`, `DiscountValue`, `Discount`) — ya **no** de `RepairOrder.DiscountPercent` (columna eliminada).
- Solo se aceptan `PaymentMethods` y metadata de cliente del frontend para completar la factura.

### Transiciones de estado de órdenes

```
Pending → InProcess → Processed → Delivered
                                       ↑
                              al facturar (Sale)

Cancelled (terminal — no se puede reactivar)
```

### No se puede facturar si:

- La orden está `Cancelled`.
- La orden ya fue facturada (`IsInvoiced = true`).
- Ya existe un `Sale` con el mismo `RepairOrderId` (índice único).

### Se puede facturar con saldo ₡0 (sin métodos de pago hoy) cuando:

- Hay **abonos previos** + **descuento al facturar** y `TotalOrden − PrepaidAmount ≤ 0` (donación/cortesía).
- Los abonos pueden ser **mayores** que el `TotalOrden` con descuento; se guarda `PrepaidAmount` real y `Total = 0`.

### Métodos de pago válidos (enum backend)

| Valor en API | Enum C# | Etiqueta en UI |
|---|---|---|
| `"Cash"` / `"cash"` / `"efectivo"` | `Cash = 0` | Efectivo |
| `"Transfer"` / `"transferencia"` | `Transfer = 1` | Transferencia |
| `"Sinpe"` / `"sinpe móvil"` | `Sinpe = 2` | SINPE móvil |
| `"Card"` / `"tarjeta"` | `Card = 3` | Tarjeta |

---

## 11. Lo que NO se cambió intencionalmente

| Tema | Razón |
|------|-------|
| `AppDbContextModelSnapshot` | EF Core lo regenera al hacer `dotnet ef migrations add`. |
| No se crea validación estricta de suma `PaymentMethods == Total` en backend | La UI muestra Falta/Sobrante; el registro es flexible. |
| `ChargeRepairOrderUseCase` sigue siendo opcional en métodos de pago | Permite facturar sin especificar método desde integraciones externas. |
| `CreatePaymentUseCase` sin validación de saldo máximo en abonos | Al facturar, `Total = max(0, TotalOrden − PrepaidAmount)`; ver FIXES §25 para `TotalAmount`. |

---

## 12. Bugs adicionales corregidos (en este mismo refactor)

Estos bugs se corrigieron durante sesiones previas, como parte de la revisión general del sistema:

| Bug | Archivo | Fix |
|-----|---------|-----|
| URL de búsqueda de Repair Orders incorrecta (404) | `repair-orders.service.ts` | Corregida ruta `/api/RepairOrders/{businessId}/search` |
| Ningún controlador tenía `[Authorize]` | Todos los controllers | Agregado `[Authorize]` a nivel clase |
| `RegisterSaleUseCase` duplicado en namespace incorrecto | `Application/UseCases/Repository/RegisterSaleUseCase.cs` | Eliminado (código muerto) |
| `InventoryMovementsService.registerSale()` parseaba GUID de response con body complejo | `inventory-movements.service.ts` | Corregido parsing |
| 6 intentos fallidos de URL para imágenes de Repair Orders | `repair-orders.service.ts` | Limpiado `chainRepairOrderImageGet/Post` |
| Race condition al restaurar sesión en el arranque | `app.ts` | `APP_INITIALIZER` espera `restoreSession()` |
| `VariantsService.getVariantsByCatalogItem()` cargaba todo y filtraba en browser | `variants.service.ts` | Delegado filtro al backend |

---

## 13. Guía QA — casos a probar

### Venta manual (sin repair order)

- [ ] Crear venta con un solo método de pago (efectivo)
- [ ] Crear venta con métodos mixtos (efectivo + SINPE)
- [ ] Verificar que el botón de guardar esté **deshabilitado** si no se agrega ningún método de pago
- [ ] Verificar que el botón esté deshabilitado si todos los métodos tienen monto 0
- [ ] Verificar que el stock del producto disminuye al guardar la venta
- [ ] Verificar descuento: `total = (subtotal × (1 - discount%)) × (1 + tax%)`

### Venta desde orden de reparación (sin abonos)

- [ ] Abrir `/sales?repairOrderId=...` → verificar banner visible
- [ ] Verificar que los paneles "Agregar producto" y "Servicio manual" NO aparecen
- [ ] Verificar que los inputs de cantidad/precio en carrito son texto (no editables)
- [ ] Verificar que el **descuento % / ₡ es editable** al facturar (no viene de la orden)
- [ ] El POS muestra: Total orden, Pagado (₡0), Saldo pendiente (= Total orden sin descuento previo)
- [ ] Elegir método de pago, guardar → verifica que factura se genera correctamente

### Venta desde orden con abonos parciales

- [ ] Registrar 2 abonos de distintos métodos en la orden
- [ ] Verificar referencia en el abono (código SINPE)
- [ ] Abrir POS → muestra saldo correcto: `TotalOrden - ΣAbonos`
- [ ] Generar factura → impresión muestra: Total orden / Abonos previos / Saldo cobrado hoy
- [ ] Verificar que `TotalProfit` en dashboard = `(TotalOrden - TaxAmount) - TotalCosto` (NO el saldo parcial)

### Orden completamente pagada con abonos (sin descuento al facturar)

- [ ] Abonar el 100% del total de una orden (sin descuento en POS)
- [ ] Abrir POS → saldo ₡0; puede facturarse **sin** métodos de pago (emite factura de cierre)

### Donación / cortesía al facturar (descuento + abonos)

- [ ] Orden ₡35.000, abono ₡5.000, descuento ₡30.000 en POS (modo **₡**) → saldo ₡0
- [ ] POST `/api/sales` **sin 500**; `TotalAmount = 0` en BD
- [ ] Factura: SubTotal Gravado, Descuento Gravado, Abonos previos, Saldo a cobrar

### Métricas de ganancia

- [ ] Orden de ₡100,000 con ₡80,000 de abono previo
- [ ] Costo de ítems: ₡30,000
- [ ] Ganancia esperada: ₡70,000 (no ₡20,000 — el saldo parcial)
- [ ] Verificar en dashboard

### Métodos de pago en factura

- [ ] Imprimir factura con 1 método → se muestra correctamente
- [ ] Imprimir factura con métodos mixtos → todos se listan con sus montos
- [ ] Enviar email → el email muestra los métodos con montos
- [ ] Venta sin métodos de pago → sección de métodos no aparece en impresión

### Validación de acceso

- [ ] Sin token → respuesta 401 en todos los endpoints
- [ ] Token de negocio A → no puede ver datos del negocio B

---

## 14. Segunda ola — descuentos unificados (Mayo 2026)

### Objetivo

Un solo flujo de descuento en **ventas** (POS manual y facturación desde reparación), con metadata para factura/dashboard.

### Backend

| Archivo | Cambio |
|---------|--------|
| `Domain/Enums/SaleDiscountKind.cs` | **NUEVO** — `None`, `Percent`, `FixedAmount` |
| `Domain/Entities/Sale.cs` | `DiscountKind` (byte), `DiscountInputValue` |
| `Application/Common/SaleDiscountCalculator.cs` | **NUEVO** — parseo, monto en ₡, totales con IVA |
| `RegisterSaleUseCase.cs` | Descuento unificado; actualiza email/nombre en `Contact`; **no** asigna navegación `Sale.Contact` (solo `ContactId`) |
| `SaleContactResolution.cs` | `.AsTracking()` al reutilizar contacto por teléfono |
| `RepairOrderContactHelper.cs` | `.AsTracking()`; create/update orden solo `ContactId` |
| `GetSaleByIdUseCase.cs` | Expone `CustomerName`, `CustomerEmail` planos + `Contact` anidado |
| `SalesListItemDto` / `SaleRepository` | Columna descuento en listados |
| Migraciones | `20260526120000`, `20260526130000` + `.Designer.cs` |

### Frontend

| Archivo | Cambio |
|---------|--------|
| `manual-sale-totals.ts` | `computeSaleTotals()` con % / ₡ |
| `sales-manual.*` | Toggle descuento, resumen pagos, **Completar restante** (POS y reparación) |
| `layout/whatsapp-panel.ts` | Panel WhatsApp **recogido por defecto** (`collapsed = true`) |
| `sale-discount-display.ts` | Etiquetas factura «Descuento (10%)» vs «Descuento» |
| `sales-invoice-*`, `sales-email-html.ts` | Línea descuento si monto > 0 |
| `dashboard.*` | Columna descuento en ventas |
| `sales.service.ts` | Envía `discountKind`, `discountValue`; **siempre** envía email/nombre aunque haya `contactId` |
| `sale-invoice-api.mapper.ts` | Lee email/nombre desde `Contact` si faltan en raíz |

### Fórmula de totales (POS y reparación)

```
Subtotal bruto = Σ (cantidad × precio con IVA incluido)
Descuento      = % sobre subtotal  O  monto fijo en ₡ (tope = subtotal)
TotalOrden     = Subtotal bruto − Descuento
TaxAmount      = TotalOrden − (TotalOrden ÷ (1 + rate/100))
Total (cobrar hoy) = TotalOrden − PrepaidAmount   (reparación con abonos)
                   = TotalOrden                     (venta manual)
TotalProfit    = (TotalOrden − TaxAmount) − TotalCost
```

---

## 15. Script SQL manual si falla `database update`

### Opción A — EF Core (preferida)

```powershell
cd MiNegocioCR.Api
dotnet ef database update
```

La API también migra al arrancar (`APPLY_MIGRATIONS_ON_STARTUP=true` por defecto).

### Opción B — Script idempotente completo

Archivo: **`MiNegocioCR.Api/Scripts/apply-pending-migrations.sql`**

```powershell
# Local
psql -U postgres -d MiNegocioCR_Dev -f MiNegocioCR.Api/Scripts/apply-pending-migrations.sql

# Producción Supabase (puerto 5432 directo)
psql "<POSTGRES_CONNECTION_STRING>" -f MiNegocioCR.Api/Scripts/apply-pending-migrations.sql
```

### Opción C — Solo las columnas nuevas de descuentos (mínimo)

Si el resto del schema ya está aplicado y solo falla POST `/api/sales` por columnas faltantes:

```sql
BEGIN;

ALTER TABLE "Sales" ADD COLUMN IF NOT EXISTS "DiscountKind" smallint NOT NULL DEFAULT 0;
ALTER TABLE "Sales" ADD COLUMN IF NOT EXISTS "DiscountInputValue" numeric(18,2) NOT NULL DEFAULT 0;

ALTER TABLE "RepairOrders" DROP COLUMN IF EXISTS "DiscountPercent";

INSERT INTO "__EFMigrationsHistory" ("MigrationId", "ProductVersion")
SELECT '20260526120000_RemoveRepairOrderDiscountPercent', '8.0.8'
WHERE NOT EXISTS (
  SELECT 1 FROM "__EFMigrationsHistory"
  WHERE "MigrationId" = '20260526120000_RemoveRepairOrderDiscountPercent'
);

INSERT INTO "__EFMigrationsHistory" ("MigrationId", "ProductVersion")
SELECT '20260526130000_AddSaleDiscountMetadata', '8.0.8'
WHERE NOT EXISTS (
  SELECT 1 FROM "__EFMigrationsHistory"
  WHERE "MigrationId" = '20260526130000_AddSaleDiscountMetadata'
);

COMMIT;
```

### Verificar después

```powershell
psql "<connection_string>" -f MiNegocioCR.Api/Scripts/verify-schema.sql
```

**Esperado en `Sales`:** `TotalCost`, `TotalProfit`, `TotalOrden`, `PrepaidAmount`, `Discount` (mapea `DiscountAmount`), `DiscountKind`, `DiscountInputValue`, `TotalAmount` (legacy, = `Total`)  
**Esperado:** tabla `SalePaymentMethods` existe  
**Historial:** 4 filas de migraciones mayo 2026 (ver script verify)

---

## 16. Bugs corregidos tras la segunda ola

| Bug | Causa | Fix |
|-----|-------|-----|
| POST `/api/sales` → 500 «entity changes» | Columnas `DiscountKind` / `DiscountInputValue` faltantes en BD | Migraciones + script SQL §15 |
| Mismo 500 con cliente existente (venta) | `QueryTrackingBehavior.NoTracking` global → EF intentaba `INSERT` contacto existente (`PK_Contacts`) | `AsTracking()` en `SaleContactResolution`; venta sin navegación `Contact` |
| Mismo 500 al crear orden (reparación) | Mismo patrón en `RepairOrderContactHelper` + `Contact = contact` en create | `.AsTracking()` + solo `ContactId` en `CreateRepairOrderUseCase` |
| Factura sin email → «No hay correo del cliente» | Email en `Contact` pero API/mapper no lo exponían; ventas reparación no guardaban email del formulario | `GetSaleById` plano + mapper + `ApplyContactDetailsFromRequest` |
| Migraciones huérfanas no aplicadas | Faltaban `.Designer.cs` / `[Migration]` | Designer stubs + `apply-pending-migrations.sql` |
| Factura reparación saldo ₡0 → 500 | EF omitía `TotalAmount` cuando `Total = 0`; Postgres NOT NULL | `ValueGeneratedNever` + sync en `SaveChangesAsync` (API `d15cd40`); test `RegisterSalePostgresIntegrationTests` |
| Abonos > total con descuento al facturar | Validación estricta `prepaid > TotalOrden` | Permitir; `Total = max(0, …)` (API `2cb8ed8`) |

---

## 17. Checklist deploy — base de datos

```
[ ] dotnet test (API) — 154+ tests
[ ] dotnet ef database update (Supabase prod, puerto 5432)
[ ] verify-schema.sql — columnas OK; RepairOrders sin DiscountPercent
[ ] apply-pending-migrations.sql — solo si verify falla
[ ] Redeploy Railway (API)
[ ] Redeploy Vercel (frontend)
[ ] Smoke: venta POS con descuento % y ₡
[ ] Smoke: venta reparación + abonos + descuento → saldo ₡0 (donación)
[ ] Smoke: crear orden con contacto repetido (sin PK_Contacts)
[ ] Smoke: enviar factura por correo
[ ] Smoke: WhatsApp panel recogido → expandir/cerrar
```

Documentación relacionada: [`DEPLOY.md`](DEPLOY.md), [`FIXES_MAYO_2026.md`](FIXES_MAYO_2026.md), [`MiNegocioCR.Api/docs/local-dev-database.md`](MiNegocioCR.Api/docs/local-dev-database.md)

---

## Contacto / Referencia

Este documento cubre el estado del sistema tras la refactorización de **Mayo 2026** (pagos mixtos + descuentos unificados).  
Para cualquier duda sobre el flujo de reparación → facturación:

- **Backend:** `Application/UseCases/Sales/RegisterSaleUseCase.cs`, `Application/Common/SaleDiscountCalculator.cs`
- **Frontend:** `features/sales/pages/sales-manual/sales-manual.ts`
- **Migraciones:** `Migrations/20260522120000_*`, `20260526120000_*`, `20260526130000_*`
- **SQL manual:** `MiNegocioCR.Api/Scripts/apply-pending-migrations.sql`
- **Deploy:** `DEPLOY.md`

*Última actualización: 26 mayo 2026*
