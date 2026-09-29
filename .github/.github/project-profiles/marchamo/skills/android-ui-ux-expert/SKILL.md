---
name: android-ui-ux-expert
description: "Experto en UI/UX para aplicaciones Android empresariales con Jetpack Compose y Material 3, especializado en apps de captura de datos en campo (escaneo RFID/código de barras con operadores usando pistola Zebra). Cubre: sistema de diseño y tokens, jerarquía visual, estados de pantalla (loading/empty/error/content), manejo de errores accionables, accesibilidad WCAG, consistencia de navegación y copy, y componentes compartidos del proyecto MarchamoDigital.AndroidRFID. USA CUANDO: crear o modificar cualquier pantalla o Composable con impacto visual, revisar UI, diseñar estados de error o vacíos, elegir colores/tipografía/íconos, mejorar accesibilidad, unificar navegación o labels, hacer auditoría de UX, 'esta pantalla se ve mal', 'mejorar el diseño', 'agregar una pantalla nueva', review de PR con cambios de UI — incluso si el usuario no menciona 'UI/UX' explícitamente."
argument-hint: "Opcional: pantalla o componente a trabajar (ej. 'Control de Calidad', 'ReaderStatusBar') o 'auditoría' para revisión completa."
user-invocable: true
---

# Android UI/UX Expert — MarchamoDigital.AndroidRFID

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
Garantizar que toda pantalla nueva o modificada de `MarchamoDigital.AndroidRFID` cumpla los estándares de UI/UX: Material 3, estados completos, errores accionables, accesibilidad y coherencia visual.

**Problema que resuelve:**
Los operadores de campo usan la app con una pistola RFID en la mano — las decisiones de diseño deben estar validadas para ese contexto operativo específico.

**Responsabilidades que NO tiene este Skill:**
- No compila ni despliega la app (eso es `android-device-deploy`).
- No realiza code review del código .NET.
- No realiza operaciones Git.

---

## Propósito

Garantizar que toda pantalla nueva o modificada de la app cumpla un estándar consistente de UI/UX: Material 3, estados completos, errores accionables, accesibilidad y coherencia visual. La app la usan **operadores de campo con una pistola RFID en la mano** — cada decisión de diseño se evalúa bajo ese contexto: guantes, poca atención a la pantalla, condiciones de luz variables.

## Referencias (cargar según necesidad)

| Archivo | Cuándo leerlo |
|---|---|
| `references/auditoria-marchamo-2026-07.md` | Al trabajar sobre pantallas existentes — contiene los hallazgos P0/P1/P2 vigentes y sus soluciones. |
| `references/checklist-ui-ux.md` | SIEMPRE antes de dar por terminado un cambio de UI o aprobar un PR visual. |

---

## Principios rectores (en orden de prioridad)

1. **El operador primero:** la pantalla se usa de pie, con una mano, sin mirar fijo. Acciones grandes, feedback háptico/sonoro, estados legibles de un vistazo.
2. **Ningún estado mudo:** todo error tiene mensaje humano + acción de recuperación. Todo vacío indica el siguiente paso. Prohibido mostrar "Error" a secas.
3. **Una sola fuente de verdad:** labels, íconos y rutas viven en `AppDestination`. Colores y espaciados viven en el theme. Nada hardcodeado en Composables.
4. **Jerarquía antes que decoración:** primero decidir qué es lo más importante de la pantalla; el color y el énfasis se derivan de eso, no al revés.
5. **Consistencia sobre novedad:** un patrón repetido imperfecto es mejor UX que cinco patrones distintos "óptimos".

---

## Sistema de diseño

### Tokens de color — SOLO desde el theme

```kotlin
// ✅ CORRECTO: siempre tokens semánticos de Material 3
Text(text = titulo, color = MaterialTheme.colorScheme.onSurface)
Surface(color = MaterialTheme.colorScheme.surfaceContainerHigh) { /* banner */ }

// ❌ PROHIBIDO: colores literales en Composables
Text(text = titulo, color = Color(0xFF2A9D8F))
```

Mapa semántico del proyecto:

| Uso | Token M3 |
|---|---|
| Texto principal (títulos de tarjeta, headings) | `onSurface` |
| Texto secundario (descripciones, hints) | `onSurfaceVariant` |
| Acción principal / marca (teal INS) | `primary` / `onPrimary` |
| Banners informativos | `surfaceContainerHigh` + `onSurfaceVariant` |
| Estado de error | `error` / `errorContainer` + `onErrorContainer` |
| Éxito (tag válido, conexión OK) | definir `extendedColors.success` en el theme (M3 no trae verde; NO reusar `primary`) |

**Regla de contraste:** texto normal ≥ 4.5:1, texto grande ≥ 3:1. Si un texto "se ve apagado", casi siempre el error es usar `primary` oscuro como color de texto sobre fondo oscuro — usar `onSurface` y dejar el acento para el ícono.

### Tipografía

| Rol | Estilo M3 |
|---|---|
| Título de pantalla (TopAppBar) | `titleLarge` |
| Título de tarjeta / ítem de lista | `titleMedium` (SemiBold) |
| Descripción / cuerpo | `bodyMedium` |
| Metadatos (sesión, timestamps) | `bodySmall` + `onSurfaceVariant` |
| Contadores numéricos | `headlineSmall` + `fontFeatureSettings = "tnum"` (dígitos tabulares: el conteo no salta) |

### Espaciado — grilla de 4dp

```kotlin
// Definir una sola vez en el design system del proyecto
object InsSpacing {
    val ExtraSmall = 4.dp   // separación interna mínima
    val Small      = 8.dp   // entre elementos relacionados
    val Default    = 16.dp  // padding estándar de pantalla y tarjetas
    val Large      = 24.dp  // entre secciones
    val ExtraLarge = 32.dp  // aire alrededor de estados vacíos
}
```

### Iconografía — semántica única por función

| Función | Ícono (Material Symbols Outlined) | Nunca usar |
|---|---|---|
| Asignación de Etiquetas | `EditNote` | — |
| Control de Calidad | `FactCheck` | el mismo check que otra función |
| Devolución de Etiquetas | `AssignmentReturn` | flechas ← (se confunden con back) |
| Lectura RFID | `Sensors` o `Contactless` | — |
| Validar Token | `Key` | play ▶ |
| Conectar lector | `Bluetooth` / `BluetoothConnected` | — |

Regla: **un ícono = una función**. Si dos entradas comparten ícono, una de las dos está mal.

---

## Estados de pantalla (patrón obligatorio)

Toda pantalla modela su estado como `sealed interface` — nunca booleanos sueltos (`isLoading`, `hasError`):

```kotlin
// Estado de UI de la pantalla — exhaustivo por diseño:
// el `when` en el Composable obliga a renderizar TODOS los casos.
sealed interface ScreenUiState<out T> {
    data object Loading : ScreenUiState<Nothing>
    data object Empty : ScreenUiState<Nothing>
    data class Content<T>(val data: T) : ScreenUiState<T>
    data class Error(val message: UiText, val retry: (() -> Unit)?) : ScreenUiState<Nothing>
}
```

### Errores accionables — el patrón central del proyecto

El estado del lector RFID es transversal y se comparte entre pantallas:

```kotlin
// ViewModel con alcance de actividad: la conexión sobrevive a la navegación.
sealed interface ReaderUiState {
    data object Disconnected : ReaderUiState
    data object Connecting : ReaderUiState
    data class Connected(val deviceName: String, val batteryPct: Int?) : ReaderUiState
    data class Error(val message: UiText, val retryAction: RetryAction) : ReaderUiState
}
```

Cada `Error` mapea a un mensaje humano y un botón concreto:

| Causa técnica | Mensaje al operador | Acción |
|---|---|---|
| Reader no encontrado | "No se encontró el lector. Verifique que esté encendido y emparejado." | Reintentar |
| Bluetooth apagado | "Bluetooth desactivado." | Activar Bluetooth |
| Permiso denegado | "La app necesita permiso de dispositivos cercanos." | Abrir ajustes |
| Backend inaccesible | "Sin conexión con el servidor. Verifique la VPN." | Reintentar |

### Estado vacío — componente compartido

```kotlin
/**
 * Estado vacío unificado para todas las pantallas.
 * Siempre indica el SIGUIENTE PASO, no solo la ausencia de datos.
 */
@Composable
fun EmptyState(
    icon: ImageVector,
    title: String,
    hint: String? = null,
    modifier: Modifier = Modifier,
    action: (@Composable () -> Unit)? = null,
)
```

---

## Componentes compartidos del proyecto

Antes de crear un Composable nuevo, verificar si existe (o debe existir) en `core/design-system`:

| Componente | Responsabilidad |
|---|---|
| `InsRfidControlBar` | Estado persistente del lector + controles Conectar/Desconectar + Iniciar/Detener. Visible en toda pantalla de escaneo. |
| `InsRfidTagRow` | Fila de tag: #N / CodTagId bold / EPC monospace. Con slot `trailing` para estado o botón extra. |
| `EmptyState` | Estados vacíos unificados (ícono + título + siguiente paso + CTA opcional). |
| `InsPrimaryButton` | Botón Filled principal. Máximo uno por pantalla. |
| `InsSecondaryButton` | Botón Outlined con soporte de ícono. Acciones secundarias. |
| `InsTextField` | Campo de texto con `RoundedCornerShape(8.dp)` y colores semánticos. |
| `ConfirmDialog` | Confirmación de acciones destructivas (Limpiar, cancelar sesión). |
| `SessionChip` | Identificadores técnicos truncados (`SAS-…e018`) con acción copiar. Nunca GUIDs completos. |

### Anatomía de una pantalla de escaneo (patrón canónico)

```
┌──────────────────────────────────┐
│ TopAppBar (título ÚNICO)         │  ← hamburguesa si top-level, back si hija
├──────────────────────────────────┤
│ InsRfidControlBar                │  ← estado conexión + Conectar/Desconectar + Iniciar/Detener
├──────────────────────────────────┤
│ StatChips: 12 Leídas · 3 Detalle │  ← contadores con "tnum" + semantics liveRegion
├──────────────────────────────────┤
│ LazyColumn(key = { it.epc })     │  ← InsRfidTagRow, o EmptyState
│                                  │
├──────────────────────────────────┤
│ [██ Acción de negocio ██]        │  ← ÚNICO InsPrimaryButton, anclado abajo
└──────────────────────────────────┘
```

Reglas del patrón:

- Máximo **un** `InsPrimaryButton` por pantalla (la acción de negocio: Guardar sesión / Devolver / Consultar Detalle).
- `Limpiar` es `InsSecondaryButton` o ícono en la TopAppBar — habilitado solo con datos y siempre con `ConfirmDialog`.
- Haptic feedback (`HapticFeedbackType.Confirm`) al detectar tags nuevos: el operador apunta la pistola, no mira la pantalla.
- El contador usa `semantics { liveRegion = LiveRegionMode.Polite }` para TalkBack.

---

## Navegación y copy

1. **Fuente única de verdad:** `enum class AppDestination(label, description, icon)` — drawer, home y TopAppBar consumen el mismo objeto. Un label distinto entre drawer y home es un bug.
2. **Jerarquía:** destinos top-level → hamburguesa; pantallas hijas → back arrow. El nombre de la pantalla hija debe coincidir con el label que la abrió.
3. **Título solo en TopAppBar** — nunca repetido como heading debajo.
4. **Strings en `strings.xml`** con revisión ortográfica (tildes: "devolución", no "devolucón").
5. **Herramientas de desarrollo** (Validar Token, RFID Demo) visibles solo en builds `debug` o bajo sección "Diagnóstico".
6. `ModalNavigationDrawer` con ancho máximo **360dp**.

---

## Accesibilidad (no negociable)

- Contraste WCAG AA: 4.5:1 texto normal, 3:1 texto grande.
- `contentDescription` en íconos de acción; `null` en decorativos.
- Touch targets ≥ 48×48dp (`Modifier.minimumInteractiveComponentSize()`).
- Texto en `sp`; validar con escala de fuente 1.5×.
- Estados dinámicos (contador de escaneo, conexión) con `liveRegion`.
- Probar en el dispositivo real (REA-NX9) con luz de bodega, no solo en el emulador.

---

## Flujo de trabajo al recibir una tarea de UI

1. **Leer** `references/checklist-ui-ux.md` (siempre) y `references/auditoria-marchamo-2026-07.md` (si toca pantalla existente).
2. **Identificar el patrón:** ¿es pantalla de escaneo? → `InsRfidControlBar` + `ScanScreenScaffold`. ¿Lista/home? → tarjetas con jerarquía título > descripción.
3. **Modelar estados primero** (`sealed interface`), UI después.
4. **Reusar componentes** de `core/design-system`; si falta uno, crearlo ahí (no inline en la feature).
5. **Verificar el checklist completo** antes de cerrar.
6. **Proponer mejoras:** si el requerimiento pide algo que contradice estos principios (ej. un segundo botón Filled), implementar lo pedido señalando la alternativa recomendada y su razón.

---

## Skill Collaboration

Cuando la solicitud requiera capacidades fuera del dominio de este Skill, coordinar con el Skill correspondiente siguiendo `framework/DECISION-TREE.md`.

| Skill | Cuándo usarlo junto con este |
|---|---|
| `android-ui-ux-reviewer` | Checklist de revisión de PR — aplica cuando el cambio ya está listo para revisión formal |
| `android-device-deploy` | Instalar y probar en el REA-NX9 físico después de cambios visuales |
| `pr-review-marchamo` | Code review de la implementación .NET detrás de la pantalla |
| `arquitecto-de-software` | Decidir en qué módulo/capa vive cada componente nuevo (`core/design-system` vs feature) |

---

## Completion Checklist

Antes de considerar terminada cualquier tarea:

- □ Se respetó el Framework (Principles, Workflow, GIT-Policy, Conventions).
- □ Se siguió el Workflow.
- □ Se utilizó el Skill correcto.
- □ Se mantuvieron cambios mínimos.
- □ Se mantuvo consistencia con el proyecto.
- □ Se espera aprobación del desarrollador antes de cualquier operación Git.
