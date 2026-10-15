# Mi-NegocioCR: instrucciones de IA

Esta carpeta es la fuente neutral de verdad para las instrucciones de trabajo asistido por IA del workspace. No pertenece a un proveedor concreto: un adaptador de Cursor, Codex u otro agente puede referenciarla sin duplicar sus reglas.

## Orden de lectura

1. `PRINCIPLES.md`: reglas de autoridad, evidencia y límites.
2. `WORKFLOW.md`: cómo investigar, decidir, cambiar y verificar.
3. `PROJECT.md`: alcance, productos y exclusiones del workspace.
4. `ARCHITECTURE.md`, `TENANCY.md` y `SAFETY.md`: restricciones transversales.
5. `instructions/<área>.md`: reglas específicas de API, frontend o móvil.

Cuando una instrucción neutral y una integración discreta de una herramienta parezcan diferir, debe conservarse la intención neutral y documentarse la adaptación; no se debe convertir la integración en la fuente de arquitectura.

## Autoridad de la información

La prioridad es: código fuente actual y configuración ejecutable > documentación vigente que el código confirme > mapa de Graphify > inferencia explícita. Graphify sirve para localizar relaciones y candidatos; cada hallazgo importante se verifica en los archivos reales antes de afirmarlo.

## Contexto histórico de creación

La fase original creó esta fuente neutral y sus instrucciones de área. Después se añadieron los Skills neutrales audit-first e implement-from-audit y adaptadores en .codex/ y .cursor/. Esa restricción describía la creación inicial; el punto de entrada compartido sigue siendo este archivo.


## Navegación vigente por tema (consolidación 15 octubre 2026)

Este mapa selecciona documentos por su contenido y por verificaciones del source; no por su ubicación. Una referencia canónica lo es solo para el alcance indicado. Código y configuración ejecutable conservan prioridad. No sincronizar copias por reemplazo completo.

- **Arquitectura general:** [ARCHITECTURE.md](./ARCHITECTURE.md), contrastado con Program.cs, app.routes.ts y los árboles reales. Los diseños antiguos explican decisiones, no imponen capas distintas al source.
- **Frontend:** [instrucciones de área](./instructions/frontend.md) y [README frontend](../mi-negociocr-frontend/README.md) para arranque/rutas. Los documentos completos de UI pueden vivir en frontend; ver familias más abajo.
- **Backend:** [instrucciones de área](./instructions/api.md) y [README API](../MiNegocioCR.Api/README.md); Program.cs e IAppDbContext son referencias ejecutables para DI y persistencia.
- **Multi-tenancy:** [TENANCY.md](./TENANCY.md). AuthHelper valida contratos autenticados; PublicStoreResolver resuelve el catálogo por slug sin perder el filtro por negocio.
- **Seguridad y autorización de operaciones:** [SAFETY.md](./SAFETY.md). [Seguridad multi-tenant Fase 5](../MiNegocioCR.Api/docs/SEGURIDAD_MULTI_TENANT_FASE5_AGO_2026.md) conserva evidencia y límites de aquella revisión; no es una certificación completa actual.
- **Marketplace:** [diseño unificado](../MiNegocioCR.Api/docs/TIENDA_DIGITAL_DISENO_UNIFICADO.md) como decisiones base; [estado de implementación](../MiNegocioCR.Api/docs/MARKETPLACE_INICIO_JUNIO_2026.md), [búsqueda pública](../MiNegocioCR.Api/docs/PUBLIC_STORE_SEARCH.md) para ese motor y [fases/frontend y Lightbox](../mi-negociocr-frontend/docs/TIENDA_PUBLICA_DEPLOY_SEP_2026.md). Contrastar con PublicStoreController y app.routes.ts. No inferir deploy de la integración local.
- **Inventario:** [spec enriquecida](../MiNegocioCR.Api/docs/INVENTARIO_ENRIQUECIDO_v1.md) como diseño base, [evolución AI-first](../MiNegocioCR.Api/docs/INVENTARIO_AI_FIRST_JULIO_2026.md) como propuesta y [cambios de fotos Gemini](../MiNegocioCR.Api/docs/CAMBIOS_FOTOS_GEMINI_WATERMARK_JULIO_2026.md) para la evolución del pipeline. CatalogEnrichmentController y el source de fotos muestran implementación parcial de esos planes. No declarar todo el plan completado ni desplegado. Los handoffs contienen notas históricas; revisar sus aclaraciones.
- **Ventas y reparaciones:** [refactor financiero](../REFACTOR_REPAIR_PAYMENTS_SALES.md) como memoria del modelo base; [anulación/saldo a favor](../MiNegocioCR.Api/docs/SALE_VOID_STORE_CREDIT_AGO_2026.md) y [diagnóstico/abonos](../MiNegocioCR.Api/docs/CAMBIOS_REPAIR_INCOME_DIAGNOSTIC_ABONO_AGO_2026.md) como evoluciones. RegisterSaleUseCase es la autoridad del flujo actual; no reaplicar los scripts históricos.
- **Créditos:** [nota operativa v1.2](../MiNegocioCR.Api/docs/CAMBIOS_CREDITOS_MULTI_CUENTA_AGO_2026.md) y [diseño v1.2](../MiNegocioCR.Api/docs/CREDITOS_CUENTAS_COBRAR_DISENO_v1.md). N cuentas y agrupación por ContactId se verifican en ListCreditDebtorsUseCase y CreditMultiAccountAndLinks. La v1.1 de raíz es histórica.
- **WhatsApp e IA conversacional:** [aclaración vigente de conversaciones](../MiNegocioCR.Api/docs/AI-Conversaciones-y-Ventas.md). Solo su aclaración inicial fue contrastada con AIService y ConversationStateService; el resto es diseño histórico. No existe aquí una validación integral vigente de conversación/venta: verificar source antes de cambiar ese contrato. La IA de catálogo/fotos se documenta en Inventario, no en el historial de conversaciones.
- **BD y migraciones:** [guía local](../MiNegocioCR.Api/docs/local-dev-database.md) para desarrollo y [DEPLOY del API](../MiNegocioCR.Api/docs/DEPLOY.md) para procedimientos por módulo. Son guías, no inventarios exhaustivos de schema aplicado. Migrations, Program.cs y la base objetivo determinan pendientes; Git no demuestra aplicación en ningún entorno.
- **Despliegues:** [DEPLOY del API](../MiNegocioCR.Api/docs/DEPLOY.md); orden y autorización en [WORKFLOW.md](./WORKFLOW.md) y [regla de deploy](../.cursor/rules/deploy-pipeline.mdc). La copia raíz conserva contexto complementario. No ejecutar operaciones por el solo hecho de leer estos documentos.
- **Historial de cambios:** [FIXES del API](../MiNegocioCR.Api/docs/FIXES_MAYO_2026.md) como entrada al registro compartido, [FIXES raíz](../FIXES_MAYO_2026.md) para entradas propias como Lightbox y notas CAMBIOS_* por módulo. No hay un registro único exhaustivo; se conservan ambas memorias sin renumerarlas ni crear un changelog paralelo.

## Procedimiento permanente de mantenimiento

[WORKFLOW.md](./WORKFLOW.md), sección 5, es la única definición del procedimiento: leer documentación canónica y decisiones antes de cambiar, verificar el source y registrar los cambios significativos después de validar. Usa las notas existentes del módulo, conserva la historia y aplica la excepción proporcional para cambios triviales; no crea otro changelog. AGENTS.md y las reglas de proveedor solo remiten a ese procedimiento.

## Estados y evidencia

- **Vigente:** referencia mantenida para un alcance concreto, subordinada al source y a su fecha de verificación.
- **Histórico:** diagnóstico/decisión de su fecha; no equivale a comportamiento actual ni a trabajo todavía pendiente.
- **Propuesta:** diseño sujeto a decisión; **pendiente** requiere comprobar que el trabajo siga faltando.
- **IMPLEMENTADO:** existe código en el repositorio y la revisión indicados (rama/commit, o cambios locales sin commit). No implica que las pruebas hayan pasado, que todos los consumidores estén integrados o que esté en producción; registrar la validación aparte.
- **INTEGRADO:** productores y consumidores están conectados en el alcance y revisiones comprobados. Declarar si se verificó solo localmente; un merge entre ramas se registra con su evidencia y no demuestra por sí solo que el flujo funcione ni que esté desplegado.
- **DESPLEGADO:** exige evidencia de la publicación en un entorno identificado, con fecha y release/revisión comprobados. No implica validación integral de todos los flujos. Una rama, un merge, un commit, un test registrado o una migración en Git no bastan; sin evidencia, usar «despliegue no verificado». La consolidación documental de las etapas anteriores no verificó producción.

## Abrir un repositorio de forma independiente

Desde frontend o API, comenzar por su README y las reglas locales, que enlazan a esta entrada. Los enlaces entre repositorios requieren el layout del workspace descrito en [WORKSPACE_GIT.md](../docs/WORKSPACE_GIT.md).

No se garantiza que todas las herramientas carguen automáticamente AGENTS.md de la carpeta padre o archivos fuera del checkout. Si solo se clonó un producto, obtener el meta-workspace autorizado y cargar manualmente esta entrada; mientras falte, informar la limitación y usar el source y la documentación local sin inventar las reglas compartidas. Esta etapa no añade AGENTS.md a los productos ni copia reglas. Si se desea descubrimiento automático por producto, evaluar adaptadores mínimos en una etapa posterior autorizada.

## Copias y trazabilidad de esta consolidación

El [índice documental del API](../MiNegocioCR.Api/docs/WORKSPACE_INDEX.md) registra las 24 familias revisadas, la referencia escogida para cada una y la lista exacta de archivos editados en la Etapa 2. Los cuerpos se conservan; las copias no se sincronizan por reemplazo.
