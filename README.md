# MiNegocioCR — Documentación del workspace

Monorepo local: **API** (`MiNegocioCR.Api/`) + **Frontend** (`mi-negociocr-frontend/`) + **App** (`mi-negociocr-app/`).  
Cada carpeta de producto tiene su **propio repositorio Git** (API → `master`, frontend → `main`, app → `feature/mobile-app-fase-a`).

Este directorio raíz también tiene **su propio Git** (meta-workspace): versiona `.ai/`, `.cursor/`, `.codex/`, `AGENTS.md` y documentación compartida en la raíz. **No** incluye el código de los tres repos de producto (están en `.gitignore`).

| Repo GitHub | Qué contiene |
|-------------|----------------|
| [MiNegocioCR.Api](https://github.com/azariasmurillo/MiNegocioCR.Api) | Backend .NET |
| [mi-negociocr-frontend](https://github.com/azariasmurillo/mi-negociocr-frontend) | Angular web |
| [mi-negociocr-app](https://github.com/azariasmurillo/mi-negociocr-app) | Flutter (si aplica) |
| [Mi-negociocr](https://github.com/azariasmurillo/Mi-negociocr) | Instrucciones IA, reglas Cursor, docs raíz |

| Proyecto | README |
|----------|--------|
| API (.NET 8) | [MiNegocioCR.Api/README.md](./MiNegocioCR.Api/README.md) |
| Frontend (Angular 21) | [mi-negociocr-frontend/README.md](./mi-negociocr-frontend/README.md) |
| App Android (Flutter) | [mi-negociocr-app/README.md](./mi-negociocr-app/README.md) |

---

## 🛒 Marketplace / Tienda digital (sep 2026)

| Documento | Uso |
|-----------|-----|
| **[MiNegocioCR.Api/docs/DEPLOY.md](./MiNegocioCR.Api/docs/DEPLOY.md)** | **Deploy prod** — SQL `docs/scripts/marketplace-prod-*.sql`, smoke test |
| [MARKETPLACE_INICIO_JUNIO_2026.md](./MiNegocioCR.Api/docs/MARKETPLACE_INICIO_JUNIO_2026.md) | Plan M1–M9, checklist |
| [mi-negociocr-frontend/docs/TIENDA_PUBLICA_DEPLOY_SEP_2026.md](./mi-negociocr-frontend/docs/TIENDA_PUBLICA_DEPLOY_SEP_2026.md) | Resumen FE + tests |
| [TIENDA_DIGITAL_DISENO_UNIFICADO.md](./MiNegocioCR.Api/docs/TIENDA_DIGITAL_DISENO_UNIFICADO.md) | Spec oficial (`/tienda/{slug}`, carrito, `StoreOrder`) |

---

## Documentos principales (raíz `Mi-negociocr/`)

| Documento | Para qué sirve |
|-----------|----------------|
| [DEPLOY.md](./DEPLOY.md) | Guía completa de deploy (Railway + Vercel + Supabase) |
| [MiNegocioCR.Api/docs/OBSERVABILIDAD_Y_MANEJO_ERRORES_v1.md](./MiNegocioCR.Api/docs/OBSERVABILIDAD_Y_MANEJO_ERRORES_v1.md) | Observabilidad — Sentry web/API/mobile |
| [mi-negociocr-frontend/docs/CAMBIOS_SENTRY_WEB_SEP_2026.md](./mi-negociocr-frontend/docs/CAMBIOS_SENTRY_WEB_SEP_2026.md) | **Sep 2026** — Sentry web Angular |
| [FIXES_MAYO_2026.md](./FIXES_MAYO_2026.md) | Changelog vivo de correcciones (bugs, UI, BD) |
| [REFACTOR_REPAIR_PAYMENTS_SALES.md](./REFACTOR_REPAIR_PAYMENTS_SALES.md) | Modelo financiero: reparaciones, abonos, ventas, descuentos |
| [SETUP_LOCAL_Y_CAMBIOS_MAYO_2026.md](./SETUP_LOCAL_Y_CAMBIOS_MAYO_2026.md) | PostgreSQL local, Resend, tenant JoyCaTech |
| [MiNegocioCR.Api/docs/CAMBIOS_APP_MOVIL_1.1.6_AGO_2026.md](./MiNegocioCR.Api/docs/CAMBIOS_APP_MOVIL_1.1.6_AGO_2026.md) | **App móvil 1.1.6+9 publicada** — UI, navegación, teclado, pagos, anulación e identidad visual |
| [mi-negociocr-frontend/docs/CAMBIOS_MARCA_Y_CABECERA_AGO_2026.md](./mi-negociocr-frontend/docs/CAMBIOS_MARCA_Y_CABECERA_AGO_2026.md) | **Web (ago 2026)** — marca clara, cabecera por módulo y retorno tras login |
| [MiNegocioCR.Api/docs/CAMBIOS_APP_MOVIL_1.1.5_AGO_2026.md](./MiNegocioCR.Api/docs/CAMBIOS_APP_MOVIL_1.1.5_AGO_2026.md) | **App móvil 1.1.5** — cobrar reparación, POS, APK |
| [MiNegocioCR.Api/docs/PUBLICAR_APP_ANDROID_APK_AGO_2026.md](./MiNegocioCR.Api/docs/PUBLICAR_APP_ANDROID_APK_AGO_2026.md) | Runbook publicar APK |
| [MiNegocioCR.Api/docs/GASTOS_CONTROL_MONETARIO_v1.md](./MiNegocioCR.Api/docs/GASTOS_CONTROL_MONETARIO_v1.md) | **Gastos** — control monetario v1 (ago 2026) |
| [mi-negociocr-frontend/docs/CAMBIOS_INVENTARIO_STOCK_CRITICO_SEP_2026.md](./mi-negociocr-frontend/docs/CAMBIOS_INVENTARIO_STOCK_CRITICO_SEP_2026.md) | **Inventario** — atajo Agotados y última unidad (`stockMax = 1`, sep 2026) |
| [mi-negociocr-frontend/docs/CAMBIOS_INVENTARIO_LISTA_CONTEO_AGO_2026.md](./mi-negociocr-frontend/docs/CAMBIOS_INVENTARIO_LISTA_CONTEO_AGO_2026.md) | **Inventario** — lista conteo físico PDF/CSV (ago 2026) |
| [CAMBIOS_INVENTARIO_JUNIO_2026_DIMENSIONES_Y_PRECIO.md](./CAMBIOS_INVENTARIO_JUNIO_2026_DIMENSIONES_Y_PRECIO.md) | **Inventario** — dimensiones, quick-add, deploy jun 2026 |
| [INVENTARIO_UX_REDISENO_v2.md](./INVENTARIO_UX_REDISENO_v2.md) | Inventario UX — diseño ✅ producción |
| [CAMBIOS_CREDITOS_JUNIO_2026.md](./CAMBIOS_CREDITOS_JUNIO_2026.md) | Créditos — deploy |
| [MiNegocioCR.Api/docs/CAMBIOS_CREDITOS_MULTI_CUENTA_AGO_2026.md](./MiNegocioCR.Api/docs/CAMBIOS_CREDITOS_MULTI_CUENTA_AGO_2026.md) | **Créditos v1.2** — multi-cuenta, UX cliente (ago 2026) |
| [MiNegocioCR.Api/docs/CAMBIOS_UI_NAV_Y_ANULACION_AGO_2026.md](./MiNegocioCR.Api/docs/CAMBIOS_UI_NAV_Y_ANULACION_AGO_2026.md) | **Ago 2026** — listados UX, anulación/saldo, menú por módulos |
| [CAMBIOS_LAYOUT_RESPONSIVE_JUNIO_2026.md](./CAMBIOS_LAYOUT_RESPONSIVE_JUNIO_2026.md) | Layout responsive |

Índice completo en API: [MiNegocioCR.Api/docs/WORKSPACE_INDEX.md](./MiNegocioCR.Api/docs/WORKSPACE_INDEX.md)

---

### Módulos del producto

| Módulo | Estado | Spec |
|--------|--------|------|
| Inventario, ventas, reparaciones, CRM, dashboard | Producción | [REFACTOR_REPAIR_PAYMENTS_SALES.md](./REFACTOR_REPAIR_PAYMENTS_SALES.md) |
| **App Android** | **1.1.7+10 preparada para la próxima publicación**; último APK `1.1.6+9` | [CAMBIOS_APP_MOVIL_1.1.6_AGO_2026.md](./MiNegocioCR.Api/docs/CAMBIOS_APP_MOVIL_1.1.6_AGO_2026.md) · [CAMBIOS_APP_MOVIL_1.1.5_AGO_2026.md](./MiNegocioCR.Api/docs/CAMBIOS_APP_MOVIL_1.1.5_AGO_2026.md) |
| Pedidos Internet | Producción + **v2 sep 2026** | [PEDIDOS_INTERNET_DISENO_v1.md](./MiNegocioCR.Api/docs/PEDIDOS_INTERNET_DISENO_v1.md) · [CAMBIOS_PEDIDOS_INTERNET_V2_SEP_2026.md](./MiNegocioCR.Api/docs/CAMBIOS_PEDIDOS_INTERNET_V2_SEP_2026.md) |
| Créditos | Producción (**v1.2** multi-cuenta) | [CAMBIOS_CREDITOS_MULTI_CUENTA_AGO_2026.md](./MiNegocioCR.Api/docs/CAMBIOS_CREDITOS_MULTI_CUENTA_AGO_2026.md) |
| Inventario UX | **Producción** | [INVENTARIO_UX_REDISENO_v2.md](./INVENTARIO_UX_REDISENO_v2.md) |
| Gastos | Producción | [GASTOS_CONTROL_MONETARIO_v1.md](./MiNegocioCR.Api/docs/GASTOS_CONTROL_MONETARIO_v1.md) |
| **Tienda digital / Marketplace** | **En prod** (código `master`/`main`; SQL manual 1ª vez) | [DEPLOY.md](./MiNegocioCR.Api/docs/DEPLOY.md) |

---

## Comandos rápidos pre-deploy

```powershell
cd MiNegocioCR.Api
dotnet test

cd ../mi-negociocr-frontend
npm run build
```

---

*Última actualización: 29 septiembre 2026 — meta-repo workspace; inventario atajo Agotados y última unidad (FE).*
