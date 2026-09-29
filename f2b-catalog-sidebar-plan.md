# F2-B / F2-B2 — Plan de Discovery, resultados, sidebar y facetas gobernadas

**Estado:** análisis y propuesta para implementación posterior por Cursor.  
**Entregable de esta fase:** exclusivamente este Markdown.  
**Fuera de esta fase:** código productivo, migraciones, cambios de frontend, backend, API pública o configuración de tiendas.

## Decisiones que guían el trabajo

1. El sidebar no es parte permanente del landing. La tienda tiene un **Discovery Mode** limpio y un **Catalog/Results Mode** explícito; solo el segundo contiene filtros.
2. En Discovery Mode, el espacio que hoy ocupa Categorías / Filtros / Ordenar se reutiliza como navegación comercial basada en secciones y datos reales del tenant, nunca en contenido fijo de una tienda de referencia.
3. F2-B debe consumir el contrato visual multi-tenant existente. No puede introducir colores, radios, sombras ni una segunda familia de themes propia.
4. F2-B2 no debe convertir una especificación en filtro simplemente porque tiene dos o más valores. Un atributo necesita autorización comercial explícita por categoría.
5. Las facetas contextuales, los counts y la selección múltiple requieren que la consulta de catálogo, filtrado y paginación pasen a SQL antes de exponer la experiencia completa.
6. Se mantienen intactos F1-D Quick View y F1-E restauración de estado/scroll.

## Restricciones no negociables

- No modificar `product-card-quick-view.directive.ts`, `store-product-quick-view.service.ts`, el panel de Quick View ni su interacción por hover. F2-B conserva las tarjetas `article.product-card` y `data-catalog-listing-key`.
- No cambiar la semántica de `StoreCatalogRestoreService`, `storefront-scroll.util.ts` ni `store-catalog-scroll-restore.util.ts`. El scroll continúa ocurriendo en el host `app-public-store-layout`.
- No incorporar nombres, promociones, destinos ni colores hardcodeados para Joyca Tech ni para la tienda de referencia.
- No duplicar el estado de filtros entre desktop y móvil: ambas superficies consumen una sola fuente de estado.
- No presentar especificaciones administrativas, identificadores o atributos sin autorización como filtros comerciales.

## Auditoría de modos, navegación y layout actuales

La ruta pública actual es única: `/tienda/:slug`. `StoreLandingPage` mezcla landing y catálogo paginado en la misma página.

- `visibleSections()` resuelve `PublicStoreInfo.storefrontSections`.
- `shouldRenderDiscoverySection()` oculta banner, destacados, categorías, nuevos e información cuando la búsqueda tiene texto, pero conserva siempre `catalog`.
- La búsqueda se considera activa solo cuando el texto tras `trim()` no está vacío; su carga se hace con debounce de 350 ms.
- La carga inicial solicita categorías, marcas y catálogo, incluso si la persona solo llegó a descubrir la tienda.
- `selectCategory()` limpia dimensiones/especificaciones dependientes, reinicia la página, sincroniza URL y carga catálogo. Las cards de categorías y los CTA de banner delegan en selección + scroll hacia `#store-catalog-section`.
- La fila actual `catalog-controls-row` contiene categorías, panel expandible de filtros y ordenar; en desktop usa un flex con anchos máximos/relativos y el catálogo usa `80vw` a partir de 1440px. Esa distribución no es una base adecuada para el nuevo sidebar.

El catálogo actual ya preserva F1-E: antes de ir al PDP captura `{ scrollY, listingKey }`; al volver, reaplica estado desde URL, espera que exista la card con el mismo `data-catalog-listing-key` y restaura el ancla. El parser del restore usa el path, no el query string. Por eso una evolución compatible del estado URL puede añadir intención y multi-select sin reescribir el servicio de restauración.

### Alternativas para activar Catalog/Results Mode

| Alternativa | Ventajas | Problemas | Decisión |
| --- | --- | --- | --- |
| A. Inferir solo desde filtros existentes | No agrega parámetro. Los enlaces con consulta ya se ven como resultados. | No representa “explorar todo”; limpiar el último filtro devuelve inesperadamente al landing; la intención no es compartible. | Solo compatibilidad para URLs antiguas. |
| B. Signal local en la landing | Rápido de prototipar. | Se pierde en refresh, deep link, nueva pestaña y back/forward; rompe la equivalencia URL/vista. | No usar. |
| C. `view=results` aditivo en URL | Persistente, compartible, conserva Results Mode al limpiar, compatible con F1-E. No afecta a la API de catálogo. | Requiere ampliar el util de URL y sus pruebas. | **Recomendada.** |
| D. Ruta `/tienda/:slug/catalogo` | Separación semántica máxima. | Amplía rutas, SEO, enlaces y el parser de restore; desproporcionado para F2-B. | Posible evolución posterior, no F2-B. |

### Recomendación de transición

Usar `view=results` como intención explícita. La raíz limpia sigue siendo Discovery Mode; una URL antigua con `q`, `cat`, `brand`, precio, facetas, sort no predeterminado o página distinta de uno debe entrar a Results Mode por compatibilidad.

| Acción | Resultado previsto |
| --- | --- |
| Abrir `/tienda/:slug` sin consulta | Discovery Mode; no cargar ni renderizar listado paginado, filtros, orden ni paginator. |
| Presionar “Explorar catálogo” | Añadir `view=results`, renderizar el shell y cargar el catálogo completo. |
| Escribir una búsqueda no vacía | Entrar o permanecer en Results Mode, persistir `q`, cargar resultados. |
| Elegir categoría desde una card, CTA o barra comercial | Entrar a Results Mode, persistir `cat`, cargar facetas contextuales y hacer scroll cuando el shell ya exista. |
| Limpiar búsqueda o filtros | Mantener `view=results`; solo se eliminan criterios. |
| Presionar “Volver a descubrir” | Eliminar `view` y el estado de consulta de forma explícita. |
| Abrir PDP sin historial de catálogo | Mantener el fallback actual a raíz limpia, es decir Discovery Mode. |

La entrada debe centralizarse en una acción semántica. La secuencia actual `selectCategory(); scrollToCatalog()` no puede conservarse literalmente: después de entrar desde Discovery, primero debe actualizarse la URL/renderizarse el shell y solo después hacerse scroll dentro del host.

## Contrato visual y Theme multi-tenant

### Cadena real de configuración

El theming no está en el catálogo ni en una constante del frontend:

1. El administrador guarda colores, theme y secciones en `BusinessStoreSettings` mediante Store Settings. Los campos relevantes son `BrandColor`, `BrandColorSecondary`, `Theme` y `StorefrontSectionsJson`.
2. `GET /api/public/store/{slug}` ejecuta `GetPublicStoreInfoUseCase`, que toma esos valores y los entrega como `PublicStoreInfoResponse`.
3. `PublicStoreApiService` y `normalizePublicStoreInfo` los convierten a `PublicStoreInfo` (`brandColor`, `brandColorSecondary`, `theme`, `storefrontSections`).
4. `PublicStoreLayout` aplica en el root `.public-store` la clase devuelta por `storeThemeRootClass(theme)` y las variables `--store-brand` y `--store-brand-secondary`. Los valores de respaldo actuales solo actúan cuando el tenant no configuró color.
5. `store-theme-tokens.scss` define la apariencia de `modern`, `elegant` y `minimal` a partir de esos valores; `store-branded-controls.scss` aplica la identidad a controles Material dentro de `.public-store`.

F2-B debe vivir bajo ese root, usar el mismo contrato y respetar las tres variantes. No debe crear un stylesheet de “sidebar moderno”, ni valores de color/radio/sombra por defecto distintos a los tokens existentes.

### Inventario de tokens existente

| Área | Tokens actuales que componen el contrato |
| --- | --- |
| Marca y semántica | `--store-brand`, `--store-brand-secondary`, `--store-accent`, `--store-link`, `--store-price`, `--store-cta-bg`, `--store-cta-bg-hover`, `--store-cta-fg`, `--store-cta-shadow`. |
| Texto | `--store-text`, `--store-body-text`, `--store-heading-text`, `--store-muted`. |
| Superficies y página | `--store-surface`, `--store-surface-solid`, `--store-surface-muted`, `--store-surface-welcome`, `--store-page-background`. |
| Bordes | `--store-border`, `--store-border-subtle`, `--store-border-strong`, `--store-border-footer`. |
| Forma y elevación | `--store-radius-card`, `--store-radius-panel`, `--store-radius-button`, `--store-radius-gallery`, `--store-shadow-card`, `--store-shadow-card-hover`. Elegant/minimal añaden `--store-panel-shadow` y `--store-filters-panel-background`; Modern usa el fallback a `--store-surface`. |
| Header | `--store-header-background`, `--store-header-foreground`, `--store-header-muted`, `--store-header-shadow`, `--store-header-blur`, `--store-header-border`, `--store-header-accent-line`, y la familia `--store-header-search-*` / `--store-header-cart-*`. |
| Chips y ritmo | `--store-chip-background`, `--store-chip-border`, `--store-chip-radius`, `--store-chip-active-background`, `--store-chip-active-border`, `--store-chip-active-weight`, `--store-content-gap`. Elegant/minimal definen además `--store-grid-gap`, `--store-main-padding-y`, `--store-section-spacing`, `--store-card-lift` y `--store-product-body-padding`. |

Las variables de header son para el header, no una paleta genérica que F2-B pueda reutilizar arbitrariamente. El fondo estructural de Modern, por ejemplo, es deliberadamente fijo; no debe copiarse al sidebar ni a un drawer.

### Mapeo obligatorio para las piezas F2-B

| Pieza | Reutilización requerida | Notas de interacción |
| --- | --- | --- |
| Barra comercial de Discovery | `--store-surface` o `--store-surface-solid`, `--store-border-subtle`, `--store-text`, `--store-link`, radios de botón/panel, `--store-content-gap`. | Los destinos activos pueden usar la familia de chips; la acción primaria puede usar `store-btn--filled`, no un color local. |
| Shell, sidebar y grupos de filtros | `--store-surface`, `--store-surface-muted`, `--store-border-subtle`/`strong`, `--store-heading-text`, `--store-text`, `--store-radius-panel`, `--store-panel-shadow` cuando exista. | El sidebar debe sentirse como panel del theme actual, no como una tarjeta importada. |
| Checkboxes, radios, selects y rango de precio | El mixin existente de controles branded para foco/select/radio, `--store-brand` y los tokens de texto/borde/superficie. | Reutilizar el mismo estado de foco de Material; no colorear input, thumb o track con hex locales. |
| Chips de filtros activos | `--store-chip-*`, `--store-brand`, `--store-text`, `--store-radius-button`. | El icono de remover debe conservar contraste, foco visible y estado hover/pressed derivados de brand/surface. |
| Botón móvil “Filtros · n” | Variante `store-btn--stroked` o filled ya existente, con radios y foco actuales. | El contador es contenido, no un nuevo badge con color fijo. |
| Drawer / bottom sheet | `--store-surface-solid`, `--store-border-subtle`, texto/radios/sombras/página del theme. | Debe recibir el contexto de theme aunque Angular lo renderice fuera del root. |
| Toolbar de resultados, breadcrumb y “Volver a descubrir” | `--store-text`, `--store-muted`, `--store-link`, superficie/borde y botones branded. | “Volver” no se convierte en un CTA de una marca concreta. |
| Hover, focus y selected | `--store-brand`, chips activos, CTA hover y los tokens Material existentes. | Foco visible y estado selected deben ser consistentes en los tres themes y no depender solo del color. |

### Gap de theming y propuesta de extensión

Hay dos gaps reales:

1. El contrato actual tiene superficies y chips suficientes para componer el sidebar, pero no una familia semántica completa para controles de faceta (`control surface`, borde en reposo/hover/focus, checkbox) ni para un panel modal. No se debe resolver repitiendo mezclas `color-mix()` por cada componente.
2. Un `MatSelect`, drawer, dialog o bottom sheet que use CDK Overlay se inserta fuera de `.public-store` y no hereda sus custom properties. Quick View ya trata este problema de forma puntual: copia una lista limitada de variables al overlay. No debe modificarse Quick View, pero F2-B no puede asumir que su solución le dará theme al nuevo overlay.

La implementación posterior debe ampliar **el sistema central de tokens**, no crear uno paralelo:

- Definir aliases semánticos de filtro/panel en `store-theme-tokens.scss`, por ejemplo superficie de control, borde de control, borde de foco, superficie hover y backdrop/panel de overlay. Cada theme los deriva de `--store-surface*`, `--store-border*`, `--store-brand`, radios y sombras ya existentes.
- Extender el mixin global del storefront para checkbox/radio/select/input, manteniendo Material como control base.
- Crear una utilidad compartida de propagación de theme a overlays del storefront: copiar la clase `store-theme-*` y el conjunto completo de tokens necesarios desde la instancia correcta de `.public-store` al panel de overlay. Esta utilidad es adicional a Quick View; no se debe acoplar ni alterar su comportamiento.
- Verificar contraste y foco en Modern, Elegant y Minimal antes de aprobar la nueva familia. Los fallbacks deben ser los tokens existentes, no una paleta de F2-B.

## Discovery Mode: navegación comercial reutilizable

En Discovery Mode, la posición de la sección `catalog` se conserva en el orden configurable de `storefrontSections`, pero su contenido pasa a ser el punto de entrada comercial al catálogo. No se renderiza el grid paginado allí.

La implementación futura debe separar un builder puro —por ejemplo, `buildStorefrontCommerceNavigation(...)`— de un componente presentacional. El builder recibe solo secciones resueltas y disponibilidad real de datos; el componente emite intenciones como `scrollToSection`, `enterResults` y `enterCategoryResults`.

| Fuente existente | Destino permitido | Regla |
| --- | --- | --- |
| `categories` visible y categorías públicas disponibles | Ancla de categorías | Omitir si está oculta o vacía. Una card concreta de categoría entra a Results Mode. |
| `featured` visible y respuesta con items | Ancla de destacados | Omitir si la sección no tiene contenido. |
| `nuevos` visible y respuesta con items | Ancla de nuevos | Omitir si no hay items. |
| `businessInfo` visible y contenido/contacto válido | Ancla de información | Omitir si `hasBusinessInfoContent()` es falso. |
| `catalog`, que hoy se fuerza visible | Entrada genérica a Results Mode | Acción primaria “Explorar catálogo”; sin claim comercial inventado. |

Los labels deben salir de `STOREFRONT_SECTION_LABELS` o de copy genérico aprobado. No se agrega una configuración paralela de menú, iconos, enlaces arbitrarios ni campañas. El banner conserva sus CTA configurables: los destinos `catalog` y `category` usan la entrada centralizada a Results Mode; `product` y `externalUrl` continúan como hoy.

En desktop la barra puede ser horizontal y compactarse/permitir desplazamiento sin crear slots vacíos. En móvil/tablet conserva la entrada a catálogo y destinos realmente disponibles; no se convierte en una fila de filtros.

## Catalog/Results Mode: shell, responsive y sticky

### Estructura prevista

Solo en Results Mode, el shell tiene dos áreas semánticas:

- `aside` de filtros: categoría, precio, marca, facetas comerciales, chips/limpieza.
- contenido principal: contexto de resultados, toolbar, chips activos, grid, loading/empty y paginator.

La toolbar sustituye la barra comercial de Discovery; no duplica el contenido del sidebar. En tablet y móvil muestra contexto, ordenar y un único activador “Filtros · n”.

### Breakpoint por capacidad, no por imitación

El sidebar solo puede aparecer si el **contenedor de resultados** permite simultáneamente el sidebar, gap y al menos tres cards de ancho útil:

```text
sidebar mínimo:          14rem  = 224px
gap sidebar/resultados:  1.5rem = 24px
3 tarjetas mínimas:      3 × 13.125rem = 630px
2 gaps del grid:         2 × 1rem = 32px
------------------------------------------------
capacidad mínima:                   910px ≈ 57rem
```

La recomendación es una container query en `57rem` del shell, no un breakpoint global ni un porcentaje de viewport. El grid usa un mínimo de card real, por ejemplo `minmax(13.125rem, 1fr)`. El contenedor de resultados puede limitarse a `93rem`, derivado de sidebar + gap + cinco cards útiles; por debajo de esa capacidad ocupa el espacio disponible con gutters definidos por layout, sin márgenes laterales artificiales.

Por debajo de la capacidad:

- no hay sidebar persistente;
- tablet abre drawer modal y móvil abre bottom sheet accesible;
- ambas variantes reutilizan los mismos controls/signals; no hay formularios divergentes;
- el contenido puede actualizar al seleccionar, como hoy, o aplicar en bloque solo si una decisión UX posterior lo justifica y se prueba de forma uniforme.

### Sticky debajo del header real

`app-public-store-layout` es el scroll host y `.store-header` es sticky. Su altura cambia si el nombre de la tienda envuelve texto. La futura implementación debe medirla con `ResizeObserver` y exponer una custom property de altura; el `aside` usa esa variable como `top` y para su `max-height`.

El sidebar puede tener scroll interno cuando exceda el viewport, pero su ancestro no debe recibir `overflow` o `transform` que invalide `position: sticky`. Debe permanecer por debajo del z-index del header.

## Auditoría del modelo de atributos

### A. `CatalogOption` y `CatalogOptionValue`

| Hallazgo | Evidencia y consecuencia |
| --- | --- |
| Alcance | `CatalogOption` pertenece a **un `CatalogItem`** mediante `CatalogItemId`; no es una definición global, de tenant ni de categoría. `CatalogOptionValue` pertenece a una opción de ese producto. |
| Creación | `CreateOptionUseCase` valida un máximo de tres dimensiones activas por producto y normaliza el nombre con `CatalogDimensionRules`. Acepta dimensiones estándar —Marca, Color, Talla, Capacidad, Presentación, Modelo, Compatibilidad, Material, Sabor y Tamaño— o una personalizada. |
| Valores y reutilización | `CreateOptionValueUseCase` normaliza el valor y hace upsert de `BusinessDimensionValue`. Esta biblioteca es por tenant y sirve para sugerir/reutilizar valores administrativos, pero no convierte `CatalogOption` en una entidad compartida ni define visibilidad pública. |
| Relación con variantes | `CatalogVariantOptionValue` une una variante con exactamente un valor de cada dimensión activa; `CreateVariantUseCase` valida que todos los valores pertenezcan al mismo producto y que no se repita la combinación. Así, Color/Talla son atributos de presentación de variante, no propiedades canónicas de categoría. |
| Metadata actual | Solo nombre, activo y valor/activo. No hay orden comercial, tipo de faceta, prioridad, categoría, visibilidad ni relación con la definición de atributos del storefront. |
| Filtros actuales | La API expone dimensiones/valores desde variantes publicables y en stock. `Parse` los guarda como `Dictionary<string,string>`, por lo que una segunda selección con el mismo nombre sobrescribe la primera: hoy no existe OR/multi-select. |

Los Option/Value son una buena **fuente** para facetas de variante, pero no son el lugar correcto para decidir que “Color” debe aparecer como filtro de una categoría: cada producto crea su propia instancia de la dimensión.

### B. `CatalogItemSpecification`

| Hallazgo | Evidencia y consecuencia |
| --- | --- |
| Estructura | Cada fila contiene `Id`, `CatalogItemId`, `SortOrder`, `Key`, `Value`, `CreatedAt`. El valor de `RAM` se guarda físicamente en cada `CatalogItem`; no hay FK a una definición central. |
| Edición | `CatalogItemEnrichedRepository.ReplaceSpecificationsAsync` borra las filas existentes del producto y las recrea con `Key.Trim()`/`Value.Trim()`. No canonicaliza claves ni evita sinónimos. |
| Creación administrativa | Product Create Wizard recibe sugerencias de IA, permite edición con `ProductSpecificationsEditor` y publica especificaciones en `CreateEnrichedCatalogProductUseCase`. Product Sheet repite el editor y llama `UpdateCatalogItemEnrichedUseCase`, que reemplaza la colección completa. |
| IA y plantillas | Gemini puede sugerir especificaciones. `CategorySpecTemplate` solo aporta `SpecKey` y `SortOrder` como hint para el editor; puede resolverse por categoría, `CategoryKey` derivado del nombre o fallback `default`. No es una taxonomía, no controla filtros y no normaliza el dato guardado. |
| Filtros actuales | `PublicStoreCatalogSpecificationFilters.BuildFacetMap` agrupa por `Key` textual case-insensitive y expone cualquier clave que tenga al menos dos valores distintos. Ese es el origen de EAN, códigos, variantes lingüísticas y campos narrativos como filtros. |
| Metadata actual | No hay `IsFilterable`, tipo, atributo canónico, visibilidad, prioridad, grupo, validación de valores ni configuración por categoría. |

Por lo tanto, hoy `Color`, `Colores`, `Función`, `Funciones` y `Funcionalidad` son claves independientes si llegaron así a distintos productos. Cualquier flag agregado únicamente sobre una fila de specification heredaría esa inconsistencia y se perdería al siguiente reemplazo del producto.

### C. Categorías

`CatalogCategory` es tenant-scoped (`BusinessId`) y está asociada a cada producto mediante `CatalogItem.CategoryId`. La categoría sí es el lugar correcto para expresar relevancia comercial, pero hoy no conoce Options ni Specifications: no hay tabla/metadata que describa atributos, orden o filtros de esa categoría.

`CategorySpecTemplate` es el antecedente arquitectónico más cercano, pero su contrato actual es insuficiente: solo contiene una clave sugerida y orden de formulario. Usarla tal cual para facetas mezclaría “sugerir una ficha rica” con “autorizar un filtro público” y no cubriría source de variante, estado, prioridad, labels ni atributos normalizados.

### D. Caminos de admin, API e importación

| Camino | Genera o modifica | Implicación para F2-B2 |
| --- | --- | --- |
| Product Quick Add / diálogo de presentaciones | Opciones, valores y combinaciones de variantes mediante endpoints de opciones y variantes. | Debe consumir la definición autorizada como sugerencia/validación futura, sin asumir que toda dimensión de un producto se vuelve filtro. |
| CRUD `api/options` y `api/option-values` | Crea, edita, activa/desactiva y elimina Option/Value por producto. | El flag no debe vivir solo aquí porque no define relevancia de categoría. |
| Product Create Wizard + IA | IA propone specs; usuario las corrige; `enriched-product` las persiste. | La incorporación debe canonicalizar o mapear la clave autorizada antes de guardar/publicar, sin bloquear datos informativos no filtrables. |
| Product Sheet / `PATCH /api/catalog/{id}/enriched` | Reemplaza todas las specifications del producto. | Cualquier metadata por fila se perdería; la gobernanza debe residir fuera de esas filas o re-resolverse por clave canónica. |
| `CategorySpecTemplate` / `GET /api/catalog/spec-templates` | Entrega hints de claves a la creación. | Puede alimentar sugerencias de la futura configuración, pero no ser la única fuente de verdad de facetas. |
| Importación existente | El único importador de catálogo encontrado es ZIP de imágenes de variante, enlazadas por SKU. Los flujos de facturas afectan costo/stock; no existe importador CSV/XLSX de productos, Options o Specifications. | No hay importador tabular que migrar ahora, pero cualquier importador futuro debe pasar por el mismo resolver/canonicalizador de atributos. |

## Modelo recomendado de gobernanza

### Alternativas evaluadas

| Alternativa | DB / backend / admin | Migración y consistencia | Escalabilidad / decisión |
| --- | --- | --- | --- |
| A. `IsFilterable` en cada `CatalogItemSpecification` | Cambio pequeño por fila y editor. | Se pierde con `ReplaceSpecificationsAsync`; obliga a marcar miles de filas; no resuelve `RAM` vs `Memoria RAM`, no controla Option/Value ni prioridad de categoría. | Rápida pero incorrecta; **descartar**. |
| B. Definición central normalizada por tenant | Nueva entidad de atributo con clave canónica, display label y normalización; admin para mantenerla. | Reduce sinónimos y da identidad estable, pero por sí sola no dice en qué categoría aparece ni cuántos son primarios. | Útil como fundación, insuficiente aislada. |
| C. Configuración de atributos por categoría | Nueva relación categoría–atributo con enable/orden/source. | Resuelve contexto y UX; si referencia strings libres sin catálogo central, vuelve a introducir variantes ortográficas. | Correcta para relevancia, necesita normalización. |
| D. Ampliar `CategorySpecTemplate` actual | Parecería reutilizar una tabla ya existente. | Sobrecarga un helper de formulario que permite fallback por nombre de categoría; aún faltarían source, estado, tipo, identidad y versionado. | Evitar como modelo principal; podría migrarse/consultarse como fuente de sugerencias. |
| E. Definiciones normalizadas + configuración por categoría + defaults guiados | Dos capas explícitas, con UI administrativa y sugerencias provenientes de datos/plantillas. | Permite backfill gradual y desactiva por defecto atributos dudosos; resuelve specs y dimensiones de variante. | **Recomendada.** |

### Propuesta recomendada

Crear una taxonomía gobernada por tenant, sin hardcodearla en el frontend:

1. **Definición de atributo comercial** a nivel `BusinessId`: identidad estable (`attributeKey`), label público, tipo/source permitido (`itemSpecification` o `variantOption`), reglas de canonicalización de clave/valor y estado. La clave es estable para API/URL; el label puede cambiar sin romper enlaces.
2. **Configuración por categoría** que referencia la definición y contiene `enabled`, prioridad numérica, grupo de visibilidad (`primary`/`secondary`) y, cuando corresponda, label/orden contextual. Solo estas configuraciones autorizan facetas públicas.
3. **Datos de producto existentes** siguen almacenando especificaciones y options donde viven hoy. Un resolver al crear/editar/importar mapea la clave normalizada a una definición. Los datos que no se mapean permanecen visibles en ficha, pero nunca aparecen como faceta.
4. **Defaults guiados, no reglas ocultas:** la administración puede proponer attributes desde `CategorySpecTemplate`, la biblioteca de dimensiones y el perfil de datos de la categoría. El negocio confirma qué es comercial y su orden. Ningún ejemplo como Laptop/Ropa/Accesorios se incorpora como condición fija del cliente.

Esta combinación se adapta al modelo real: las categorías son tenant-scoped, las opciones son por producto, las specifications son libres y el frontend ya tiene un flujo de settings/configuración. No requiere transformar una entidad de variante en un catálogo global artificial.

### Fronteras del modelo

- Marca, categoría y precio son criterios base del catálogo, no `CatalogItemSpecification`; su tratamiento se conserva como first-class en la consulta y UX.
- Una definición comercial puede admitir source de specification, de variante o ambos solo si la semántica fue aprobada. No se deben fusionar automáticamente valores con el mismo texto.
- La configuración de categoría es autorización de presentación; el resultado se sigue calculando únicamente con valores que existen y son publicables.
- No añadir metadata de faceta directamente a todas las filas actuales ni depender de nombres de categoría para determinar comportamiento público.

## Reglas de elegibilidad de filtros

La respuesta de facetas debe derivar de una consulta contextual y respetar, en orden, estas reglas:

1. **Base siempre visible cuando tenga sentido:** categoría, precio y marca. Categoría se ofrece si hay más de una categoría disponible en el contexto; precio si existe un rango real; marca si al menos dos marcas válidas son elegibles.
2. **Atributo autorizado:** un Option o Specification solo puede ser facet si tiene definición comercial activa y configuración `enabled` para la categoría/contexto aplicable.
3. **Mínimo de alternativas útiles:** ocultar una faceta autorizada si, tras aplicar el contexto, no tiene al menos dos valores seleccionables relevantes. No se aplica la regla de “dos valores” a una specification sin autorización.
4. **Publicabilidad:** considerar solo productos/variantes que la tienda ya considera listables y en stock. No mostrar una opción para una variante inactiva, sin stock o no pública.
5. **Sin callejones sin salida:** un valor no seleccionado se muestra habilitado solo si al aplicarlo junto a los criterios de las otras dimensiones produce resultados. Los valores seleccionados permanecen visibles para que puedan quitarse, aunque un cambio concurrente los deje sin resultado.
6. **No confundir fuente con UX:** especificaciones narrativas, códigos, EAN/barcode, SKUs, características libres y campos de una sola alternativa quedan fuera salvo configuración explícita y revisión comercial.

## Prioridad/orden

La interfaz no debe terminar con veinte filtros abiertos. El orden se conserva en servidor/configuración, no se infiere en Angular por nombre.

1. Categoría.
2. Precio contextual.
3. Marca contextual.
4. Entre dos y cuatro atributos `primary` con mayor prioridad que sean elegibles en el contexto actual.
5. Los atributos autorizados restantes se agrupan en “Más filtros” o secciones secundarias colapsables.

El número 2–4 es una política de presentación inicial, no un máximo técnico: una faceta útil no deja de existir porque exceda ese espacio. Al abrir “Más filtros” se conservan selección, count y orden de configuración. La configuración por categoría admite prioridades sin que el frontend tenga un mapa de tenants o categorías.

## Facetas contextuales

### Contexto de cálculo

El endpoint actual `GET /catalog/filters` acepta solamente `categoryId`, carga todos los productos/variantes de la categoría y devuelve dimensiones sin counts. F2-B2 necesita reemplazarlo por un contrato contextual que reciba el mismo estado de Results Mode: búsqueda, categoría, marcas, precio, option facets y specification facets.

Para cada dimensión **D**, el servidor debe construir el universo aplicando todos los criterios activos **excepto D**, agrupar sus valores y devolver counts de resultados publicables. Así:

- los valores de la misma dimensión compiten como alternativas OR;
- otros criterios siguen restringiendo la lista;
- no se ofrece un valor que lleve a cero;
- cambiar búsqueda, categoría, marca, precio u otra faceta recalcula todas las opciones relevantes.

Los counts deben usar la misma unidad que el grid. Como el storefront actual muestra listings de variantes, el diseño de consulta debe definir y utilizar un identificador de listing estable, evitando contar dos veces por joins de specifications/options. Las specifications del producto se aplican a sus listings de variante; las OptionValues se aplican a la propia variante.

### Contrato de respuesta futuro

El plan no prescribe una clase de código, pero el contrato debe incluir como mínimo:

- contexto normalizado/eco de selección para detectar respuestas obsoletas;
- categorías disponibles con count cuando aplique;
- rango real de precio (`min`, `max`, y opcionalmente intervalos solo si se justifican);
- marcas con label, clave estable y count;
- facetas con `attributeKey` estable, label, source, prioridad/grupo y valores `{ valueKey, label, count, selected, enabled }`;
- información suficiente para que el cliente no reconstruya elegibilidad por su cuenta.

El cliente conserva una sola representación de estado, renderiza desktop/móvil desde ella, descarta respuestas de facetas que no correspondan al contexto más reciente y no calcula counts sobre la página visible.

## Semántica AND/OR

La semántica objetivo es:

```text
Marca: Lenovo
RAM: 16 GB
Color: Negro o Azul

=> Lenovo AND 16 GB AND (Negro OR Azul)
```

- Dentro de la misma dimensión, los valores son **OR**.
- Entre dimensiones distintas, los filtros son **AND**.
- Para una faceta de variante, todas las dimensiones AND deben satisfacer la **misma variante/listing**; no se puede combinar Color de una variante con Capacidad de otra del mismo producto.
- Para specifications de item, todas las specifications AND deben existir en el mismo item y luego aplicar a sus variantes listables.
- Marca múltiple, si se habilita, sigue la misma regla OR dentro de Marca. Categoría se mantiene de selección única en esta fase salvo que una decisión futura amplíe ese alcance.

La implementación actual no cumple esto: sus parsers almacenan un solo string por nombre, de modo que el último valor repetido sobrescribe el anterior. El nuevo contrato/query debe representar conjuntos de valores y el backend debe traducirlos a `EXISTS`/`IN` o equivalentes SQL, no a filtros posteriores sobre listas materializadas.

## Rango contextual de precio

El precio no puede ser dos campos arbitrarios sin saber el universo disponible. Debe calcularse sobre listings publicables que satisfacen el contexto **sin el propio criterio de precio** —categoría, búsqueda, marca y facetas sí aplican— para obtener mínimo y máximo reales.

- El UI inicializa/ajusta controles al rango recibido, sin sobreescribir una selección válida del usuario durante una respuesta asincrónica.
- El rango seleccionado se valida contra `min <= max` y contra el universo contextual.
- Si no existe rango útil, se oculta el control; si todos los resultados comparten precio, se muestra de manera no engañosa o se omite según decisión UX.
- La fuente debe ser el precio de venta de la listing/variante que el grid expone, no `BasePrice` de item cuando no coincide con sus variantes.

## Estado URL multi-select

El `StoreCatalogUrlState` actual persiste `q`, `cat`, `brand`, `sort`, `page`, `ps`, `pmin`, `pmax`, `of` y `sf`, pero `brand`, `of` y `sf` contienen solo una selección por dimensión. F2-B2 debe evolucionarlo de forma compatible:

- Mantener `view=results` como intención de presentación.
- Convertir la representación interna de brand y facetas a conjuntos ordenados/deduplicados, por ejemplo `Record<attributeKey, valueKey[]>` para options/specifications y `brands: string[]` si se habilita multi-marca.
- Conservar lectura de enlaces existentes: `brand=Lenovo` se interpreta como una selección; el formato legado de `of`/`sf` con un string se interpreta como una lista de un elemento.
- Emitir una forma canónica estable. Una opción recomendada es repetir `brand` para marcas y serializar `of`/`sf` como JSON de arrays de claves estables; la decisión exacta se confirma junto con el contrato del endpoint, no con labels visibles.
- Ordenar claves y valores antes de serializar y comparar, para evitar recargas o entradas de historial distintas por el orden de clic.
- Resetear `page` a uno ante cualquier cambio de criterios; limpiar criterios mantiene `view=results`.

F1-E permanece compatible porque el restore ya persiste scroll/listingKey y el historial conserva query params. Las pruebas deben comprobar round-trip de legacy single-select, multi-select, entrada PDP y back/forward con `view=results`.

## Impacto administrativo

La futura administración necesita una superficie de configuración de atributos por categoría, no una lista de flags por producto:

- Seleccionar o crear una definición comercial del tenant, con label y source permitida.
- Asociarla a una categoría con habilitado/deshabilitado, prioridad y grupo primary/secondary.
- Mostrar métricas de cobertura y valores detectados antes de publicar una faceta; advertir sinónimos, valores vacíos y baja cobertura.
- Usar `CategorySpecTemplate` y la biblioteca `BusinessDimensionValue` solo como sugerencias para acelerar la selección; no autoactivar facetas por ellas.
- Al editar una ficha, sugerir claves canónicas y detectar equivalencias aprobadas, pero permitir specifications informativas fuera de la taxonomía.
- Al crear Options/Values o importar productos en el futuro, resolver contra la misma definición/normalizador. La importación ZIP de imágenes actual no requiere modificación funcional.

La autorización de faceta debe tener permisos y auditoría acordes a Store Settings. El storefront público solo recibe definiciones ya publicadas/activas y datos faceteables, no herramientas administrativas ni metadata interna innecesaria.

## Migración/backfill

No se crea migración en esta fase. La secuencia que Cursor debe planificar antes de activar F2-B2 es:

1. Inventario de datos por tenant/categoría: claves de specification normalizadas, valores, cobertura, cardinalidad, sinónimos y posibles identificadores/códigos.
2. Crear taxonomía/configuración vacía o con atributos explícitamente aprobados; **nada queda filterable por defecto**.
3. Proponer mappings de claves existentes a `attributeKey` mediante revisión administrativa. Separar claves ambiguas en lugar de adivinar equivalencias.
4. Backfill gradual de una clave canónica o índice de búsqueda para las specifications aprobadas; conservar texto original donde sea necesario para la ficha.
5. Validar con una categoría/tenant piloto: counts, URL, resultados y administración.
6. Habilitar públicamente solo categorías con cobertura/calidad suficiente; mantener fallback sin faceta para el resto.
7. Establecer el resolver en todos los flujos de creación/edición/importación futuros para no reintroducir variantes textuales.

El backfill debe ser idempotente, reversible y medible. No se deben eliminar specifications existentes para “limpiar” datos ni modificar `CategorySpecTemplate` masivamente sin revisión de negocio.

## Performance/escalabilidad

### Estado actual basado en código

`GetPublicStoreCatalogUseCase` empieza con filtros SQL básicos de tenant, activo, producto, categoría, marca y búsqueda, pero luego hace `Include` de categorías, imágenes, specifications, variantes y OptionValues, materializa todo con `ToListAsync`, filtra specifications/options en memoria, construye listings, aplica precio/orden en memoria y pagina al final.

`GetPublicStoreCatalogFiltersUseCase` recibe solo categoría, materializa todos los items/variantes/specifications de ella y construye facetas en memoria. `GetPublicStoreBrandsUseCase` y categorías también materializan colecciones para calcular disponibilidad. Las facetas no devuelven counts ni conocen búsqueda, marca, precio o filtros activos.

Los índices declarados cubren parte del acceso base (`CatalogItem` por negocio/categoría, negocio/activo y negocio/marca; variantes por item), pero las specifications solo tienen índice `(CatalogItemId, SortOrder)`. No hay índice compuesto diseñado para búsquedas por `Key`/`Value` de specification ni para la dirección inversa de las uniones de OptionValue necesaria para faceting. El diseño final de índices debe partir del plan de consulta SQL real.

### Riesgo por escala, sin inventar benchmarks

| Escenario | Riesgo actual | Efecto adicional de F2-B2 |
| --- | --- | --- |
| ~1.000 productos con pocas variantes | Ya se descargan y procesan colecciones completas para cada carga/faceta. Puede parecer aceptable en datos pequeños, pero el costo crece linealmente y se repite al cambiar filtros. | Counts contextuales multiplicarían joins/agregaciones en memoria y transferirían más datos de lo necesario. |
| ~10.000 productos | `ToListAsync` antes de paginar carga objetos, relaciones e imágenes de todo el universo; memoria, latencia y GC dejan de estar acotados por page size. | Recalcular facetas por búsqueda/click sería costoso y no fiable para UX reactiva. |
| Muchos productos con múltiples variantes | El número de listings y enlaces OptionValue crece más rápido que el número de items; las includes producen un grafo mucho mayor y hacen fácil contar duplicados. | AND entre dimensiones, OR por valores y counts requieren controlar explícitamente joins, `EXISTS` y `COUNT(DISTINCT listing)`. |

### Decisión de performance

**Sí: antes de publicar facetas contextuales F2-B2 se debe mover filtrado de options/specifications, rango de precio, orden y paginación a SQL.** No es necesario esperar a tener millones de productos; el código actual ya rompe la propiedad esencial de paginar antes de materializar.

La implementación debe partir de una única definición de universo publicable y traducir a SQL:

- filtro base tenant/activo/tipo/stock;
- búsqueda, categoría, marcas y precio;
- `EXISTS` por specification y dimensión de variante, con OR dentro de valores y AND entre dimensiones;
- proyección mínima del page de listings, con carga de imágenes solo para sus IDs;
- query de facetas separada que reutiliza el contexto y excluye la dimensión propia;
- aggregates de min/max y `COUNT(DISTINCT listing)`;
- índices compuestos/funcionales solo después de revisar el SQL generado y `EXPLAIN ANALYZE` sobre datos representativos.

No añadir cache global como sustituto de una consulta correcta. Una cache por tenant/contexto puede evaluarse después, con invalidación al publicar, cambiar stock/precio, categoría, atributos o configuración de facetas.

## Plan incremental de implementación para Cursor

El orden reduce regresiones y evita montar UI sobre un contrato de facetas insuficiente.

1. **Caracterizar y fijar regresiones actuales.** Añadir/validar pruebas de Search Mode, categorías, CTA, URL y F1-D/F1-E antes de cambiar estructura visual.
2. **Contrato visual transversal.** Inventariar componentes F2-B, añadir aliases centrales de tokens y una solución compartida para overlays themed. Validar Modern/Elegant/Minimal sin tocar Quick View.
3. **Separar modos y URL sin F2-B2.** Implementar `view=results`, compatibilidad de enlaces existentes, entrada/salida centralizada y barra comercial derivada de `storefrontSections`. En esta etapa se puede conservar temporalmente el filtro simple existente, pero no debe cargar catálogo en Discovery.
4. **Shell responsive.** Reemplazar `catalog-controls-row` por toolbar + results shell; sidebar por container query, sticky con altura medida de header y drawer/bottom sheet en capacidad insuficiente. Mantener las cards y anclas de F1-E sin cambios.
5. **Modelo y administración de gobernanza.** Diseñar/aprobar esquema de definiciones y configuración por categoría; implementar CRUD/admin, normalizador/resolver y permisos. No activar automáticamente datos existentes.
6. **Migración/backfill gobernado.** Ejecutar inventario, mappings revisados, backfill e indicadores de cobertura por tenant/categoría; activar un piloto.
7. **Consulta SQL de catálogo y facetas.** Introducir multi-select, paginación SQL, counts, rango y marca contextuales, contrato estable de IDs y URL compatible. Medir SQL real antes de índices/caching.
8. **Conectar UI F2-B2-D (Results + sidebar).** Sidebar/drawer con prioridades, “Más filtros”, chips removibles y estados contextualizados. El cliente nunca calcula counts ni autorización de atributos.

### Deuda visual obligatoria F2-B2-D (registrada sep 2026, QA F2-B1)

No parchear aislado en F2-B2-A/B/C. En **F2-B2-D** todos los controles nuevos o reconstruidos de Results deben usar el lenguaje visual del tenant:

- `BrandColor`, `BrandColorSecondary`, `Theme`, variantes **Modern / Elegant / Minimal**
- tokens semánticos `--store-*` (superficie, borde, texto, chips, CTA, foco)
- estados hover / focus / selected / disabled con contraste accesible
- overlays (drawer, bottom sheet, selects) con propagación de theme al CDK (util compartida; **no** modificar F1-D Quick View)

Incluye explícitamente alinear (hoy parcialmente desalineados en QA): **“Volver a descubrir”**, **“Limpiar filtros”**, ordenar, chips activos, inputs/checkboxes/radios, rango de precio, botón móvil “Filtros”. Sin colores hardcodeados de tiendas de referencia.

9. **Rollout y observabilidad.** Telemetría de facetas sin resultados, latencia, coverage y errores de parsing; feature flag por tenant/categoría para reversión segura.

## Tests requeridos

### Modos, navegación y layout

- Raíz limpia en Discovery sin catálogo paginado, filtros, sort ni paginator.
- `view=results`, búsqueda, categorías, CTA de banner y deep link legacy entran a Results Mode.
- Limpiar filtros/búsqueda conserva Results Mode; “Volver a descubrir” lo abandona explícitamente.
- Barra comercial omite secciones ocultas/vacías, respeta orden configurado y nunca contiene destinos ficticios.
- Transición categoría/CTA espera shell antes de scroll.
- 375px, 768px, justo debajo/encima de `57rem` de contenedor, 1024px, 1280px, 1440px y 1920px; sin márgenes laterales excesivos ni compresión simultánea de sidebar/grid.
- Header con nombre largo y sidebar sticky; scroll interno de aside sin romper el scroll host.

### Theme multi-tenant y accesibilidad

- Snapshot/visual checks de Discovery, Results, sidebar, toolbar, chips y drawer para Modern, Elegant y Minimal con colores de tenant distintos.
- Ningún selector F2-B usa hex/radio/sombra de una marca concreta fuera de los fallbacks del sistema central.
- Selects y overlay de drawer/bottom sheet heredan theme y variables del tenant correcto; dos tiendas abiertas en secuencia no filtran estilos entre sí.
- Foco visible, teclado, Escape, focus trap/retorno del drawer, labels/landmarks de navegación vs filtros y anuncio accesible de filtros activos.

### Gobernanza y facetas

- Una specification con dos valores no aparece si no está autorizada; EAN, barcode, SKU, texto libre y sinónimos no aprobados quedan fuera.
- Una configuration de categoría habilita solo source/attribute permitidos y conserva prioridad/grupo.
- Cambiar categoría, búsqueda, marca, precio y otra faceta recalcula facetas contextuales.
- Valores no seleccionados de count cero no son seleccionables; valores seleccionados siguen removibles.
- Orden: categoría/precio/marca, luego 2–4 primarios elegibles y resto en “Más filtros”.
- Semántica: `Negro OR Azul`, y AND con marca/RAM; dimensiones de variante se satisfacen en la misma variante; specifications en el mismo item.
- Rango mínimo/máximo y marcas corresponden al contexto, no a toda la tienda.

### URL, F1-D, F1-E y performance

- Round-trip de URL con `view=results`, filtros legacy single-select y nuevos arrays multi-select; serialización canónica estable.
- Back/forward y retorno desde PDP restauran modo, query, scroll host y card ancla.
- Quick View conserva hover en puntero fino y sigue deshabilitado para touch; no se alteran sus tests ni estilos.
- Tests de integración verifican que filtering, aggregate de facetas y paginación ocurren en SQL antes de materializar la página; casos de múltiples variantes no duplican counts.
- `EXPLAIN ANALYZE` sobre conjuntos representativos de 1.000 y 10.000 productos/múltiples variantes antes de aprobar índices o cache; no usar benchmarks inventados.

## Archivos/áreas que Cursor deberá revisar en la fase de implementación

| Área | Responsabilidad futura |
| --- | --- |
| `store-landing.*` | Separar modes, centralizar transiciones, barra comercial, results shell y conservar cards/F1-E. |
| `store-catalog-url-state.util.ts` | `view=results`, multi-select, legacy parser, serialización y equality. |
| `public-store-layout.*`, `styles/_store-theme-tokens.scss`, `styles/_store-branded-controls.scss` | Medición de header, tokens semánticos y propagación de theme a overlays. |
| `store-front-sections.util.ts` y nuevo builder/componente de navegación | View-model multi-tenant sin menú paralelo. |
| `PublicStoreController`, catálogo/filtros públicos y use cases | Contrato contextual, consulta SQL, paginación, counts y range. |
| Entidades/configuración de catálogo y Store Settings/admin | Definiciones de atributos, configuración por categoría, resolver y backfill. |
| Tests de landing, URL, quick view, restore, API/integración y performance | Contratos descritos arriba. |

Este documento no autoriza implementar ninguna de esas piezas todavía.
