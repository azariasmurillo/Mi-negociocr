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

## Alcance de esta fase

Fase 1 crea solamente esta fuente neutral y sus instrucciones de área. No crea Skills ni adaptadores de proveedor. Los consumidores futuros deben leer esta carpeta y aportar un adaptador delgado que indique cómo cargarla.

