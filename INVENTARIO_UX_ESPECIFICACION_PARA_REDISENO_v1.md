# Inventario — Especificación para rediseño UX (v1)

> **Rol documental (15 octubre 2026): Copia/resumen conservado.** Consultar [MiNegocioCR.Api/docs/INVENTARIO_UX_ESPECIFICACION_PARA_REDISENO_v1.md](./MiNegocioCR.Api/docs/INVENTARIO_UX_ESPECIFICACION_PARA_REDISENO_v1.md). Análisis histórico anterior al rediseño; no describe la UI actual. La evolución se documenta en INVENTARIO_UX_REDISENO_v2.md y notas posteriores.

**Workspace:** `c:\Mi-negociocr`  
**Copia completa en repo API:** [MiNegocioCR.Api/docs/INVENTARIO_UX_ESPECIFICACION_PARA_REDISENO_v1.md](./MiNegocioCR.Api/docs/INVENTARIO_UX_ESPECIFICACION_PARA_REDISENO_v1.md)

---

## Para qué sirve

Documento para pasar a **Chat** con todo el contexto de cómo funciona inventario hoy (modelo, API, UI, problemas) y un brief para diseñar una **interfaz más amigable**.

## Resumen en una línea

El backend ya soporta catálogo → opciones → variantes → stock; la UI actual reparte eso en **6 pestañas engorrosas** y el usuario quiere agregar productos **rápido y simple**.

## Contenido del doc completo

1. Modelo de datos (CatalogItem → CatalogVariant)
2. Flujo actual paso a paso (9+ pasos para un producto con variantes)
3. API disponible (todos los endpoints)
4. Reglas de negocio (stock, SKU, precios CRC ₡5)
5. Integración con Ventas, Reparaciones, Créditos
6. Problemas y fricciones (feedback + código)
7. Casos de uso P1–P3
8. Brief con preguntas y entregables para Chat

---

*Junio 2026*
