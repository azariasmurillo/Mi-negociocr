# Auditoría UI/UX — MarchamoDigital.AndroidRFID · Julio 2026

> Hallazgos identificados durante la sprint HU#261569 (2026-07-06).
> Estado al cierre de la auditoría: **P0 resueltos, P1 parcialmente resueltos, P2 pendiente.**

---

## Leyenda de prioridad

| Nivel | Criterio | Acción |
|---|---|---|
| **P0 — Bloqueante** | Rompe la UX, impide uso o viola accesibilidad básica | Corregir en el mismo PR |
| **P1 — Importante** | Inconsistencia notable, falta de estado, texto técnico expuesto | Corregir en el sprint actual |
| **P2 — Deuda técnica** | Mejora de calidad, accesibilidad avanzada, polish | Backlog / siguiente sprint |

---

## P0 — Resueltos ✅

### P0-01 · Botones inconsistentes entre pantallas
**Pantallas afectadas:** `CalidadScreen`, `DevolucionScreen`, `RfidDemoScreen`, `TokenValidationScreen`
**Problema:** 4 pantallas usaban `Button(raw)` o `OutlinedButton(raw)` directamente, con color, forma y altura distintos a los demás botones de la app.
**Solución aplicada:** Migrar a `InsPrimaryButton` y `InsSecondaryButton` del design-system. Agregar parámetro `icon` a `InsSecondaryButton` para paridad con `InsPrimaryButton`.
**Commit:** `feat(ui-ux): homologate UI across all RFID screens — 3 groups`

### P0-02 · Control RFID ad-hoc en cada pantalla
**Pantallas afectadas:** Todas las pantallas con scanner (5 pantallas distintas).
**Problema:** Cada pantalla implementaba su propio bloque de estado de conexión + botones Conectar/Desconectar + Iniciar/Detener con diseño diferente (tarjeta roja en RfidDemo, texto plano en Inventario, Row con Triple en Devolucion...).
**Solución aplicada:** Crear `InsRfidControlBar` en `core/design-system` — unifica estado de conexión (con color semántico), badge de conteo y par de botones secundario/primario.
**Commit:** `feat(ui-ux): homologate UI across all RFID screens — 3 groups`

### P0-03 · Filas de tags sin consistencia visual
**Pantallas afectadas:** `LecturaRfidScreen` (EtiquetaRow custom), `InventarioScreen` (TagItem custom).
**Problema:** Cada pantalla definía su propio composable para mostrar un tag RFID con diseño distinto (Card azul vs fondo `InsGray05`).
**Solución aplicada:** Crear `InsRfidTagRow` en `core/design-system` (#N / CodTagId bold / EPC monospace, slot trailing opcional).
**Commit:** `feat(ui-ux): homologate UI across all RFID screens — 3 groups`

### P0-04 · `ConnectionStatusCard` roja propia en RfidDemoScreen
**Pantalla afectada:** `RfidDemoScreen`
**Problema:** Tarjeta roja con `Color(0xFFFFEBEE)` hardcodeado y botón "Conectar" rojo — incompatible con el design system y confuso para el operador (rojo = error permanente aunque la app funcione).
**Solución aplicada:** Reemplazar con `InsRfidControlBar`.
**Commit:** `fix(ui-ux): apply InsRfidControlBar to RfidDemo and Inventario screens`

### P0-05 · Estado de conexión en `InventarioScreen` como texto plano
**Pantalla afectada:** `InventarioScreen`
**Problema:** "Estado del Lector: ERROR" en texto rojo con `Color.Red` literal. Sin acción de recuperación.
**Solución aplicada:** Reemplazar con `InsRfidControlBar` que incluye botón Conectar/Reintentar.
**Commit:** `fix(ui-ux): apply InsRfidControlBar to RfidDemo and Inventario screens`

---

## P1 — Pendientes ⚠️

### P1-01 · Título duplicado en pantallas hijas
**Pantallas afectadas:** `CalidadScreen`, `DevolucionScreen`, `LecturaRfidScreen`, `InventarioScreen`
**Problema:** El título de la pantalla aparece en la `TopAppBar` (correcto) Y también como `Text(headlineSmall/titleLarge)` debajo de la barra — duplicación que consume espacio en pantallas pequeñas.
**Solución esperada:** Eliminar el `Text` de título redundante. El `TopAppBar` ya lo comunica.
**Impacto:** Layout más compacto, más espacio para la lista de tags.

### P1-02 · Mensaje de error genérico "Error"
**Pantallas afectadas:** `CalidadScreen`, `DevolucionScreen` (texto `InsRfidControlBar` muestra "Error" como estado de conexión).
**Problema:** El `InsRfidControlBar` muestra el `.name` del enum `RfidConnectionState.ERROR` como "Error" sin contexto ni acción específica.
**Solución esperada:** Mapear `RfidConnectionState.ERROR` a un mensaje humano con acción Reintentar (ver tabla en SKILL.md).
**Nota:** Requiere propagar el `errorMessage` desde el ViewModel al `InsRfidControlBar`.

### P1-03 · Iconos no semánticos en el menú principal
**Pantalla afectada:** `InicioScreen`
**Problema:**
- "Devolución de Etiquetas" usa `Icons.Default.KeyboardArrowRight` (flecha → navegación, no devolución).
- "Lectura RFID para Asignación" usa `Icons.Default.CheckCircle` (semántica de éxito/verificación, no escaneo).
- "Asignación de Etiquetas" usa `Icons.Default.Edit` (edición de texto, no asignación).
**Solución esperada:** Migrar a iconos semánticos del mapa en SKILL.md (`EditNote`, `AssignmentReturn`, `Sensors`/`Contactless`).

### P1-04 · Strings con errores ortográficos
**Pantallas afectadas:** `InicioScreen`, `DevolucionScreen`
**Hallazgos:**
- "Registrar devolucón" → "Registrar devolución" (falta tilde).
- Strings hardcodeados en código Kotlin, no en `strings.xml`.
**Solución esperada:** Mover TODOS los strings a `strings.xml` y corregir ortografía.

### P1-05 · Colores hardcodeados en components legacy
**Archivos afectados:** `RfidDemoScreen` (antes de la migración), resto de screens con `InsJade50` o `InsOrange100` usados directamente en vez de `MaterialTheme.colorScheme.primary`.
**Problema:** `InsJade50` y `InsOrange100` son tokens de brand, no semánticos de M3. El theme dark/light puede no respetarlos correctamente.
**Solución esperada:** Mapear `InsJade50` → `colorScheme.primary`, `InsError60` → `colorScheme.error` en el `InsTheme`. Los Composables usan solo tokens semánticos.

---

## P2 — Deuda técnica (próximo sprint) 📋

### P2-01 · Estado `Loading` no modelado
**Pantallas afectadas:** Todas.
**Problema:** Las pantallas usan `isLoading: Boolean` suelto en lugar de `sealed interface ScreenUiState`.
**Solución esperada:** Migrar a `sealed interface` para que el compilador garantice exhaustividad de estados.

### P2-02 · `EmptyState` no existe como componente compartido
**Pantallas afectadas:** Todas las pantallas de escaneo muestran el estado vacío de forma diferente.
**Problema:** Cada pantalla define su propia visualización de "sin datos" con diseños distintos.
**Solución esperada:** Crear `EmptyState(icon, title, hint, action?)` en `core/design-system` y reemplazar todas las implementaciones inline.

### P2-03 · Haptic feedback no implementado
**Pantallas afectadas:** `LecturaRfidScreen`, `CalidadScreen`, `DevolucionScreen`.
**Problema:** Al detectar un tag nuevo no hay vibración. El operador apunta la pistola Zebra sin mirar la pantalla — necesita retroalimentación táctil.
**Solución esperada:** `LocalHapticFeedback.current.performHapticFeedback(HapticFeedbackType.Confirm)` en el handler de `EpcRecibido`.

### P2-04 · Contadores sin dígitos tabulares
**Pantallas afectadas:** `CalidadScreen` (`InfoBadge`), `InsRfidControlBar` (badge de conteo).
**Problema:** Los dígitos cambian de ancho al actualizar el contador (ej. "9" → "10") causando un salto visual.
**Solución esperada:** `fontFeatureSettings = "tnum"` en los `Text` de contadores numéricos.

### P2-05 · GUIDs y EPCs crudos visibles al usuario
**Pantallas afectadas:** `InventarioScreen` (muestra el `sessionId` completo), `InsRfidTagRow` (muestra EPC completo).
**Problema:** `SAS-e2336a7a6fff4f4baa5743f0828ecffc` visible completo — innecesariamente largo e ilegible para el operador.
**Solución esperada:** Crear `SessionChip` que trunca a `SAS-…ecfc` con acción de copiar el valor completo.

### P2-06 · Sección "Herramientas" visible en producción
**Pantalla afectada:** `InicioScreen`
**Problema:** "RFID Demo" y "Validar Token" son herramientas de desarrollo visibles en todos los builds incluyendo producción.
**Solución esperada:** Mostrar la sección "HERRAMIENTAS" solo en `BuildConfig.DEBUG`.

### P2-07 · `AppDestination` no unificado
**Pantallas afectadas:** Toda la app.
**Problema:** Los labels del drawer, las cartas de `InicioScreen` y los títulos en `TopAppBar` son strings sueltos independientes — cualquier renombre requiere 3 cambios manuales.
**Solución esperada:** `enum class AppDestination(val label: String, val description: String, val icon: ImageVector)` como única fuente de verdad.

---

## Historial de cambios

| Fecha | Acción | P0/P1/P2 |
|---|---|---|
| 2026-07-06 | Creación de `InsRfidControlBar`, `InsRfidTagRow`, `InsSecondaryButton(icon)` | P0-01, P0-02, P0-03 resueltos |
| 2026-07-06 | Migración de `RfidDemoScreen` e `InventarioScreen` a design system | P0-04, P0-05 resueltos |
| — | P1-01 a P1-05, P2-01 a P2-07 pendientes | Próximo sprint |
