---
name: android-ui-ux-reviewer
description: "Revisor UI/UX para pantallas Android (Jetpack Compose) del ecosistema MarchamoDigital. Aplica el checklist oficial de 6 secciones: estados de pantalla, accesibilidad, consistencia, patrón RFID, jerarquía visual y validación en dispositivo. Un hallazgo en secciones 1–3 bloquea el merge. USA CUANDO: revisar pantallas nuevas o modificadas, validar PR con cambios de UI, detectar inconsistencias de design system, revisar antes de demo o entrega."
argument-hint: "Indica qué pantallas revisar, por ejemplo: 'revisa LecturaRfidScreen y CalidadScreen' o 'revisa todos los cambios del PR #5 de MarchamoDigital.AndroidRFID'"
user-invocable: true
---

# Revisor UI/UX — MarchamoDigital Android

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
Revisar pantallas Android (Jetpack Compose) aplicando el checklist oficial de UI/UX de 6 secciones y emitir veredicto de merge.

**Problema que resuelve:**
Los PRs con cambios de UI necesitan una revisión estandarizada que garantice accesibilidad, consistencia y el patrón RFID correcto antes del merge.

**Responsabilidades que NO tiene este Skill:**
- No modifica código fuente.
- No publica comentarios en el PR automáticamente (eso es `pr-review-publisher`).
- No realiza operaciones Git.

---

## Propósito

Revisar pantallas Android (Jetpack Compose) del repositorio `MarchamoDigital.AndroidRFID` aplicando el **checklist oficial de UI/UX** del equipo.

Un hallazgo **Bloqueante** (secciones 1–3) impide el merge hasta ser corregido.
Un hallazgo **Mejora** (secciones 4–6) se recomienda pero no bloquea.

---

## Checklist de revisión (fuente de verdad)

### Sección 1 — Estados de pantalla (Bloqueante)

| # | Control | Criterio |
|---|---|---|
| 1.1 | Modelado de estados | La pantalla modela **todos** los estados: `Loading`, `Empty`, `Content`, `Error` como sealed interface, nunca booleanos sueltos |
| 1.2 | Mensajes de error | Ningún error muestra texto genérico ("Error") — siempre mensaje humano + acción de recuperación |
| 1.3 | Estado vacío | `Empty` usa el componente `EmptyState` compartido (ícono + título + siguiente paso) |
| 1.4 | Botones condicionales | Botones deshabilitados cuando su acción no aplica (`Limpiar` con lista vacía) y habilitados solo con precondiciones cumplidas |
| 1.5 | Confirmación destructiva | Acciones destructivas (`Limpiar`, `Cancelar sesión`) piden confirmación con `ConfirmDialog` |

### Sección 2 — Accesibilidad (Bloqueante)

| # | Control | Criterio |
|---|---|---|
| 2.1 | Contraste de color | Texto/fondo ≥ 4.5:1 (normal) o ≥ 3:1 (texto grande ≥ 18sp). Verificar contra tokens del theme |
| 2.2 | `contentDescription` | Todos los íconos de acción tienen `contentDescription`; decorativos usan `null` |
| 2.3 | Touch targets | ≥ 48×48dp — usar `minimumInteractiveComponentSize()` si el visual es menor |
| 2.4 | Unidades de texto | Texto en `sp`, nunca `dp`. No se rompe con escala de fuente 1.5× |
| 2.5 | Regiones en vivo | Contadores/estados que cambian solos usan `semantics { liveRegion = LiveRegionMode.Polite }` |

### Sección 3 — Consistencia (Bloqueante)

| # | Control | Criterio |
|---|---|---|
| 3.1 | Labels de navegación | Provienen de `AppDestination` (única fuente de verdad: drawer, home, pantalla) |
| 3.2 | Título duplicado | Título de pantalla SOLO en `TopAppBar` — nunca duplicado como heading debajo |
| 3.3 | Navegación | Top-level → hamburguesa; hijas → back arrow. Nunca mezclados |
| 3.4 | Strings externalizados | Todos los strings en `strings.xml`, revisados ortográficamente (tildes, español correcto) |
| 3.5 | Sin colores hardcodeados | Cero `Color(0xFF...)` — solo `MaterialTheme.colorScheme.*` |
| 3.6 | Datos técnicos visibles | Cero GUIDs, EPCs completos ni URLs visibles al usuario final: truncar + acción de copiar |

### Sección 4 — Patrón de pantalla RFID (Mejora)

| # | Control | Criterio |
|---|---|---|
| 4.1 | `InsRfidControlBar` | Usa el componente compartido del design-system (no implementación ad-hoc) |
| 4.2 | Botón toggle escaneo | "Iniciar escaneo" ↔ "Detener escaneo" — color `error` tonal cuando está activo |
| 4.3 | Contador en vivo | Contador de etiquetas visible, con `fontFeatureSettings = "tnum"` o `animateContentSize` |
| 4.4 | Haptic feedback | `HapticFeedbackType.LongPress` o `TextHandleMove` al detectar tags nuevos |
| 4.5 | `LazyColumn` con key | Lista de tags usa `LazyColumn` con `key = { it.epc }` |

### Sección 5 — Jerarquía visual (Mejora)

| # | Control | Criterio |
|---|---|---|
| 5.1 | Pesos tipográficos | Título con más peso visual que descripción (`titleMedium onSurface` > `bodyMedium onSurfaceVariant`) |
| 5.2 | Banners informativos | En superficie tonal (`surfaceContainerHigh` / `secondaryContainer`) — no bloques blancos sobre fondo oscuro |
| 5.3 | Botón único Filled | Máximo UN `InsPrimaryButton` visible por pantalla; secundarias en `InsSecondaryButton` / `TextButton` |
| 5.4 | Spacing en grilla | Valores de spacing múltiplos de 4dp usando constantes del design system (`InsSpacing.*`), no `dp` mágicos |

### Sección 6 — Validación en dispositivo (Mejora)

| # | Control | Criterio |
|---|---|---|
| 6.1 | Dispositivo referencia | Probado en REA-NX9 (Android 14) en condiciones reales de luz |
| 6.2 | Desconexión brusca | Probado el flujo con el lector desconectándose a mitad de escaneo |
| 6.3 | Screenshots | Adjuntos al PR el antes/después de las pantallas modificadas |

---

## Procedimiento de revisión

### PASO 1 — Identificar pantallas afectadas

```
Listar archivos *Screen.kt modificados en el PR o en el conjunto indicado.
Para cada pantalla: leer el archivo completo antes de evaluar.
```

### PASO 2 — Evaluar sección por sección

Para **cada pantalla** y **cada control del checklist**:

1. Buscar la evidencia en el código (composable, state, ViewModel, Contract).
2. Marcar como: ✅ Cumple / ⚠️ Mejora / ❌ Bloqueante / ⬜ No aplica.
3. Si ❌ o ⚠️: citar línea exacta y proponer la corrección concreta.

### PASO 3 — Consultar el design system

Antes de marcar un color, spacing o componente como "inconsistente":
- Verificar en `core/design-system/src/main/java/.../theme/Color.kt`
- Verificar en `core/design-system/src/main/java/.../theme/Spacing.kt`
- Verificar en `core/design-system/src/main/java/.../components/`

### PASO 4 — Generar el reporte

Formato del reporte:

```
## Revisión UI/UX — [NombrePantalla]

### Resumen
| Sección | Resultado | Hallazgos |
|---|---|---|
| 1. Estados | ✅/⚠️/❌ | N |
| 2. Accesibilidad | ✅/⚠️/❌ | N |
| 3. Consistencia | ✅/⚠️/❌ | N |
| 4. Patrón RFID | ✅/⚠️/⬜ | N |
| 5. Jerarquía | ✅/⚠️/⬜ | N |
| 6. Dispositivo | ✅/⬜ | N |

### Veredicto: APROBADO / REQUIERE CAMBIOS

### Hallazgos detallados

#### [ID] [Severidad] [Control] — [Descripción breve]
**Archivo**: `feature/xxx/XxxScreen.kt` línea N
**Evidencia**: `código actual`
**Corrección**:
```kotlin
// código corregido
```
```

### PASO 5 — Decisión de merge

- Si hay 1 o más ❌ en secciones 1–3 → **REQUIERE CAMBIOS** (no mergeable)
- Solo ⚠️ en secciones 4–6 → **APROBADO CON OBSERVACIONES** (mergeable, mejoras en siguiente iteración)
- Todos ✅ → **APROBADO**

---

## Design system de referencia

| Elemento | Ubicación | Notas |
|---|---|---|
| Colores | `core/design-system/.../theme/Color.kt` | `InsJade50`, `InsError60`, `InsGray*`, `InsPrimary*` |
| Spacing | `core/design-system/.../theme/Spacing.kt` | `InsSpacing.Small/Default/Large/ExtraLarge` |
| Botón primario | `InsButtons.kt → InsPrimaryButton` | `InsJade50`, `shape.extraLarge`, `height 48dp` |
| Botón secundario | `InsButtons.kt → InsSecondaryButton` | `OutlinedButton` con `border InsJade50` |
| Control RFID | `InsRfidComponents.kt → InsRfidControlBar` | Estado + Conectar/Desconectar + Iniciar/Detener |
| Fila tag RFID | `InsRfidComponents.kt → InsRfidTagRow` | #N / CodTagId bold / EPC monospace |
| Campo de texto | `InsTextFields.kt → InsTextField` | `RoundedCornerShape(8.dp)` |

---

## Patrones conocidos del proyecto

### Pantallas RFID (LecturaRfid, Calidad, Devolucion, Inventario, RfidDemo)

Estructura esperada:
```kotlin
Column {
    // 1. Título (solo si NO hay TopAppBar — evitar duplicar)
    Text(text = "...", style = headlineSmall, fontWeight = Bold)
    
    // 2. Control RFID unificado
    InsRfidControlBar(connectionState, isScanning, tagCount, onConnect, onDisconnect, onToggleScan)
    
    // 3. Contenido (lista lazy con key)
    LazyColumn {
        itemsIndexed(tags, key = { _, t -> t.epc }) { idx, tag ->
            InsRfidTagRow(numero = idx+1, codTagId = ..., epc = tag.epc)
        }
    }
    
    // 4. CTA anclado al fondo (InsPrimaryButton)
    InsPrimaryButton(text = "Guardar sesión", onClick = ..., enabled = tags.isNotEmpty())
}
```

### Manejo de errores correcto
```kotlin
// ❌ MAL — mensaje genérico
Text("Error", color = InsError60)

// ✅ BIEN — mensaje humano + acción
if (state.error != null) {
    ErrorBanner(
        message = state.error!!,
        actionLabel = "Reintentar",
        onAction = { viewModel.handleIntent(Intent.Retry) }
    )
}
```

### Empty state correcto
```kotlin
// ❌ MAL — nada o texto plano
Text("Sin etiquetas", color = outline)

// ✅ BIEN — EmptyState compartido
EmptyState(
    icon = Icons.Default.Nfc,
    title = "Sin etiquetas escaneadas",
    subtitle = "Conecta el lector e inicia el escaneo"
)
```

---

## Restricciones

- **NO modificar archivos** durante la revisión — solo analizar y reportar.
- **NO inventar estándares** que no existan en el design system o en implementaciones existentes.
- Si una pantalla no usa RFID (ej. `AbrirSesionScreen`), marcar la sección 4 como ⬜ No aplica.
- Si el PR no incluye screenshots adjuntos, marcar 6.3 como ❌ y solicitar como bloqueante suave (sin impedir merge si las otras secciones pasan).

---

## Skill Collaboration

Cuando la solicitud requiera capacidades fuera del dominio de este Skill, coordinar con el Skill correspondiente siguiendo `framework/DECISION-TREE.md`.

| Necesidad | Skill |
|---|---|
| Diseñar o crear la pantalla | `android-ui-ux-expert` |
| Instalar y probar en dispositivo físico | `android-device-deploy` |
| Publicar hallazgos en el PR | `pr-review-publisher` |

---

## Completion Checklist

Antes de considerar terminada cualquier tarea:

- □ Se respetó el Framework (Principles, Workflow, GIT-Policy, Conventions).
- □ Se siguió el Workflow.
- □ Se utilizó el Skill correcto.
- □ Se mantuvieron cambios mínimos.
- □ Se mantuvo consistencia con el proyecto.
- □ Se espera aprobación del desarrollador antes de cualquier operación Git.
